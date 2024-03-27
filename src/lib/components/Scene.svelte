<script lang="ts">
	import { T } from '@threlte/core';
	import { ContactShadows, Float, Grid, OrbitControls, interactivity } from '@threlte/extras';
	import { Box, Flex, tailwindParser } from '@threlte/flex';
	import { spring } from 'svelte/motion';
	import { writable } from 'svelte/store';
	import { Group, MeshStandardMaterial, PerspectiveCamera } from 'three';
	import Label from './Label.svelte';

	interactivity();

	let rotX = writable(0);
	let rotY = writable(0);
	let rotZ = writable(0);

	let posZ = spring(0, {
		stiffness: 0.4,
		damping: 0.4
	});

	// // wiggle effect
	setInterval(() => {
		if (flipping) return;

		if (flipped) {
			rotX.update((n) => Math.sin(Date.now() / 1000) * 0.1 + Math.PI);
		} else {
			rotX.update((n) => Math.sin(Date.now() / 1000) * 0.1);
		}
		rotX.update((n) => Math.sin(Date.now() / 1000) * 0.1);
	}, 1000 / 60);

	setInterval(() => {
		if (flipping) return;

		rotY.update((n) => Math.cos(Date.now() / 1000) * 0.1);
	}, 1000 / 60);

	setInterval(() => {
		if (flipping) return;

		rotZ.update((n) => Math.sin(Date.now() / 1000) * 0.03);
	}, 1000 / 60);

	// card flip
	let flipped = false;
	let flipping = false;
	function flip() {
		rotX.update((n) => n + Math.PI);
		flipped = !flipped;
	}

	// obj - your object (THREE.Object3D or derived)
	// point - the point of rotation (THREE.Vector3)
	// axis - the axis of rotation (normalized THREE.Vector3)
	// theta - radian value of rotation
	// pointIsWorld - boolean indicating the point is in world coordinates (default = false)
	function rotateAboutPoint(obj, point, axis, theta, pointIsWorld = false) {
		if (pointIsWorld) {
			obj.parent.localToWorld(obj.position); // compensate for world coordinate
		}

		obj.position.sub(point); // remove the offset
		obj.position.applyAxisAngle(axis, theta); // rotate the POSITION
		obj.position.add(point); // re-add the offset

		if (pointIsWorld) {
			obj.parent.worldToLocal(obj.position); // undo world coordinates compensation
		}

		obj.rotateOnAxis(axis, theta); // rotate the OBJECT
	}


	let rotYY = writable(0);
	setInterval(() => {
		rotYY.update((n) => n - 0.01);
	}, 1000 / 60);
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 15]} fov={15}></T.PerspectiveCamera>

<Flex>
	<T.Group rotation.x={$rotYY}> 
		<T.Mesh
			position.z={$posZ}
			position.y={-1}
			rotation.x={$rotX}
			rotation.y={$rotY}
			rotation.z={$rotZ}
			on:pointerover={() => posZ.set(1)}
			on:pointerout={() => posZ.set(0)}
			on:click={() => flip()}
			origin={[0.5, 0.5, 0.5]}
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
