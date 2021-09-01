<script lang="ts">
    import {default as dayjs} from "dayjs"
    
    import CountDown from "$lib/components/CountDown.svelte";
    import TimeDisplay from "$lib/components/TimeDisplay.svelte";

    import {default as  relativeTime} from "dayjs/plugin/relativeTime";
    dayjs.extend(relativeTime)

    let timeDisplayed = dayjs("2021-09-01 13:00");

</script>
<main>
    <div>
        <TimeDisplay bind:timeDisplayed/>
        <CountDown bind:timeDisplayed/>
    </div>
</main>


<style lang="scss">
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
        overflow: hidden;
        margin: 0;
        padding: 0;
        background: repeating-linear-gradient(
            45deg,
            #ffcc00,
            #ffcc00 50px,
            #ffff 50px,
            #ffff 100px,
            );
        div {
            background: white;
            width: 80%;
            height: 60vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
    }
</style>