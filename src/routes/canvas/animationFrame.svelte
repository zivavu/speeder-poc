<script context="module" lang="ts">
	import { canvas } from '../+page.svelte';
	import { draw } from './draw.svelte';
	import { update } from './update.svelte';

	export function animationFrame() {
		update();

		if (canvas) {
			draw(canvas);
		}

		window.requestAnimationFrame(animationFrame);
	}
</script>
