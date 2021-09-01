<script lang="ts">
    import {default as dayjs} from "dayjs"
    
    import CountDown from "$lib/components/CountDown.svelte";
    import TimeDisplay from "$lib/components/TimeDisplay.svelte";

    import {default as  relativeTime} from "dayjs/plugin/relativeTime";
    dayjs.extend(relativeTime)

    let timeDisplayed = dayjs("2021-09-01 13:00");

</script>
<main>
    <TimeDisplay bind:timeDisplayed/>
    <CountDown bind:timeDisplayed/>

</main>


<style lang="scss">
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
    }
</style>