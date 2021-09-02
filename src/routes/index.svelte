<script lang="ts">
    import {default as dayjs} from "dayjs"
    import {default as  relativeTime} from "dayjs/plugin/relativeTime.js";
    import { BarLoader } from 'svelte-loading-spinners'

    
    import CountDown from "$lib/components/CountDown.svelte";
    import TimeDisplay from "$lib/components/TimeDisplay.svelte";

    import { onMount } from "svelte";
    dayjs.extend(relativeTime)

    let timeDisplayed;
    let loaded = false;

    onMount(()=>{
        (async () => {
		try {
			let response = await fetch('https://covid-announcement-backend.host.qrl.nz/api/get-announcement-time');
			let body = await response.json();
			console.log('Date pushed', body);
            if (body['date_of_announcement']!==null) {
                timeDisplayed = dayjs(body['date_of_announcement']);
                if(timeDisplayed.isBefore(dayjs().subtract(3,'h'))) {
                    // Give three hour leeway, then start counting down to the next one
                    timeDisplayed = timeDisplayed.add(1,'d').hour(13)
                }
                loaded = true

            }
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
            {#if typeof timeDisplayed !== "undefined"}
                <TimeDisplay bind:timeDisplayed/>
                <CountDown bind:timeDisplayed/>
            {:else}
                <h3>Hasn't been announced yet</h3>
            {/if}
        {:else}
            <BarLoader size="5" color="currentColor" unit="em" duration="1s"></BarLoader>
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
            text-align: center;
        }
    }
    @media (max-width: 750px) {
        main div {
            width: 100%;
            padding: 1em;
        } 
    }
</style>