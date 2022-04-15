<script>
import { onMount } from "svelte";

let mouse = {x: 0, y: 0}
let mouseRel = {x: 0, y: 0}
let theta = 0; //angle if offset is 0

class Dot {
	constructor(offset) {
		this.element;
		this.offset = offset;
		this.pressed = false;
		this.x = 0;
		this.y = 0;
	}
	update() {
		if (!this.element) return
		this.x = circle.r * Math.cos(theta)
		this.y = circle.r * Math.sin(theta)
		this.element.style.left = circle.r + this.x + "px"
		this.element.style.top = circle.r - this.y + "px"
	}
}

class Circle {
	constructor(element) {
		this.element = element;
		this.x = 0;
		this.y = 0;
		this.r = 0;
		this.border = 0;
		this.pos = {};
	}
	update() {
		if (!this.element) return
		this.pos = this.element.getBoundingClientRect();
		this.borderWidth = parseInt(window.getComputedStyle(this.element).outlineTopWidth);
		this.r = (this.pos.width / 2);
		this.x = this.pos.left + this.r;
		this.y = this.pos.top + this.r;
	}
	mouseUp() {
		dotOne.pressed = false
		dotTwo.pressed = false
	}
}

let dotOne = new Dot(0)
let dotTwo = new Dot(0 - Math.PI / 4)
let circle = new Circle()

onMount(() => { circle.update() })

$: {
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

	if (dotOne.pressed) { dotOne.update(); }
	if (dotTwo.pressed) { dotTwo.update(); }

}

</script>

<div on:mousemove={(e) => {mouse = {x: e.clientX, y: e.clientY}}} on:mouseup={() => circle.mouseUp()}>
	<div class="circle" bind:this={circle.element}>
		<div class="dot" bind:this={dotOne.element} on:mousedown={() => dotOne.pressed = true} ></div>
		<div class="dot" bind:this={dotTwo.element} on:mousedown={() => dotTwo.pressed = true} ></div>
	</div>

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