<script lang="ts">
	import type { Dayjs } from 'dayjs';

	export let timeDisplayed: Dayjs;
</script>

<h1>
	{timeDisplayed.format('h:mm A')}
</h1>

<style lang="scss">
	h1 {
		font-weight: 600;
		font-size: 10rem;
		margin: 0;
		line-height: 1;
	}
	@media (max-width: 750px) {
		h1 {
			font-size: 4rem;
		}
	}
	@media (min-width: 750px) and (max-width: 1000px) {
		h1 {
			font-size: 5rem;
		}
	}
</style>
