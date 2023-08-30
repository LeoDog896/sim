<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

	type Node = number;
	type Connection = [a: Node, b: Node];
	type Vector = [x: number, y: number];
	
	const normalizeConnection = (connection: Connection) => connection.sort((a, b) => a - b);
	let nodeCount = 6;
	$: {
		game.nodeCount = nodeCount;
		game = game;
	}

    interface Game {
		nodeCount: number;
		connections: Connection[];
    }

	let game: Game = {
		nodeCount: 6,
		connections: []
	};

	let mouseX = 0;
	let mouseY = 0;

	const padding = 50;
	const radius = 10;

	function angleToCoords(angle: number, width: number, height: number): Vector {
		return [
			(Math.sin(angle) + 1) * 1/2 * (width - padding) + padding / 2, 
			(Math.cos(angle) + 1) * 1/2 * (height - padding) + padding / 2
		]
	}

    let render: Render;
	$: render = ({ context, width, height }) => {
		context.strokeStyle = "gray";
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);
			for (let j = 0; j < game.nodeCount; j++) {
				if (i == j) continue;
				const jAngle = j * (2 * Math.PI / game.nodeCount);
				const [jx, jy] = angleToCoords(jAngle, width, height);

				if (Math.sqrt(Math.pow(mouseX - jx, 2) + Math.pow(mouseY - jy, 2)) < radius / 2) {
					context.strokeStyle = 'green';
				} else {
					context.strokeStyle = 'gray';
				}

				context.beginPath();
				context.moveTo(x, y);
				context.lineTo(jx, jy);
				context.stroke();
			}
		}

		context.strokeStyle = "black";
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);

			if (Math.sqrt(Math.pow(mouseX - x, 2) + Math.pow(mouseY - y, 2)) < radius / 2) {
				context.fillStyle = 'green';
			} else {
				context.fillStyle = 'black';
			}

			context.beginPath()
			context.arc(
				x,
				y,
				radius,
				0,
				2 * Math.PI
			)
			context.fill()
		}
	};
</script>

<main>
	<div class="description">
		<h1>Sim</h1>
		<p>A simple combinatorial paper and pencil game (<a href="https://en.wikipedia.org/wiki/Sim_(pencil_game)">wikipedia</a>). Every player takes a turn - the first player to make a complete triangle, from dot to dot, loses.</p>
	</div>
	<Canvas on:mousemove={({ offsetX, offsetY }) => {
		mouseX = offsetX;
		mouseY = offsetY;
	}} class="canvas" width={640} height={640}>
		<Layer {render} />
	</Canvas>

	<input type="number" bind:value={nodeCount} />
</main>

<style>
	.description {
		max-width: 50ch;
		text-align: center;
	}

	main {
		display: flex;
		justify-content: space-between;
		align-items: center;
		flex-direction: column;
		height: 100%;
		margin: 2rem;
	}

	:global(.canvas) {
		border: 1px solid black;
		border-radius: 1rem;
	}
</style>