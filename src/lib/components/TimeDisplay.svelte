<script lang="ts">
    import type {Dayjs} from "dayjs";

    export let timeDisplayed: Dayjs;
</script>

<span class="mono">{timeDisplayed.format("h:mm:ss A")}</span>

<style lang="scss">
    span {
        font-size: 10rem;
    }
    @media (max-width: 750px) {
        span {
            font-size: 3rem;
        }
    }
    @media (min-width: 750px) and (max-width: 1000px) {
        span {
            font-size: 5rem;
        }
    }
</style>