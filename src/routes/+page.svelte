<script>

    import axios from "axios";
    import {Box, P, SectionVariants, Select, Typography} from "dnit-sx";
    import {onMount} from "svelte";
    import {Instant} from "js-joda";

    let selectedTime = "8:00";

    let bpms;
    let data;
    onMount(() => {
        setInterval(() => getBpm(), 1000)
    })

    async function getBpm() {
        const res = await axios.get("http://localhost:8080/home/")
        data = res.data
        bpms = data.rates
        console.log(data.wake)
    }

    async function setAlarm() {
        const res = await axios.post("http://localhost:8080/home/set-time", {
            time: parseInstant(selectedTime)
        })
        console.log(res.data)
    }

    function parseInstant(timeString) {
        const [hours, minutes] = timeString.split(':');
        const currentDate = new Date();
        const tomorrow = new Date(currentDate.getFullYear(), currentDate.getMonth(), currentDate.getDate() + 1, hours, minutes)
        return new Instant.parse(tomorrow.toISOString());
    }

    const times = Array.from({length: 20}, (_, index) => {
        const hours = Math.floor((index + 10) / 2);
        const minutes = (index % 2) * 30;
        return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
    });
    function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    function playMusic() {
        sleep(10000).then(()=>responsiveVoice.speak("Hey Google Speel muziek op spotify", "Dutch Female"))
    }
</script>

{#if bpms}
    <Typography variant="1" sx={{fontsize:4, fontWeight:"bold", color:"#944a43"}}>Heart rate reader</Typography>
    <Select selected={selectedTime} items={times} on:change={(e)=>{selectedTime=e.detail.value; setAlarm()}}/>
    <Box variant={SectionVariants.MAIN} sx={{flex:"col", flexAlign:"center", bgc:"#eee"}}>
        {#each bpms as bpm, i}
            <P id={`bpm${i}`}>{bpm}</P>
        {/each}
    </Box>

{/if}

{#if data}
    {#if data.wake}
        {responsiveVoice.speak("Hey Google Goedemorge", "Dutch Female",{onend:()=>playMusic()})}
    {/if}
{/if}
