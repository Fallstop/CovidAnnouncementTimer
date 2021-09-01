<script lang="ts">
    import type {Dayjs} from "dayjs";

    export let timeDisplayed: Dayjs;
</script>
<h2>{timeDisplayed.fromNow()}</h2>
