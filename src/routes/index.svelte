<script lang="ts">
	import { default as dayjs } from 'dayjs';
	import { default as relativeTime } from 'dayjs/plugin/relativeTime.js';
	import { BarLoader } from 'svelte-loading-spinners';

	import CountDown from '$lib/components/CountDown.svelte';
	import TimeDisplay from '$lib/components/TimeDisplay.svelte';

	import { onMount } from 'svelte';
	dayjs.extend(relativeTime);

	let timeDisplayed;
	let isPredicted = false;
	let loaded = false;

	onMount(() => {
		(async () => {
			try {
				let response = await fetch(
					'https://covid-announcement-backend.host.qrl.nz/api/get-announcement-time'
				);
				let body = await response.json();
				console.log('Date pushed', body);
				if (body['date_of_announcement'] !== null) {
					timeDisplayed = dayjs(body['date_of_announcement']);
					if (timeDisplayed.isBefore(dayjs().subtract(0, 'h'))) {
						// Give three hour leeway, then start counting down to the next one
						timeDisplayed = timeDisplayed.add(1, 'd').hour(13);
						isPredicted = true;
					}
					loaded = true;
				}
				loaded = true;
			} catch (e) {
				console.log('It shit itself', e);
			}
		})();
	});
</script>

<main class:loading={!loaded}>
	<div>
		{#if loaded}
			{#if typeof timeDisplayed !== 'undefined'}
				<TimeDisplay bind:timeDisplayed />
				<CountDown bind:timeDisplayed predicted="false" />
				{#if isPredicted}
					<p>*Predicted</p>
				{/if}
			{:else}
				<h3>Hasn't been announced yet</h3>
			{/if}
		{:else}
			<BarLoader size="5" color="currentColor" unit="em" duration="1s" />
		{/if}
	</div>
</main>

<style lang="scss">
	main {
		--social-distance: 2em;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		overflow: hidden;
		margin: 0;
		padding: 0;
		background: repeating-linear-gradient(45deg, #ffcc00, #ffcc00 50px, #ffff 50px, #ffff 100px);
		div {
			background: white;
			width: calc(100vw - var(--social-distance));
			height: calc(100vh - var(--social-distance));
			transition: 0.4s ease width, 0.4s ease height;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			text-align: center;
		}
	}
	main.loading {
		--social-distance: 8em;
	}
	@media (max-width: 750px) {
		main div {
			width: 100%;
			padding: 1em;
		}
	}
</style>
