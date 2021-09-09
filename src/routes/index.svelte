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
	let anyPastVideos = true;

	let backendServerAddress = "https://covid-announcement-backend.host.qrl.nz/api/";
	// let backendServerAddress = "http://localhost:8000/api/";


	function sleep(ms) {
		return new Promise((resolve) => setTimeout(resolve, ms));
	}

	async function loadData() {
		try {
			const minLoadingTime = new Date(Date.now() + 500);
			let nextTimeResp = await fetch(
				backendServerAddress+'get-announcement-time'
			);
			let liveVideoResp = await fetch(
				backendServerAddress+'get-youtube-live'
			);

			let historicVideoResp = await fetch(
				backendServerAddress+'get-historic-youtube-live'
			);
			let nextTime = await nextTimeResp.json();
			let liveVideo = await liveVideoResp.json();
			let pastVideos = await historicVideoResp.json();
			pastVideoIDs = pastVideos['youtube_video_ids'];

			// Check if there is any past videos
			anyPastVideos = pastVideoIDs.map((x) => x !== null).reduce((acu, cur) => acu || cur);

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
					// Try to add a day to the last found time, but if that's in the past then we'll need to give up
					if (!timeDisplayed.add(1, 'd').hour(13).isBefore(dayjs().subtract(1, 'h'))) {
						// Nope, we're good. Give one hour leeway, then start counting down to the next one
						timeDisplayed = timeDisplayed.add(1, 'd').hour(13);
						isPredicted = true;
					} else {
						// Uh oh! Our predicted time was wrong, so either the folks at MoH skipped a day or we're a bit lost.
						console.log("Tried to add a day and predict the next one, but it wasn't enough!");
					}
				}
				loaded = true;
			} else {
				console.error(
					'Warning: Backend API is unstable or down! Loading up some strong assumptions'
				);
				// Things went wrong, let's start assuming
				timeDisplayed = dayjs().hour(13).minute(0).second(0);
				if (timeDisplayed.isBefore(dayjs().subtract(1, 'h'))) {
					timeDisplayed = timeDisplayed.add(1, 'd');
					isPredicted = true;
				}
				isPredicted = true;
			}
			loaded = true;
		} catch (e) {
			console.log('It shit itself', e);
		}
	}

	onMount(() => {
		// Fix issue with scroll position glitching on reload
		document.body.scrollTop = document.documentElement.scrollTop = 0;
		loadData();
		// reload data once per 5 seconds
		setInterval(loadData, 5000);
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
					{#if !liveVideoId}
						<CountDown bind:timeDisplayed predicted={isPredicted} />
					{:else}
						<h2 class="live">(happening now)</h2>
					{/if}
				{:else}
					<h3>Hasn't been announced yet</h3>
				{/if}
			{:else}
				<div class="loading-countdown-container">
					<BarLoader size="5" color="currentColor" unit="em" duration="1s" />
				</div>
			{/if}
			<p>
				Looking for the NZ COVID daily update? We are too. By our estimates, the next update will be
				at {timeDisplayed.format('h:mm A')}, {timeDisplayed.fromNow()}. Until then,
				{#if anyPastVideos}
					browse through the previous updates below, or
				{/if}
				keep the tab open and we'll pull up the Ministry of Health's live stream as soon as it appears.
			</p>
			<div class="spacer" />
		</div>
	</main>
	<div class="updates {liveVideoId ? 'live' : ''}">
		<h3 class:invisible={!liveVideoId && loaded}>Live update:</h3>
		<YouTube
			size="lg"
			placeholder={!loaded || !liveVideoId}
			videoId={liveVideoId}
			enabled={liveVideoId || !loaded}
			autoplay={true}
		/>
		<div class="more">
			{#if anyPastVideos}
				<h3>Previous updates</h3>
				<div class="gallery">
					{#each pastVideoIDs as data}
						{#if data !== null}
						<div class="youtubeVideoWrapper">
							<h4>{dayjs(data[0]).format("dddd['s update], DD MMMM")}</h4>
							<YouTube videoId={data[1]} />
						</div>
							
						{/if}
					{/each}
				</div>
			{/if}
		</div>
		<footer>
			<p>
				This project was developed by
				<a href="https://jmw.nz/" target="_blank" rel="noreferrer">Jasper</a>
				&amp;
				<a href="https://zac.nz/" target="_blank" rel="noreferrer">Zac</a>
				through
				<a href="https://qrl.nz/" target="_blank" rel="noreferrer">Questionable Research Labs </a>
			</p>
		</footer>
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
			margin-left: calc(var(--social-distance));
			margin-right: calc(var(--social-distance));
			.countdown {
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;
				text-align: center;
			}

			h2.live {
				font-weight: 400;
				font-size: 4rem;
			}
			@media (max-width: 750px) {
				h2.live {
					font-size: 2rem;
				}
			}
			@media (min-width: 750px) and (max-width: 1000px) {
				h2.live {
					font-size: 3rem;
				}
			}

			p {
				max-width: 40em;
				// margin: 0 1em;
			}

			.spacer {
				padding-top: 25vh;
			}
		}

		.updates {
			position: relative;
			z-index: 10;
			margin-top: -45vh;
			text-align: center;
			background: linear-gradient(to bottom, #fff0 0, white 40vh);
			padding-bottom: 1rem;

			h3 {
				font-size: 2em;
			}

			.invisible {
				visibility: hidden;
				transition: 0.25s ease height;
			}

			.more.live {
				padding-top: var(--social-distance);
			}
			.more {
				.gallery {
					display: flex;
					flex-wrap: wrap;
					row-gap: 2rem;
					column-gap: 2rem;
					max-width: 62rem;
					margin: auto;
					h4 {
						font-size: 1.25rem;
					}
				}
			}

			footer {
				margin-top: 5rem;
				margin-left: 1em;
				margin-right: 1em;
				a {
					color: #000d;
					text-decoration: none;
					font-weight: bold;
				}
			}
		}
	}
	.content.loading {
		--social-distance: 4em;
	}

	.loading-countdown-container {
		min-height: 24rem;
		display: flex;
		align-items: center;
	}
	@media (max-width: 62rem) {
		.gallery {
			justify-content: center;
			flex-direction: row;
		}
	}
	@media (max-width: 750px) {
		.content {
			--social-distance: 2em;

			h2,
			p {
				margin: 0;
			}
			main .spacer {
				padding-top: 40vh;
			}
		}
		.content.loading {
			--social-distance: 1em;
		}

		.loading-countdown-container {
			min-height: 10rem;
			display: flex;
			align-items: center;
		}

		.gallery {
			justify-content: center;
		}

		.invisible {
			display: none;
		}
	}
</style>
