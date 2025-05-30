<script>
    import 'leaflet/dist/leaflet.css';
    import "https://kit.fontawesome.com/0a5209b735.js" 
    import Map from '../components/Map.svelte'
    import BottomSheet from '../components/BottomSheet.svelte';
    import { onMount } from 'svelte';
	import { tweened } from 'svelte/motion';

    // let jeepMarkers = [
    //     { lat: 14.648792269553422, lng: 121.06847648049134, popup: 'Jeepney #1' }, // DCS
    //     { lat: 14.655, lng: 121.07, popup: 'Jeepney #2' }
    // ];

    let jeepMarkers = [];
    let nearbyJeepneys = []; // only those <50m near me
    let myPosition = { lat: 14.652642382508079, lng: 121.0675288961253 } // CAL

    // Haversine (in meters)
    function getDistance([lat1, lon1], [lat2, lon2]) {
        const R = 6371000; // Earth radius in m
        const toRad = x => x * Math.PI / 180;
        const dLat = toRad(lat2 - lat1);
        const dLon = toRad(lon2 - lon1);
        const a = Math.sin(dLat/2)**2
                + Math.cos(toRad(lat1)) * Math.cos(toRad(lat2))
                * Math.sin(dLon/2)**2;
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }

    // make nearbyJeepneys reactive depende sa current position mo 
    $: nearbyJeepneys = jeepMarkers.filter(m => {
        const dist = getDistance([m.lat, m.lng], [myPosition.lat, myPosition.lng]);
        return dist <= 500;
    });
    
    onMount(() => {
        // Fetch markers initiallly
        fetchMarkers();
        
        // Poll every 50 ms
        const interval = setInterval(fetchMarkers, 30);

        return () => clearInterval(interval);
    });

    async function fetchMarkers() {
        const res = await fetch('http://127.0.0.1:8000/api/jeepneys'); // Server endpoint
        const data = await res.json();
        console.log('Fetched markers:', data);
        jeepMarkers = data;
    }

    const sheetHeight = tweened(80, { duration: 200 });
</script>

<style>
    /* Product Sans font */
    @import url("https://fonts.googleapis.com/css2?family=Product+Sans&display=swap");

    :global(html), :global(body), :global(#app) {
        font-family: 'Product Sans', sans-serif;
        height: 100%;
        margin: 0;
    }

    .overlay .fa-solid.fa-gear.icon {
        position: absolute;
        top: 2%;
        left: 92%;
        transform: translateX(-50%);
        background: rgba(255,255,255,0.9);
        padding: 8px 8px;
        border-radius: 9999px;
        display: flex;
        align-items: center;
        box-shadow: 0 2px 4px rgba(0,0,0,0.15);
        z-index: 500;
    }

    .overlay .fa-solid.fa-envelope{
        position: absolute;
        top: 8%;
        left: 92%;
        transform: translateX(-50%);
        background: rgba(255,255,255,0.9);
        padding: 8px 8px;
        border-radius: 9999px;
        display: flex;
        align-items: center;
        box-shadow: 0 2px 4px rgba(0,0,0,0.15);
        z-index: 500;
    }

    .overlay .title {
        position: absolute;
        top: 16px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(255,255,255,0.9);
        padding: 8px 16px;
        border-radius: 9999px;
        display: flex;
        align-items: center;
        box-shadow: 0 2px 4px rgba(0,0,0,0.15);
        z-index: 1000;
        font-weight: bold;
    }

    .container {
        position: relative;
        height: 100vh;
        width: 100%;
    }
</style>

<div class="container">
    <Map center={[14.652750607787588, 121.067457597345416]} zoom={15.3} markers={jeepMarkers} myPosition={myPosition}/>

    <div class="overlay" aria-label="Settings and title">
        <i class="fa-solid fa-gear icon" aria-label="Settings"></i>
        <i class="fa-solid fa-envelope" aria-label="Message"></i>
        <span class="title">PARA PO</span>
    </div>

    <BottomSheet on:stateChange={(e) => console.log('expanded?', e.detail.expanded)}>
        <!-- UI sa Jeepney -->
        <h2>Jeepneys Nearby (500m)</h2>
        {#if nearbyJeepneys.length}
            <ul>
                {#each nearbyJeepneys as { popup }}
                    <li>{popup}</li>
                {/each}
            </ul>
        {:else}
            <p>No jeepneys within 500 m</p>            
        {/if}
    </BottomSheet>
</div>
