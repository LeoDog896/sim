<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';
	import { Connect } from 'vite';

	type Node = number;
	type Connection = [a: Node, b: Node];
	type Vector = [x: number, y: number];

	let width = 640;
	let height = 640;
	
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

	function isConnection(connection: [number | undefined, number | undefined]): connection is Connection {
		return connection[0] !== undefined && connection[1] !== undefined;
	}

	let hoveredNode: number | undefined = undefined;
	let selectedNodes: [number | undefined, number | undefined] = [undefined, undefined];

	const colors = {
		p1: "#2274A5",
		p2: "#E83F6F",
		sketch: "lightgray"
	}

	$: currentColor = game.connections.length % 2 == 0 ? colors.p1 : colors.p2;

	$: {
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);

			if (Math.sqrt(Math.pow(mouseX - x, 2) + Math.pow(mouseY - y, 2)) < radius) {
				hoveredNode = i;
				break;
			} else {
				hoveredNode = undefined;
			}
		}
	}

    let render: Render;
	$: render = ({ context, width, height }) => {
		context.strokeStyle = "lightgray";
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);
			let startIndex = i;

			// mutates stroke, lineWidth, lineDash
			for (let j = startIndex; j < game.nodeCount; j++) {
				const jAngle = j * (2 * Math.PI / game.nodeCount);
				const [jx, jy] = angleToCoords(jAngle, width, height);

				if (j == hoveredNode || i == hoveredNode || selectedNodes.includes(j) || selectedNodes.includes(i)) {
					context.strokeStyle = currentColor

					if ((selectedNodes.includes(j) && i == hoveredNode) || (selectedNodes.includes(i) && j == hoveredNode)) {
						context.lineWidth = 3;
					} else {
						context.lineWidth = 1;
					}
				} else {
					context.strokeStyle = "lightgray";
					context.lineWidth = 1;
				}

				if (game.connections.some(connection => connection.includes(i) && connection.includes(j))) {
					const index = game.connections.findIndex(connection => connection.includes(i) && connection.includes(j));
					context.setLineDash([]);
					context.lineWidth = 2;
					if (index % 2 == 0) {
						context.strokeStyle = colors.p1;
					} else {
						context.strokeStyle = colors.p2;
					}
				} else {
					context.setLineDash([15, 5]);
				}

				context.beginPath();
				context.moveTo(x, y);
				context.lineTo(jx, jy);
				context.stroke();
			}
		}

		context.setLineDash([]);
		context.strokeStyle = "black";
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);

			if (i == hoveredNode || selectedNodes.includes(i)) {
				context.fillStyle = currentColor;
				context.strokeStyle = currentColor;

				if (selectedNodes.includes(i)) {
					context.beginPath();
					context.arc(
						x, y,
						radius * 3 / 2,
						0, 2 * Math.PI
					);
					context.stroke();
				}
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
	<Canvas on:click={() => {
		if (hoveredNode !== undefined) {
			if (selectedNodes[0] === undefined) {
				selectedNodes[0] = hoveredNode;
			} else if (selectedNodes[1] === undefined) {
				selectedNodes[1] = hoveredNode;
				console.assert(isConnection(selectedNodes), "selectedNodes is not a connection");
				if (isConnection(selectedNodes)) {
					game.connections = [...game.connections, normalizeConnection(selectedNodes)];
					selectedNodes = [undefined, undefined];
				}
			}
		}
	}}
	on:mousemove={({ offsetX, offsetY }) => {
		mouseX = offsetX;
		mouseY = offsetY;
	}} class="canvas {hoveredNode === undefined ? "" : "active"}" {width} {height}>
		<Layer {render} />
	</Canvas>

	<input type="number" bind:value={nodeCount} />
</main>

<style>
	.description {
		max-width: 50ch;
		text-align: center;
	}

	:global(.active) {
		cursor: pointer;
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