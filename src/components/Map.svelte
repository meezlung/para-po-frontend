<script>
    import { onMount } from "svelte";
    import L from 'leaflet';
    import 'leaflet/dist/leaflet.css';

    const jeepIcon = L.icon({
        iconUrl: '/jeepney.png',
        iconSize: [40, 40],
        iconAnchor: [20, 40],
        popupAnchor: [0, -40],   
    })

    export let center = [14.652, 121.065]; // Near UP Diliman
    export let zoom = 14;
    export let markers = []; // [{lat: 0, lng: 0, popup: 'Jeep 1'}, ...]

    let map;
    let mapElement; // Reference to div container for the map
    let markerLayer; // We'll store a LayerGroup for easy updates

    onMount(() => {
        // Initialize the leaflet map
        map = L.map(mapElement, {
            center,
            zoom
        });

        // Add a tile layer (OpenStreetMap)
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution:
                '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
        }).addTo(map);

        // Automatically plot markers
        markerLayer = L.layerGroup().addTo(map);

        // // Manually plot markers
        // markers.forEach(({ lat, lng, popup }) => {
        //     const marker = L.marker([lat, lng]).addTo(map);
        //     if (popup) {
        //         marker.bindPopup(popup);
        //     }
        // });
    });

    // A Svelte reactive statement that re-runs whenever markers changes
    $: if (markerLayer) {
        // Clear existing markers
        markerLayer.clearLayers();

        // Add new markers from the updated array
        markers.forEach(({ lat, lng, popup }) => {
            const marker = L.marker([lat, lng], { icon: jeepIcon });
            if (popup) {
                marker.bindPopup(popup);
            }
            marker.addTo(markerLayer);
        })
    };
</script>

<style>
    /* Make the map container take some space */
    .map-container {
        width: 100%;
        height: 90vh;
    }
</style>
  

<div bind:this={mapElement} class="map-container"></div>