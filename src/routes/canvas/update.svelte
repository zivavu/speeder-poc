<script context="module" lang="ts">
	import { player } from './canvasHandlers.svelte';
	import { updateParticles } from './particles.svelte';

	export function update() {
		player.update();
		updateParticles();
	}
</script>
