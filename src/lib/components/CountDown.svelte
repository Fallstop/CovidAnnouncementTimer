<script lang="ts">
	import type { Dayjs } from 'dayjs';

	export let timeDisplayed: Dayjs;
	export let predicted = false;
</script>

<h2>
	({timeDisplayed.fromNow()}{#if predicted}, probably{/if})
</h2>

<style lang="scss">
	h2 {
		font-weight: 400;

		font-size: 4rem;
	}
	@media (max-width: 750px) {
		h2 {
			font-size: 2rem;
		}
	}
	@media (min-width: 750px) and (max-width: 1000px) {
		h2 {
			font-size: 3rem;
		}
	}
</style>
