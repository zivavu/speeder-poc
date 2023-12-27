<script context="module" lang="ts">
	class Particle {
		x: number;
		y: number;
		dirX: number;
		dirY: number;
		alpha: number;
		size: number;

		constructor({ x = 0, y = 0, dirX = 0, dirY = 0, alpha = 1, size = 10 }) {
			this.x = x;
			this.y = y;
			this.dirX = dirX;
			this.dirY = dirY;
			this.alpha = alpha;
			this.size = size;
		}

		draw(ctx: CanvasRenderingContext2D) {
			ctx.beginPath();

			ctx.fillStyle = `rgba(255, 255, 255, ${this.alpha})`;

			ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
			ctx.fill();
			ctx.closePath();
		}
	}

	const particlesArr: Particle[] = [];

	export function createParticleAtXY(x: number, y: number) {
		const particle = new Particle({ x, y });
		particle.size = Math.random() * 5 + 1;

		particle.dirX = Math.random() * 5 - 2.5;
		particle.dirY = Math.random() * 5 - 2.5;
		particlesArr.push(particle);
		console.log(particlesArr);
	}

	export function createParticle(props: Partial<Particle>) {
		const particle = new Particle(props);

		particlesArr.push(particle);
	}

	export function updateParticles() {
		particlesArr.forEach((particle) => {
			particle.x += particle.dirX;
			particle.y += particle.dirY;
			particle.alpha -= 0.01;
		});
		particlesArr.forEach((particle, index) => {
			if (particle.alpha <= 0) {
				particlesArr.splice(index, 1);
			}
		});
	}

	export function drawParticles(ctx: CanvasRenderingContext2D) {
		if (!ctx) return;
		particlesArr.forEach((particle) => {
			particle.draw(ctx);
		});
	}
</script>
