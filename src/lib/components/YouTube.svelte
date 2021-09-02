<script lang="ts">
	import { BarLoader } from 'svelte-loading-spinners';

	export let videoId: String;
	export let size: 'lg' | 'md' | 'sm' = 'md';
	export let placeholder: Boolean = false;
	export let enabled: Boolean = true;
</script>

<div class="container {size} {enabled ? '' : 'hidden'}">
	{#if placeholder}
		<span class="video placeholder">
			<BarLoader size="5" color="white" unit="em" duration="1s" />
		</span>
	{:else}
		<iframe
			title="COVID Update Live Stream"
			id="ytplayer"
			class="video"
			type="text/html"
			src="https://www.youtube.com/embed/{videoId}"
			frameborder="0"
			allowfullscreen
		/>
	{/if}
</div>

<style lang="scss">
	.md {
		--width-size: 30rem;
	}
	.lg {
		--width-size: 45rem;
	}

	.video {
		width: var(--width-size);
		height: calc(var(--width-size) * 0.5625);
		border-radius: 0.4em;
		box-shadow: 0 0.3em 8px -8px rgb(35 35 35 / 65%), 0 7px 20px -8px #ffcc009e;
		background: radial-gradient(farthest-corner at 30% 0, #272727, black);
		transition: opacity 0.5s ease, height 0.6s ease;
	}
	.video.placeholder {
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}

	.hidden .video {
		opacity: 0;
		height: 0;
	}
</style>
