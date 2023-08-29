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

	const padding = 100;

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
				const jAngle = j * (2 * Math.PI / game.nodeCount);
				context.beginPath();
				context.moveTo(x, y);
				const [jx, jy] = angleToCoords(jAngle, width, height);
				context.lineTo(jx, jy);
				context.stroke();
			}
		}

		context.strokeStyle = "black";
		for (let i = 0; i < game.nodeCount; i++) {
			const angle = i * (2 * Math.PI / game.nodeCount);
			const [x, y] = angleToCoords(angle, width, height);
			context.beginPath()
			context.arc(
				x,
				y,
				10,
				0,
				2 * Math.PI
			)
			context.fill()
		}
	};
</script>

<Canvas width={640} height={640}>
	<Layer {render} />
</Canvas>

<input type="number" bind:value={nodeCount} />
