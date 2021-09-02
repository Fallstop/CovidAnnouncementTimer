<script lang="ts">
	import { default as dayjs } from 'dayjs';
	import { default as relativeTime } from 'dayjs/plugin/relativeTime.js';
	import { BarLoader } from 'svelte-loading-spinners';

	import CountDown from '$lib/components/CountDown.svelte';
	import TimeDisplay from '$lib/components/TimeDisplay.svelte';
	import YouTube from '$lib/components/YouTube.svelte';

	import { onMount } from 'svelte';
	import { debug } from 'svelte/internal';
	dayjs.extend(relativeTime);

	let timeDisplayed = dayjs();
	let isPredicted = false;
	let loaded = false;
	$: isInPast = timeDisplayed.isBefore(dayjs().subtract(1, 'h'));
	let liveVideoId;
	let pastVideoIDs = [];

	function sleep(ms) {
		return new Promise((resolve) => setTimeout(resolve, ms));
	}

	onMount(() => {
		// Fix issue with scroll position glitching on reload
		document.body.scrollTop = document.documentElement.scrollTop = 0;

		(async () => {
			try {
				const minLoadingTime = new Date(Date.now() + 500);
				let nextTimeResp = await fetch(
					'https://covid-announcement-backend.host.qrl.nz/api/get-announcement-time'
				);
				let liveVideoResp = await fetch(
					'https://covid-announcement-backend.host.qrl.nz/api/get-youtube-live'
				);
				let historicVideoResp = await fetch(
					'https://covid-announcement-backend.host.qrl.nz/api/get-historic-youtube-live'
				);
				let nextTime = await nextTimeResp.json();
				let liveVideo = await liveVideoResp.json();
				let pastVideos = await historicVideoResp.json();
				pastVideoIDs = pastVideos['youtube_video_ids'];
				const timeLeft = minLoadingTime.getTime() - Date.now();
				if (timeLeft > 0) {
					await sleep(timeLeft);
				}
				liveVideoId = liveVideo['youtube_video_id'];
				console.log('Date pushed', nextTime);
				if (nextTime['date_of_announcement'] !== null) {
					timeDisplayed = dayjs(nextTime['date_of_announcement']);
					// Can't use reactive variable because updates are batched
					if (timeDisplayed.isBefore(dayjs().subtract(1, 'h'))) {
						// Give one hour leeway, then start counting down to the next one
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

<div class="content" class:loading={!loaded}>
	<div class="frameOuter">
		<div class="frameInner" />
	</div>
	<main>
		<div class="countdown">
			{#if loaded}
				{#if typeof timeDisplayed !== 'undefined'}
					<TimeDisplay bind:timeDisplayed />
					<CountDown bind:timeDisplayed predicted={isPredicted} />
				{:else}
					<h3>Hasn't been announced yet</h3>
				{/if}
			{:else}
				<BarLoader size="5" color="currentColor" unit="em" duration="1s" />
			{/if}
			<div class="spacer" />
		</div>
	</main>
	<div class="updates {liveVideoId ? 'live' : ''}">
		<h3 class:invisible={!isInPast}>Previous updates</h3>
		<h3 class:invisible={!liveVideoId && loaded}>Live update:</h3>
		<YouTube
			size="lg"
			placeholder={!loaded || !liveVideoId}
			videoId={liveVideoId}
			enabled={liveVideoId || !loaded}
		/>
		<div class="more">
			{#if !isInPast}<h3>Previous updates</h3>{/if}
			<div class="gallery">
				{#each pastVideoIDs as id}
					<YouTube videoId={id} />
				{/each}
			</div>
		</div>
	</div>
</div>

<style lang="scss">
	.content {
		--social-distance: 8em;

		.frameOuter {
			position: fixed;
			z-index: 1;
			display: flex;
			align-items: center;
			justify-content: center;
			height: 100vh;
			width: 100vw;
			overflow: hidden;
			margin: 0;
			padding: 0;
			background: repeating-linear-gradient(-45deg, #ffcc00, #ffcc00 50px, #ffff 50px, #ffff 100px);
			.frameInner {
				background: white;
				width: calc(100vw - var(--social-distance));
				height: calc(100vh - var(--social-distance));
				transition: 0.4s ease width, 0.4s ease height;
			}
		}

		main {
			position: relative;
			z-index: 10;
			display: flex;
			align-items: center;
			justify-content: center;
			flex-direction: column;
			min-height: 100vh;
			margin-left: calc(var(--social-distance) / 2);
			margin-right: calc(var(--social-distance) / 2);
			.countdown {
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;
				text-align: center;
			}

			.spacer {
				padding-top: 25em;
			}
		}

		.updates {
			position: relative;
			z-index: 10;
			margin-top: -40rem;
			text-align: center;
			background: linear-gradient(to bottom, transparent 0, white 30em);
			padding-bottom: 5rem;

			h3 {
				font-size: 2em;
			}

			.invisible {
				visibility: hidden;
			}

			.more.live {
				padding-top: var(--social-distance);
			}
			.more {
				.gallery {
					display: flex;
					align-items: center;
					flex-wrap: wrap;
					row-gap: 2em;
					column-gap: 2em;
					max-width: 62rem;
					margin: auto;
				}
			}
		}
	}
	.content.loading {
		--social-distance: 4em;
	}
	@media (max-width: 750px) {
		main div {
			width: 100%;
			padding: 1em;
		}
	}
</style>
