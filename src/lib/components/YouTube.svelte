<script lang="ts">
	export let videoId: String;
	export let size: 'lg' | 'md' | 'sm' = 'md';
</script>

<div class={size}>
	<iframe
		title="COVID Update Live Stream"
		id="ytplayer"
		type="text/html"
		src="https://www.youtube.com/embed/{videoId}"
		frameborder="0"
		allowfullscreen
	/>
</div>

<style lang="scss">
	.md {
		--width-size: 30rem;
	}
	.lg {
		--width-size: 45rem;
	}

	iframe {
		width: var(--width-size);
		height: calc(var(--width-size) * 0.5625);
		border-radius: 0.4em;
		box-shadow: 0 0.3em 8px -8px rgb(35 35 35 / 65%), 0 7px 20px -8px #ffcc009e;
	}
</style>
