<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

    interface Game {
        
    }

    let render: Render;
	$: render = ({ context, width, height }) => {
		context.fillStyle = `hsl(${10 / 40}, 100%, 50%)`;
		context.beginPath();
		context.arc((10 / 4) % width, (1 / 4) % height, 100, 0, Math.PI * 2);
		context.fill();
	};
</script>

<Canvas width={640} height={640}>
	<Layer {render} />
</Canvas>
