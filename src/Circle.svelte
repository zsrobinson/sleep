<script>
import { onMount, afterUpdate } from "svelte";

let mouse = {};
let mouseRel = {};
let circleDOM;
let circle = {};
let theta;
let dot = {};
let dot2 = {};
let dotDOM, dot2DOM;
let dotTouch = false;
let offset = -Math.PI / 4;

onMount(() => {
	var circlePos = circleDOM.getBoundingClientRect();
	circle.borderWidth = parseInt(window.getComputedStyle(circleDOM).outlineTopWidth);
	circle.r = (circlePos.width / 2) /* - circle.borderWidth */;
	circle.x = circlePos.left + circle.r;
	circle.y = circlePos.top + circle.r;

	dotDOM.style.left = 2 * circle.r + "px"
	dotDOM.style.top = circle.r + "px"
});

function mouseMove(e) {

	if (!dotTouch) return

	mouse = {x: e.clientX, y: e.clientY}

	mouseRel.x = mouse.x - circle.x
	mouseRel.y = circle.y - mouse.y
	theta = Math.atan(mouseRel.y / mouseRel.x)

	// quadrant 2
	if (mouseRel.x < 0 && mouseRel.y > 0) {
		theta += Math.PI
	// quadrant 3
	} else if (mouseRel.x < 0 && mouseRel.y <= 0) {
		theta += Math.PI
	// quadrant 4
	} else if (mouseRel.x > 0 && mouseRel.y <= 0) {
		theta += 2 * Math.PI
	}

	dot.x = circle.r * Math.cos(theta)
	dot.y = circle.r * Math.sin(theta)
	
	dot2.x = circle.r * Math.cos(theta + offset)
	dot2.y = circle.r * Math.sin(theta + offset)

	dotDOM.style.left = circle.r + dot.x + "px"
	dotDOM.style.top = circle.r - dot.y + "px"

	dot2DOM.style.left = circle.r + dot2.x + "px"
	dot2DOM.style.top = circle.r - dot2.y + "px"

	console.log("running")
}


</script>

<div on:mousemove={mouseMove} on:mouseup={() => dotTouch = false}>
	<div class="circle" bind:this={circleDOM}>
	<div class="dot" bind:this={dotDOM} on:mousedown={() => dotTouch = true} ></div>
	<div class="dot" bind:this={dot2DOM} ></div></div>

	Mouse: {mouse.x}, {mouse.y} <br>
	Circle: {circle.x}, {circle.y} <br>
	MouseRel: {mouseRel.x}, {mouseRel.y} <br>
	{theta*180 / Math.PI}
</div> 

<style>
:root {
  --borderRadius: .5rem;
  --circleDiameter: 25rem;
  --dotDiameter: 3rem;
}

.circle {
	height: var(--circleDiameter);
	width: var(--circleDiameter);
	outline: var(--borderRadius) solid black;
	outline-offset: calc(var(--borderRadius) * -0.5);
	border-radius: 50%;
	position: relative;
	margin: calc(var(--borderRadius) /2 + var(--dotDiameter) / 2);
}

.dot {
	height: var(--dotDiameter);
	width: var(--dotDiameter);
	background-color: white;
	border: var(--borderRadius) solid black;
	border-radius: 50%;
	position: absolute;
	transform: translate(-50%, -50%);
}
</style>