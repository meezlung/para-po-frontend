<script>
    import 'leaflet/dist/leaflet.css';
    import Map from '../components/Map.svelte'
	import { onMount } from 'svelte';


    // let jeepMarkers = [
    //     { lat: 14.648792269553422, lng: 121.06847648049134, popup: 'Jeepney #1' }, // DCS
    //     { lat: 14.655, lng: 121.07, popup: 'Jeepney #2' }
    // ];

    let jeepMarkers = [];

    onMount(() => {
        // Fetch markers initiallly
        fetchMarkers();
        
        // Poll every 5 secs
        const interval = setInterval(fetchMarkers, 5000);

        return () => clearInterval(interval);
    });

    async function fetchMarkers() {
        const res = await fetch('http://127.0.0.1:8000/api/jeepneys'); // Server endpoint
        const data = await res.json();
        console.log('Fetched markers:', data);
        jeepMarkers = data;
    }
</script>

<h1>Map</h1>

<Map center={[14.655, 121.066]} zoom={15.5} markers={jeepMarkers} />