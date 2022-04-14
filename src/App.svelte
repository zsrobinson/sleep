<script>
	import { format, add } from 'date-fns';
	import Circle from "./Circle.svelte";

	let array = [];

	function whenToWakeUp() {
		const now = new Date();
		const fallAsleep = add(now, {minutes: 14});
		let cycles = [];

		for (let i = 0; i < 7; i++) {
			const hrs = Math.floor(i * 1.5)
			const mins = (i * 1.5 - hrs) * 60
			const x = add(fallAsleep, {hours: hrs, minutes: mins})
			cycles[i] = format(x, "p")
		}

		return cycles;
	}

</script>

<section class="section">
	<div class="container">
		<h1 class="title">Sleep Cycle Calculator</h1>

		<p class="block">If I'm going to bed now, when should I wake up?</p>
		<button on:click={ () => array = whenToWakeUp() } class="button">Calculate</button>

		{#each array as cycle, i}
			{#if i != 0}
				<p>{i} sleep {i==1 ? "cycle" : "cycles"}: {cycle}</p>
			{/if}
		{/each}

		<Circle />

	</div>
</section>

<style>

</style>