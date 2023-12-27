<script lang="ts" context="module">
	import { onMount } from 'svelte';
	import { animationFrame } from './canvas/animationFrame.svelte';
	import { onMouseOver, primaryAction, primaryActionEnd } from './canvas/canvasHandlers.svelte';
	export let canvas: HTMLCanvasElement | null;
	export let context: CanvasRenderingContext2D | null;
</script>

<script lang="ts">
	function setCanvasSize() {
		if (!canvas) return;
		canvas.width = window.innerWidth;
		canvas.height = window.innerHeight;
	}

	onMount(() => {
		if (!canvas) return;
		context = canvas.getContext('2d');
		setCanvasSize();
		window.onresize = setCanvasSize;
		window.requestAnimationFrame(animationFrame);

		window.onkeydown = (e: KeyboardEvent) => {
			primaryAction();
		};
		window.onkeyup = (e: KeyboardEvent) => {
			primaryActionEnd();
		};
	});
</script>

<canvas on:pointermove={onMouseOver} bind:this={canvas}>
	Your browser does not support the canvas tag.
</canvas>

<style>
	:global(body) {
		background-color: black;
	}
	canvas {
		position: absolute;
		top: 0;
		left: 0;
		background-color: black;
		width: 100vw;
		height: 100vh;
	}
</style>
