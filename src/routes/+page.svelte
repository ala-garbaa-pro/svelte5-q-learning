<script lang="ts">
	import { onMount } from 'svelte';
	import { Button } from '$lib/components/ui/button';
	import * as Select from '$lib/components/ui/select';
	import { Badge } from '$lib/components/ui/badge';
	import { Progress } from '$lib/components/ui/progress';


	import nodeSoundFile from '$lib/sounds/node.wav';
	import successSoundFile from '$lib/sounds/success.wav';
	import clickSoundFile from '$lib/sounds/click.wav';
    
	import ModeToggle from '$lib/components/mode-toggle.svelte';

	// Canvas configuration
	const CANVAS_WIDTH = 600;
	const CANVAS_HEIGHT = 600;
	const NODE_RADIUS = 40;
	const NODE_SPACING_X = 100;
	const NODE_SPACING_Y = 150;
	const INITIAL_X = 100;
	const INITIAL_Y = 50;
	const NODE_FONT_SIZE = 20;

	// Node positions in a grid-like layout
	const nodePositions = {
		A: { x: INITIAL_X, y: INITIAL_Y },
		B: { x: INITIAL_X + NODE_SPACING_X * 2, y: INITIAL_Y },
		C: { x: INITIAL_X + NODE_SPACING_X * 4, y: INITIAL_Y },
		D: { x: INITIAL_X, y: INITIAL_Y + NODE_SPACING_Y },
		E: { x: INITIAL_X + NODE_SPACING_X * 2, y: INITIAL_Y + NODE_SPACING_Y },
		F: { x: INITIAL_X + NODE_SPACING_X * 4, y: INITIAL_Y + NODE_SPACING_Y },
		G: { x: INITIAL_X, y: INITIAL_Y + NODE_SPACING_Y * 2 },
		H: { x: INITIAL_X + NODE_SPACING_X * 2, y: INITIAL_Y + NODE_SPACING_Y * 2 },
		I: { x: INITIAL_X + NODE_SPACING_X * 4, y: INITIAL_Y + NODE_SPACING_Y * 2 },
		J: { x: INITIAL_X, y: INITIAL_Y + NODE_SPACING_Y * 3 },
		K: { x: INITIAL_X + NODE_SPACING_X * 2, y: INITIAL_Y + NODE_SPACING_Y * 3 },
		L: { x: INITIAL_X + NODE_SPACING_X * 4, y: INITIAL_Y + NODE_SPACING_Y * 3 }
	};

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

	// Game state
	let startNode = $state('E');
	let endNode = $state('G');
	let currentPath: string[] = $state([]);
	let isLearning = $state(false);
	let score = $state(0);
	let learningProgress = $state(0);
	let selectedDifficulty = $state('easy');
	let gameStatus = $state('ready'); // 'ready', 'learning', 'complete'
	let iterations = $state(0);

	// Sound effects
	let nodeSound: HTMLAudioElement;
	let successSound: HTMLAudioElement;
	let clickSound: HTMLAudioElement;

	onMount(() => {
		nodeSound = new Audio(nodeSoundFile);
		successSound = new Audio(successSoundFile);
		clickSound = new Audio(clickSoundFile);
	});

	// Q-Learning parameters based on difficulty
	const difficultySettings = {
		easy: { learningRate: 0.9, discount: 0.75, iterations: 500 },
		medium: { learningRate: 0.7, discount: 0.8, iterations: 1000 },
		hard: { learningRate: 0.5, discount: 0.9, iterations: 2000 }
	};

	function calculateReward(path: string[]) {
		return 1000 - path.length * 100;
	}

	async function startLearning() {
		gameStatus = 'learning';
		isLearning = true;
		score = 0;
		learningProgress = 0;
		iterations = 0;

		const settings = difficultySettings[selectedDifficulty as keyof typeof difficultySettings];

		// Simulate Q-learning steps with animation
		const possiblePaths = [
			['E', 'I', 'J', 'F', 'B', 'C', 'G'],
			['E', 'I', 'J', 'K', 'H', 'G'],
			['E', 'I', 'J', 'F', 'B', 'C', 'G']
		];

		const selectedPath = possiblePaths[Math.floor(Math.random() * possiblePaths.length)];

		for (let i = 0; i < selectedPath.length; i++) {
			currentPath = selectedPath.slice(0, i + 1);
			nodeSound.play();
			iterations++;
			learningProgress = (iterations / settings.iterations) * 100;
			await new Promise((resolve) => setTimeout(resolve, 1000));
		}

		score = calculateReward(currentPath);
		successSound.play();
		gameStatus = 'complete';
		isLearning = false;
	}

	function resetGame() {
		clickSound.play();
		currentPath = [];
		score = 0;
		learningProgress = 0;
		iterations = 0;
		gameStatus = 'ready';
	}

	let canvas: HTMLCanvasElement;

	function isDarkMode() {
		return document.documentElement.classList.contains('dark');
	}

	onMount(() => {
		const ctx = canvas.getContext('2d');

		function draw() {
			if (!ctx) return;

			ctx.clearRect(0, 0, canvas.width, canvas.height);

			// Draw connections with gradient
			connections.forEach(([from, to]) => {
				const fromPos = nodePositions[from as keyof typeof nodePositions];
				const toPos = nodePositions[to as keyof typeof nodePositions];

				const gradient = ctx.createLinearGradient(fromPos.x, fromPos.y, toPos.x, toPos.y);
				if (isDarkMode()) {
					gradient.addColorStop(0, '#4a5568');
					gradient.addColorStop(1, '#718096');
				} else {
					gradient.addColorStop(0, '#666');
					gradient.addColorStop(1, '#999');
				}

				ctx.beginPath();
				ctx.moveTo(fromPos.x, fromPos.y);
				ctx.lineTo(toPos.x, toPos.y);
				ctx.strokeStyle = gradient;
				ctx.lineWidth = 2;
				ctx.stroke();
			});

			// Draw current path with glow effect
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

				ctx.shadowColor = '#4CAF50';
				ctx.shadowBlur = 10;
				ctx.strokeStyle = '#4CAF50';
				ctx.lineWidth = 4;
				ctx.stroke();
				ctx.shadowBlur = 0;
				ctx.lineWidth = 2;
			}

			// Draw nodes with animation
			Object.entries(nodePositions).forEach(([node, pos]) => {
				ctx.beginPath();
				ctx.arc(pos.x, pos.y, NODE_RADIUS, 0, Math.PI * 2);

				if (node === startNode) {
					ctx.fillStyle = '#4CAF50';
					ctx.shadowColor = '#4CAF50';
				} else if (node === endNode) {
					ctx.fillStyle = '#F44336';
					ctx.shadowColor = '#F44336';
				} else {
					ctx.fillStyle = isDarkMode() ? '#60A5FA' : '#2196F3';
					ctx.shadowColor = isDarkMode() ? '#60A5FA' : '#2196F3';
				}

				ctx.shadowBlur = currentPath.includes(node) ? 15 : 5;
				ctx.fill();
				ctx.shadowBlur = 0;
				ctx.strokeStyle = isDarkMode() ? '#4a5568' : '#000';
				ctx.stroke();

				ctx.font = `${NODE_FONT_SIZE}px sans-serif`;
				ctx.fillStyle = isDarkMode() ? '#e5e7eb' : 'white';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'middle';
				ctx.fillText(node, pos.x, pos.y);
			});
		}

		function handleResize() {
			if (!canvas) return;
			const container = canvas.parentElement;
			if (!container) return;
			
			const containerWidth = container.clientWidth;
			const scale = containerWidth / CANVAS_WIDTH;
			
			canvas.style.width = '100%';
			canvas.style.height = 'auto';
			
			draw();
		}

		// Initial draw
		draw();

		// Redraw when path changes or when theme changes
		$effect(() => {
			if (currentPath) draw();
		});

		// Add theme change observer
		const observer = new MutationObserver(() => {
			draw();
		});

		observer.observe(document.documentElement, {
			attributes: true,
			attributeFilter: ['class']
		});

		window.addEventListener('resize', handleResize);
		handleResize();
		
		return () => {
			observer.disconnect();
			window.removeEventListener('resize', handleResize);
		};
	});
</script>

<div class="container mx-auto p-4 max-w-full">
	<div class="mb-6 flex flex-col sm:flex-row items-center justify-between gap-4">
		<h1 class="text-xl sm:text-2xl font-bold">Q-Learning Pathfinding Game</h1>
		<div class="flex flex-wrap items-center gap-4">
			<ModeToggle />
			<Select.Root
				type="single"
				value={selectedDifficulty}
				onValueChange={(value) => (selectedDifficulty = value)}
			>
				<Select.Trigger class="w-[140px] sm:w-[180px]">
					{selectedDifficulty ?? 'Select difficulty'}
				</Select.Trigger>
				<Select.Content>
					<Select.Item value="easy">Easy</Select.Item>
					<Select.Item value="medium">Medium</Select.Item>
					<Select.Item value="hard">Hard</Select.Item>
				</Select.Content>
			</Select.Root>

			<Badge variant={gameStatus === 'complete' ? 'default' : 'secondary'}>
				Score: {score}
			</Badge>
		</div>
	</div>

	<div class="mb-4 flex flex-wrap gap-4">
		<Button onclick={startLearning} disabled={isLearning} variant="default" class="w-full sm:w-auto">
			{isLearning ? 'Learning...' : 'Start Learning'}
		</Button>
		<Button onclick={resetGame} variant="outline" disabled={isLearning} class="w-full sm:w-auto">Reset</Button>
	</div>

	<div class="mb-4">
		<Progress value={learningProgress} />
	</div>

	<div class="w-full overflow-hidden">
		<div class="canvas-container">
			<canvas
				bind:this={canvas}
				width={CANVAS_WIDTH}
				height={CANVAS_HEIGHT}
				class="w-full h-auto rounded-lg border border-gray-300 dark:border-gray-700 shadow-lg dark:bg-gray-950 bg-white"
			></canvas>
		</div>
	</div>

	<div class="mt-4 space-y-2 text-md">
		<p class="text-gray-600 dark:text-gray-400">
			Start Node (Green): {startNode} | End Node (Red): {endNode}
		</p>
		<p class="text-gray-600 dark:text-gray-400">Current Path: {currentPath.join(' → ')}</p>
		<p class="text-gray-600 dark:text-gray-400">Iterations: {iterations}</p>
	</div>

	<div class="mt-8 text-center text-sm text-gray-500 dark:text-gray-400">
		<p>
			Developed by <a 
				href="https://github.com/ala-garbaa-pro" 
				target="_blank" 
				rel="noopener noreferrer" 
				class="font-medium hover:text-gray-700 dark:hover:text-gray-300 transition-colors"
			>
				Ala GARBAA
			</a>
		</p>
        
		<p class="mt-1">
			<a 
				href="https://github.com/ala-garbaa-pro/svelte5-q-learning" 
				target="_blank" 
				rel="noopener noreferrer" 
				class="inline-flex items-center gap-1 hover:text-gray-700 dark:hover:text-gray-300 transition-colors"
			>
				<svg class="w-4 h-4" viewBox="0 0 24 24" fill="currentColor">
					<path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
				</svg>
				View on GitHub
			</a>
		</p>
		<p class="mt-1">
			Licensed under <a 
				href="https://github.com/ala-garbaa-pro/svelte5-q-learning/blob/main/LICENSE" 
				target="_blank" 
				rel="noopener noreferrer" 
				class="font-medium hover:text-gray-700 dark:hover:text-gray-300 transition-colors"
			>
				MIT License
			</a>
		</p>
	</div>
</div>

<style>
	canvas {
		background: white;
	}
	
	.canvas-container {
		position: relative;
		width: 100%;
		max-width: 600px;
		margin: 0 auto;
	}
	
	@media (max-width: 640px) {
		.canvas-container {
			max-width: 100%;
		}
	}
</style>
