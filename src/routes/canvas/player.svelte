<script context="module" lang="ts">
	import { canvas } from '../+page.svelte';
	import { mousePos } from './canvasHandlers.svelte';
	import { createParticle } from './particles.svelte';

	interface TrustVector {
		x: number;
		y: number;
		power: number;
		time: number;
		startTime: number;
	}

	export const Player = class {
		x: number;
		y: number;
		size: number;
		engineSize: number;
		rotationAngle: number;

		manoeverability: number;
		trustVector: TrustVector | null;
		momentum: { x: number; y: number };

		maxEnergy: number;
		chargedEnergy: number;
		isCharging: boolean;
		chargeRate: number;

		constructor() {
			this.x = 300;
			this.y = 300;
			this.rotationAngle = Math.random() * 2 * Math.PI;
			this.size = 20;
			this.engineSize = this.size / 2.5;
			this.manoeverability = 0.1;
			this.trustVector = null;
			this.momentum = { x: 0, y: 0 };

			this.maxEnergy = 30;
			this.chargedEnergy = 0;
			this.isCharging = false;
			this.chargeRate = 0.5;
		}

		update() {
			this.updateTrustVector();
			this.moveToMouseAngle();

			this.updateMomentum();
			this.updatePosition();
			this.updateCharge();

			this.borderCrossCheck();
		}

		draw(ctx: CanvasRenderingContext2D, canvas: HTMLCanvasElement) {
			const drawKiteBody = () => {
				ctx.strokeStyle = 'white';
				ctx.lineWidth = 2;

				ctx.beginPath();

				ctx.moveTo(this.x, this.y);
				ctx.lineTo(
					this.x - this.size * Math.cos(this.rotationAngle + Math.PI / 4),
					this.y - this.size * Math.sin(this.rotationAngle + Math.PI / 4)
				);
				ctx.lineTo(
					this.x - this.size * Math.cos(this.rotationAngle + Math.PI),
					this.y - this.size * Math.sin(this.rotationAngle + Math.PI)
				);
				ctx.lineTo(
					this.x - this.size * Math.cos(this.rotationAngle - Math.PI / 4),
					this.y - this.size * Math.sin(this.rotationAngle - Math.PI / 4)
				);
				ctx.lineTo(this.x, this.y);
				ctx.fillStyle = 'black';
				ctx.fill();

				ctx.closePath();
				ctx.stroke();
			};

			const drawKiteEngine = () => {
				const engineSize = this.engineSize;

				ctx.fillStyle = 'white';
				ctx.translate(this.x, this.y);
				ctx.rotate(this.rotationAngle);

				ctx.fillRect(-engineSize, -engineSize / 4, engineSize, engineSize / 2);
				ctx.fillStyle = 'white';
				ctx.fill;
				ctx.setTransform(1, 0, 0, 1, 0, 0);
			};

			const drawEnergryBar = () => {
				const energyBarWidth = canvas.width / 2;
				const energyBarHeight = 30;

				const energyBarX = energyBarWidth / 2;
				const energyBarY = canvas?.height - 50 + energyBarHeight / 2;

				const energyBarFillWidth = (this.chargedEnergy / this.maxEnergy) * energyBarWidth;

				ctx.fillStyle = 'white';
				ctx.fillRect(energyBarX, energyBarY, energyBarWidth, energyBarHeight);
				ctx.fillStyle = 'green';
				ctx.fillRect(energyBarX, energyBarY, energyBarFillWidth, energyBarHeight);
			};

			drawKiteBody();
			drawKiteEngine();

			drawEnergryBar();
		}

		startCharging = () => {
			this.isCharging = true;
		};
		stopCharging = () => {
			this.isCharging = false;
		};

		updateCharge = () => {
			if (this.isCharging) {
				this.chargedEnergy += this.chargeRate;
				if (this.chargedEnergy > this.maxEnergy) {
					this.chargedEnergy = this.maxEnergy;
				}
			}
			if (!this.isCharging) {
				this.chargedEnergy -= this.chargeRate;
				if (this.chargedEnergy < 0) {
					this.chargedEnergy = 0;
				}
			}
		};

		spendEnergy = () => {
			this.stopCharging();
			if (this.chargedEnergy > 0) {
				this.addTrustVector(this.chargedEnergy);
				this.chargedEnergy = 0;
			}
		};

		updateTrustVector = () => {
			if (this.trustVector === null) return;

			if (this.trustVector.time > 0) {
				this.trustVector.time--;

				const particleX = this.x - this.engineSize / 2;
				const particleY = this.y - 4;

				const enginePower = this.getEnginePower();

				createParticle({
					x: particleX,
					y: particleY,
					dirX: Math.cos(this.rotationAngle) * -1 + Math.random() * 1 - 0.5,
					dirY: Math.sin(this.rotationAngle) * -1 + Math.random() * 1 - 0.5,
					size: Math.random() * 4 + 1 + enginePower / 2
				});
			} else {
				this.trustVector = null;
			}
		};

		addTrustVector = (charge: number) => {
			const power = charge / 5;
			const startTime = charge * 6;
			const trustVector: TrustVector = {
				x: Math.cos(this.rotationAngle),
				y: Math.sin(this.rotationAngle),
				power,
				startTime: startTime,
				time: startTime
			};

			this.trustVector = trustVector;
		};

		getTrust = () => {
			if (this.trustVector === null) return { x: 0, y: 0 };

			const trustVector = this.trustVector;

			const timeBasedPower = this.getEnginePower() * 0.03;

			const rotationAngleX = Math.cos(this.rotationAngle);
			const rotationAngleY = Math.sin(this.rotationAngle);

			const trustX = rotationAngleX * timeBasedPower;
			const trustY = rotationAngleY * timeBasedPower;

			return { x: trustX, y: trustY };
		};

		getEnginePowerTimeFactor = () => {
			if (this.trustVector === null) return 0;

			const trustVector = this.trustVector;

			const timeFactor = trustVector.time / trustVector.startTime;
			// https://easings.net/#easeOutCirc
			const easedTimeFactor = timeFactor === 1 ? 1 : 1 - Math.pow(2, -10 * timeFactor);
			return easedTimeFactor;
		};

		getEnginePower = () => {
			if (this.trustVector === null) return 0;

			const trustVector = this.trustVector;

			const timeFactor = this.getEnginePowerTimeFactor();

			const timeBasedPower = trustVector.power * timeFactor;

			return timeBasedPower;
		};

		updateMomentum = () => {
			const trust = this.getTrust();

			this.momentum.x += trust.x;
			this.momentum.y += trust.y;

			this.momentum.x *= 0.995;
			this.momentum.y *= 0.995;
		};

		updatePosition = () => {
			this.x += this.momentum.x;
			this.y += this.momentum.y;
		};

		borderCrossCheck = () => {
			if (!canvas) return;
			if (this.x > canvas.width) this.x = 0;
			if (this.x < 0) this.x = canvas.width;
			if (this.y > canvas.height) this.y = 0;
			if (this.y < 0) this.y = canvas.height;
		};

		moveToMouseAngle() {
			const mouseRad = Math.atan2(this.y - mousePos.y, this.x - mousePos.x) + Math.PI;

			const isSmallMovement = Math.abs(mouseRad - this.rotationAngle) < this.manoeverability;

			if (isSmallMovement) {
				this.rotationAngle = mouseRad;
				return;
			}

			const isMouseClockwise =
				Math.cos(this.rotationAngle) * (mousePos.y - this.y) -
					Math.sin(this.rotationAngle) * (mousePos.x - this.x) <
				0;

			this.rotationAngle += isMouseClockwise ? -this.manoeverability : +this.manoeverability;

			if (this.rotationAngle > Math.PI * 2) this.rotationAngle = 0;
			if (this.rotationAngle < 0) this.rotationAngle = Math.PI * 2;
		}
	};
</script>
