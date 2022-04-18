<script>
import { onMount } from "svelte";
import { format, add} from "date-fns";

let mouse = {x: 0, y: 0}
let mouseRel = {x: 0, y: 0}
let theta;

let cycles = 5; // integer from 1 to 7
let timeToFallAsleep = 14; // integer in minutes

$: {
	dots[1].offset = hoursToOffset(1.5 * cycles + timeToFallAsleep / 60);
	dots[1].update(true);
}

const now = new Date();
const nowHours = now.getHours()
const nowMins = now.getMinutes()
theta = hoursToRad(nowHours + nowMins / 60)


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
		if (!this.element) return

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
		this.borderWidth = parseInt(window.getComputedStyle(this.element).borderTopWidth);
		this.r = (this.pos.width / 2) - this.borderWidth;
		this.x = this.pos.left + this.r;
		this.y = this.pos.top + this.r;
	}
	mouseUp() {
		dots[0].pressed = false
		dots[1].pressed = false
	}
}

let dots = [ new Dot(0, 0), new Dot(1, hoursToOffset(1.5 * cycles)) ]
let circle = new Circle()

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

function radToHours(rad) {
	const piOverSixes = rad / (Math.PI / 6);
	let hours = -1 * piOverSixes + 15;
	if (hours > 12) { hours -= 12; }
	let minutes = hours - Math.floor(hours);
	minutes *= 60;
	hours = Math.floor(hours);

	let time = new Date();
	time.setHours(hours);
	time.setMinutes(minutes);
	const formattedTime = format(time, "p").slice(0, -3);

	// had to insert a zero width joiner so that iOS wouldn't format as clickable link
	// return formattedTime.slice(0,1) + "&zwj;" + formattedTime.slice(1)
	// nevermind? there was something weird before but now it's good...

	return formattedTime
}

function hoursToRad(time) {
	let piOverSixes = -1 * time + 15
	if (piOverSixes >= 12) { piOverSixes -= 12; }
	return piOverSixes * (Math.PI / 6);
}

function hoursToOffset(time) {
	return time * (Math.PI/6)
}

function changeCycles(value) {
	if (cycles > 1 && value == -1) {
		cycles--;
	} else if (cycles < 7 && value == 1) {
		cycles++;
	}
}

function initialize() {
	circle.update();
	for (let i = 0; i < dots.length; i++) {
		dots[i].update(true);
	}
}
onMount(initialize);

// sometimes the icons aren't aligned when
// first loaded, hopefully this will help
setTimeout(initialize, 50);

</script>

<main on:pointermove={(e) => {mouseMove(e)}} on:pointerup={() => circle.mouseUp()} >

	<header>
		<h1>Circular Sleep</h1>
	</header>

	<section>

		<div class="timeDisplay">
			<div class="bedTime">
				<h2>
					<i class="fa-solid fa-moon"></i>
					Bedtime
				</h2>
				<p>{@html radToHours(theta - dots[0].offset)}</p>
			</div>
			<div class="wakeTime">
				<h2>
					<i class="fa-solid fa-sun"></i>
					Wakeup
				</h2>
				<p>{@html radToHours(theta - dots[1].offset)}</p>
			</div>
		</div>

		<div class="circle" bind:this={circle.element}>
			<div class="dot" bind:this={dots[0].element} on:pointerdown={() => dots[0].pressed = true}>
				<i class="fa-solid fa-moon"></i>
			</div>
			<div class="dot" bind:this={dots[1].element} on:pointerdown={() => dots[1].pressed = true}>
				<i class="fa-solid fa-sun"></i>
			</div>

			<div class="cycleInput">
				<h2>Sleep Cycles:</h2>
				<div>
					<div class="cycleInputButton" on:click={() => changeCycles(-1)}>
						<i class="fa-solid fa-minus"></i>
					</div>
					<p>{cycles}</p>
					<div class="cycleInputButton" on:click={() => changeCycles(1)}>
						<i class="fa-solid fa-plus"></i>
					</div>
				</div>
				<p>Equal to {cycles * 1.5} hours</p>
				<p>plus {timeToFallAsleep} minutes</p>
			</div>
		</div>

	<section>

	<section>
		<button onclick="window.open('https://github.com/zsrobinson/sleep')">Learn More</button>
	</section>

	<footer>
		<p>Created by <a href="https://zsrobinson.com"> Zachary Robinson </a> </p>
		<p>View this project on <a href="https://github.com/zsrobinson/sleep"> GitHub </a> </p>
	</footer>

</main>

<style>
:root {
  --borderWidth: 0.75rem;
  --circleDiameter: 18rem;
  --dotDiameter: 3rem;
}

.circle {
	height: var(--circleDiameter);
	width: var(--circleDiameter);
	/* outline: var(--borderWidth) solid var(--tertiary);
	outline-offset: calc(var(--borderWidth) * -0.5); */
	border: var(--borderWidth) solid var(--tertiary);
	border-radius: 50%;
	position: relative;
	margin: calc(var(--borderWidth) /2 + var(--dotDiameter) / 2);

	/*For aligning cycle input to center*/
	display: flex;
	justify-content: center;
	align-items: center;
}

.dot {
	height: var(--dotDiameter);
	width: var(--dotDiameter);
	border: var(--borderWidth) solid var(--theme);
	border-radius: 50%;
	position: absolute;
	transform: translate(-50%, -50%);
	background-color: var(--secondary); 
	cursor: default; /* hopefully disables dragging */

	display: flex;
	justify-content: center;
	align-items: center;
}

.dot > i {
	color: var(--theme);
	font-size: 2rem;
}

.circle, .dot {
	touch-action: none
}

main {
	display: flex;
	flex-flow: column;
	height: 100%;
	align-items: center
}

header {
  flex: 0 1 auto;
}

section {
	flex: 1 1 auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
}

footer {
	flex: 0 1 auto;
	color: var(--secondary);
	display: flex;
	flex-wrap: wrap;
    justify-content: center;
	margin: 1rem 0;
	font-size: 0.75rem;
}

footer > p {
	color: var(--secondary);
	padding: 0 1rem;
}
a { color: var(--secondary); }
a:hover { color: var(--primary); }

h1 {
	padding: 1rem 0;
}

.timeDisplay {
	display: flex;
	flex-direction: row;
	justify-content: center;
	padding: .5rem 0;
}

.timeDisplay > div {
	padding: 0 2rem;
}

.timeDisplay > div > h2 {
	text-align: center;
}

.timeDisplay > div > p {
	font-size: 3rem;
	text-align: center;
}

.cycleInput > div {
	display: flex;
	align-items: center;
}

.cycleInput > h2 {
	text-align: center;
	font-weight: bold;
	font-size: 1.25rem;
}

.cycleInput > p {
	text-align: center;
}

.cycleInput > div > p {
	font-size: 3rem;
	background-color: var(--entry);
	border: none;
	border-bottom: 0.5rem solid var(--tertiary);
	border-radius: 0.5rem;
	text-align: center;
	width: 4rem;
	font-weight: bold;
	margin: 0.75rem 1rem;
}

.cycleInputButton {
	height: var(--dotDiameter);
	width: var(--dotDiameter);
	border-radius: 50%;
	background-color: var(--tertiary); 
	cursor: pointer;
	display: flex;
	justify-content: center;
	align-items: center;
}

.cycleInputButton > i {
	color: var(--primary);
	font-size: 2rem;
}

button {
	background-color: var(--entry);
	border: 0.25rem solid var(--tertiary);
	border-radius: 0.25rem;
	text-align: center;
	font-weight: bold;
	padding: 0.25rem 1rem;
	cursor: pointer;
	font-size: 1rem;
	width: 10em;
}

</style>