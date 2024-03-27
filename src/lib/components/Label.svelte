<script lang="ts">
	import { Text } from '@threlte/extras';
	import type { ColorRepresentation } from 'three';
	import { useReflow } from '@threlte/flex';
	import { forwardEventHandlers } from '@threlte/core';
	export let text: string;
	export let color: ColorRepresentation = 'white';
	export let z = 0;
	export let fontStyle:
		| 'black'
		| 'bold'
		| 'extra-bold'
		| 'extra-light'
		| 'light'
		| 'medium'
		| 'regular'
		| 'semi-bold'
		| 'thin' = 'regular';
	export let anchorX = '50%';
	export let anchorY = '50%';
	export let fontSize: number = 1;
	
    // $: fontUrl = `/fonts/inter/inter-${fontStyle}.ttf`;
	
    const reflow = useReflow();
</script>

<!-- font={fontUrl} -->
<Text
	position.z={z}
	{text}
	{anchorX}
	{anchorY}
	fontSize={fontSize}
	{color}
	on:sync={reflow}
/>
