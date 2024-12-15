<script lang="ts">
	import { onMount } from 'svelte';
	import { Button } from '$lib/components/ui/button';

	// Node positions in a grid-like layout
	const nodePositions = {
		A: { x: 200, y: 100 },
		B: { x: 300, y: 100 },
		C: { x: 400, y: 100 },
		D: { x: 200, y: 200 },
		E: { x: 300, y: 200 },
		F: { x: 400, y: 200 },
		G: { x: 200, y: 300 },
		H: { x: 300, y: 300 },
		I: { x: 400, y: 300 },
		J: { x: 200, y: 400 },
		K: { x: 300, y: 400 },
		L: { x: 400, y: 400 }
	};

	// Connections based on the rewards matrix
	const connections = [
		['A', 'B'],
		['B', 'C'],
		['B', 'F'],
		['C', 'G'],
		['D', 'H'],
		['E', 'I'],
		['F', 'J'],
		['G', 'H'],
		['H', 'L'],
		['I', 'J'],
		['J', 'K'],
		['K', 'L']
	];

	let startNode = 'E';
	let endNode = 'G';
	let currentPath: string[] = $state([]);
	let isLearning = $state(false);

	function startLearning() {
		isLearning = true;
		// Simulate Q-learning steps
		const possiblePath = ['E', 'I', 'J', 'F', 'B', 'C', 'G'];
		let step = 0;

		const interval = setInterval(() => {
			if (step < possiblePath.length) {
				currentPath = possiblePath.slice(0, step + 1);
				step++;
			} else {
				clearInterval(interval);
				isLearning = false;
			}
		}, 1000);
	}

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d');

		function draw() {
			if (!ctx) return;

			ctx.clearRect(0, 0, canvas.width, canvas.height);

			// Draw connections
			connections.forEach(([from, to]) => {
				const fromPos = nodePositions[from as keyof typeof nodePositions];
				const toPos = nodePositions[to as keyof typeof nodePositions];

				ctx.beginPath();
				ctx.moveTo(fromPos.x, fromPos.y);
				ctx.lineTo(toPos.x, toPos.y);
				ctx.strokeStyle = '#666';
				ctx.stroke();
			});

			// Draw path
			if (currentPath.length > 1) {
				ctx.beginPath();
				ctx.moveTo(
					nodePositions[currentPath[0] as keyof typeof nodePositions].x,
					nodePositions[currentPath[0] as keyof typeof nodePositions].y
				);

				for (let i = 1; i < currentPath.length; i++) {
					const pos = nodePositions[currentPath[i] as keyof typeof nodePositions];
					ctx.lineTo(pos.x, pos.y);
				}

				ctx.strokeStyle = '#4CAF50';
				ctx.lineWidth = 3;
				ctx.stroke();
				ctx.lineWidth = 1;
			}

			// Draw nodes
			Object.entries(nodePositions).forEach(([node, pos]) => {
				ctx.beginPath();
				ctx.arc(pos.x, pos.y, 20, 0, Math.PI * 2);

				if (node === startNode) ctx.fillStyle = '#4CAF50';
				else if (node === endNode) ctx.fillStyle = '#F44336';
				else ctx.fillStyle = '#2196F3';

				ctx.fill();
				ctx.stroke();

				ctx.fillStyle = 'white';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'middle';
				ctx.fillText(node, pos.x, pos.y);
			});
		}

		// Initial draw
		draw();

		// Redraw when path changes
		$effect(() => {
			if (currentPath) draw();
		});
	});
</script>

<div class="container mx-auto p-4">
	<h1 class="mb-4 text-2xl font-bold">Q-Learning Pathfinding Visualization</h1>

	<div class="mb-4">
		<Button onclick={startLearning} disabled={isLearning}>
			{isLearning ? 'Learning...' : 'Start Learning'}
		</Button>
	</div>

	<canvas bind:this={canvas} width="600" height="500" class="rounded-lg border border-gray-300"
	></canvas>

	<div class="mt-4">
		<p class="text-sm text-gray-600">
			Start Node (Green): {startNode}<br />
			End Node (Red): {endNode}<br />
			Current Path: {currentPath.join(' → ')}
		</p>
	</div>
</div>

<style>
	canvas {
		background: white;
	}
</style>
