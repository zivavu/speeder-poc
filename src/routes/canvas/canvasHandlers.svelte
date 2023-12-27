<script lang="ts" context="module">
	import { Player } from './player.svelte';
	export let mousePos = { x: 0, y: 0 };

	export const points: { x: number; y: number }[] = [];
	export const player = new Player();

	export function onMouseOver(e: MouseEvent) {
		mousePos = { x: e.clientX, y: e.clientY };
	}

	export function primaryAction() {
		player.startCharging();
	}

	export function primaryActionEnd() {
		player.spendEnergy();
	}
</script>
