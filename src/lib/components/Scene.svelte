<script lang="ts">
	import { T, useTask } from '@threlte/core';
	import { ContactShadows, Float, Grid, OrbitControls, interactivity } from '@threlte/extras';
	import { Box, Flex, tailwindParser } from '@threlte/flex';
	import { spring } from 'svelte/motion';
	import { writable } from 'svelte/store';
	import { Group, MeshStandardMaterial, PerspectiveCamera } from 'three';
	import Label from './Label.svelte';

	interactivity();

	let rotX = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});
	let rotY = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});
	let rotZ = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});

	let outerRotX = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});
	let outerRotY = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});
	let outerRotZ = spring(0, {
		stiffness: 0.4,
		damping: 0.5
	});

	let posZ = spring(0, {
		stiffness: 0.4,
		damping: 0.4
	});

	useTask(() => {
		if (flipping) {
			rotX.set(0);
			rotY.set(0);
			rotZ.set(0);
			return;
		}

		outerRotX.update((n) => Math.sin(Date.now() / 700) * 0.1);
		outerRotY.update((n) => Math.cos(Date.now() / 700) * 0.1);
		outerRotZ.update((n) => Math.sin(Date.now() / 500) * 0.02);
	});

	// card flip
	let flipped = false;
	let flipping = false;
	let flipState = 1;

	let flipRotation = writable(0);
	let flipPosition = writable(1);

	let timeSinceStateChange = 0;
	useTask((delta) => {
		if (!flipping) return;

		const slideInTime = 0.05;
		const rotationTime = 0.15;
		const slideOutTime = 0.05;

		switch (flipState) {
			// case 1: sliding down to y -= 1
			case 1:
				timeSinceStateChange += delta;
				flipPosition.update((n) => n - delta / 1 / slideInTime);
				if (timeSinceStateChange >= slideInTime) {
					flipState = 2;
					timeSinceStateChange = 0;
				}
				break;
			// case 2: rotating flipRotation Math.PI
			case 2:
				timeSinceStateChange += delta;
				flipRotation.update((n) => n - (delta / rotationTime) * Math.PI);
				if (timeSinceStateChange >= rotationTime) {
					flipState = 3;
					timeSinceStateChange = 0;
				}
				break;
			// case 3: sliding down to y -= 1
			case 3:
				timeSinceStateChange += delta;
				flipPosition.update((n) => n - delta / 1 / slideOutTime);
				if (timeSinceStateChange >= slideOutTime) {
					flipState = 1;
					timeSinceStateChange = 0;
					flipping = false;
					flipPosition.set(flipped ? -1 : 1);
					posZ.set(0);
				}
				break;
		}
	});

	function flip() {
		if (flipping) return;

		flipped = !flipped;
		flipping = true;
	}

	function mouseOverCard(e: any) {
		// get the eventObject intersection
		const int = e.intersections.find((i: any) => i.object === e.eventObject);
		if (!int) return;

		const { uv } = int;
		const { x, y } = uv;

		// x: 0 -> 1
		// y: 0 -> 1
		// rotate the card based on the mouse position
		rotX.set(-1 * (y - 0.5) * 0.5);
		rotY.set((x - 0.5) * 0.5);
	}

	let cardGroup: Group;
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 15]} fov={15}></T.PerspectiveCamera>

<Flex>
	<T.Group
		position.z={$posZ}
		rotation.x={$outerRotX}
		rotation.y={$outerRotY}
		rotation.z={$outerRotZ}
	>
		<T.Mesh
			rotation.x={$rotX}
			rotation.y={$rotY}
			rotation.z={$rotZ}
			on:pointerover={() => {
				if (flipping) return;
				posZ.set(1);
			}}
			on:pointerout={() => {
				if (flipping) return;
				rotX.set(0);
				rotY.set(0);
				posZ.set(0);
			}}
			on:pointermove={(e) => {
				e.stopPropagation();

				if (flipping) return;
				mouseOverCard(e);
			}}
			on:click={(e) => {
				e.stopPropagation();
				flip();
			}}
		>
			<T.BoxGeometry args={[3.5, 2, 0.025]} />
			<T.MeshStandardMaterial color="white" metalness={0.8} />
			<Label
				text={'please work :3'}
				z={0.013}
				fontStyle="semi-bold"
				fontSize={0.3}
				color="#FF0000"
			/>
		</T.Mesh>
	</T.Group>
</Flex>

<T.DirectionalLight intensity={4} position.x={5} position.y={10} position.z={15} />

<ContactShadows scale={10} blur={2} far={2.5} opacity={0.5} />
