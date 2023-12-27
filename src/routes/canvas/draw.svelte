<script context="module" lang="ts">
	import { player } from './canvasHandlers.svelte';
	import { drawParticles } from './particles.svelte';

	export function draw(canvas: HTMLCanvasElement) {
		const ctx = canvas.getContext('2d');
		if (!ctx) return;

		ctx.clearRect(0, 0, canvas.width, canvas.height);
		drawGrid(ctx);

		drawParticles(ctx);
		player.draw(ctx, canvas);
	}

	function drawGrid(ctx: CanvasRenderingContext2D) {
		const { width, height } = ctx.canvas;
		const gridSize = 50;
		ctx.strokeStyle = '#333';
		ctx.lineWidth = 1;
		ctx.beginPath();
		for (let x = 0; x < width; x += gridSize) {
			ctx.moveTo(x, 0);
			ctx.lineTo(x, height);
		}
		for (let y = 0; y < height; y += gridSize) {
			ctx.moveTo(0, y);
			ctx.lineTo(width, y);
		}
		ctx.stroke();
	}
</script>
