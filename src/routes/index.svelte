<script lang="ts">
    import {default as dayjs} from "dayjs"
    
    import CountDown from "$lib/components/CountDown.svelte";
    import TimeDisplay from "$lib/components/TimeDisplay.svelte";

    import {default as  relativeTime} from "dayjs/plugin/relativeTime.js";
import { onMount } from "svelte";
    dayjs.extend(relativeTime)

    let timeDisplayed = dayjs("TODO");
    let loaded = false;

    onMount(()=>{
        (async () => {
		try {
			let response = await fetch('https://covid-announcement-backend.host.qrl.nz/api/get-announcement-time');
			let body = await response.json();
			console.log('Date pushed', body);
			timeDisplayed = dayjs(body['date_of_announcement']);
            loaded = true
            
		} catch (e) {
			console.log('It shit itself', e);
		}
        })();
    })

</script>
<main>
    <div>
        {#if loaded}
        <TimeDisplay bind:timeDisplayed/>
        <CountDown bind:timeDisplayed/>
        {:else}
            hmm yes loading here
        {/if}
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