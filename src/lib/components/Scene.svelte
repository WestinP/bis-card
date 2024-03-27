<script lang="ts">
	import { T } from '@threlte/core';
	import { ContactShadows, Float, Grid, OrbitControls, interactivity } from '@threlte/extras';
	import { Box, Flex, tailwindParser } from '@threlte/flex';
	import { spring } from 'svelte/motion';
	import { writable } from 'svelte/store';
	import { MeshStandardMaterial, PerspectiveCamera } from 'three';
	import Label from './Label.svelte';

	interactivity();

	let rotX = writable(0);
	let rotY = writable(0);
	let rotZ = writable(0);

	let posZ = spring(0, {
		stiffness: 0.4,
		damping: 0.4
	});

	// wiggle effect
	setInterval(() => {
		rotX.update((n) => Math.sin(Date.now() / 1000) * 0.1);
	}, 1000 / 60);

	setInterval(() => {
		rotY.update((n) => Math.cos(Date.now() / 1000) * 0.1);
	}, 1000 / 60);

	setInterval(() => {
		rotZ.update((n) => Math.sin(Date.now() / 1000) * 0.03);
	}, 1000 / 60);
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 15]} fov={15}></T.PerspectiveCamera>

<Flex>
	<T.Mesh
		position.z={$posZ}
		rotation.x={$rotX}
		rotation.y={$rotY}
		rotation.z={$rotZ}
		on:pointerover={() => posZ.set(1)}
		on:pointerout={() => posZ.set(0)}
		on:click={() => {}}
	>
		<T.BoxGeometry args={[3.5, 2, 0.025]} />
		<T.MeshStandardMaterial color="white" metalness={0.8} />
		<Label text={'please work :3'} z={0.013} fontStyle="semi-bold" fontSize={0.3} color="#FF0000" />
	</T.Mesh>
</Flex>

<T.DirectionalLight intensity={4} position.x={5} position.y={10} position.z={15} />

<ContactShadows scale={10} blur={2} far={2.5} opacity={0.5} />
