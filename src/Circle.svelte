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
	for (let i = 0; i < dots.length; i++) {
		if (dots[i].pressed) {

			// update circle values in case window changed size
			circle.update()

			// set and calculate mice values
			mouse = {x: e.clientX, y: e.clientY}
			mouseRel.x = mouse.x - circle.x
			mouseRel.y = circle.y - mouse.y

			// calculate theta
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

			// offset theta and update dot
			theta += dots[i].offset
			dots[i].update();
		}
	}

}

function radToTime(rad) {
	const piOverSixes = rad / (Math.PI / 6)
	let hours = -1 * piOverSixes + 15
	if (hours > 12) { hours -= 12; }
	return hours;
}

function timeToRad(time) {
	let piOverSixes = -1 * time + 15
	if (piOverSixes >= 12) { piOverSixes -= 12; }
	return piOverSixes * (Math.PI / 6);
}

</script>

<div on:pointermove={(e) => {mouseMove(e)}} on:pointerup={() => circle.mouseUp()}>
	<div class="circle" bind:this={circle.element}>
		<div class="dot" bind:this={dots[0].element} on:pointerdown={() => dots[0].pressed = true} ></div>
		<div class="dot" bind:this={dots[1].element} on:pointerdown={() => dots[1].pressed = true} ></div>
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

@media (max-width:600px) {
	:root {
		--circleDiameter: 18rem;
		}
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

div {
	touch-action: none
}




</style>