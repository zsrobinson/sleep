<script>
import { onMount } from "svelte";

let mouse = {x: 0, y: 0}
let mouseRel = {x: 0, y: 0}
let theta = Math.PI; // angle if offset is 0

class Dot {
	constructor(id, offset) {
		this.element;
		this.offset = offset;
		this.pressed = false;
		this.x = 0;
		this.y = 0;
		this.id = id;
		this.alpha = 0; // theta - offset
	}
	update(indirect=false) {
		// if (!this.element) return

		this.alpha = theta - this.offset

		this.y = circle.r * Math.sin(this.alpha)
		this.x = circle.r * Math.cos(this.alpha)
		this.element.style.left = circle.r + this.x + "px"
		this.element.style.top = circle.r - this.y + "px"

		if (indirect) return
		// update the other dot in "indirect" mode
		for (let i = 0; i < dots.length; i++) {
			if (i != this.id) {
				dots[i].update(true);
			}
		}

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
		dots[0].pressed = false
		dots[1].pressed = false
	}
}

let dots = [ new Dot(0, 0), new Dot(1, Math.PI / 4) ]
let circle = new Circle()

// initialize values
onMount(() => { 
	circle.update();
	for (let i = 0; i < dots.length; i++) {
		dots[i].update(true);
	}
})

function mouseMove(e) {
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

	// add offset
	for (let i = 0; i < dots.length; i++) {
		if (dots[i].pressed) {
			theta += dots[i].offset
			dots[i].update();
		}
	}

}

</script>

<div on:mousemove={(e) => {mouseMove(e)}} on:mouseup={() => circle.mouseUp()}>
	<div class="circle" bind:this={circle.element}>
		<div class="dot" bind:this={dots[0].element} on:mousedown={() => dots[0].pressed = true} style="background-color: white; "></div>
		<div class="dot" bind:this={dots[1].element} on:mousedown={() => dots[1].pressed = true} style="background-color: gray; "></div>
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
	border: var(--borderRadius) solid black;
	border-radius: 50%;
	position: absolute;
	transform: translate(-50%, -50%);
}
</style>