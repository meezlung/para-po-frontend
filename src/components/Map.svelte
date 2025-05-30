<script>
    import { onMount } from "svelte";
    import L from 'leaflet';
    import 'leaflet/dist/leaflet.css';

    const jeepIcon = L.icon({
        iconUrl: '/jeepney.png',
        iconSize: [25, 25],
        iconAnchor: [10, 10],
        popupAnchor: [0, 0],   
    })
    
    // a new “you are here” icon
    const meIcon = L.icon({
        iconUrl: '/me.png',
        iconSize: [16, 16],
        iconAnchor: [8, 8],
        popupAnchor: [0, 0],
    });

    export let center = [14.652, 121.065]; // Near UP Diliman
    export let zoom = 14;
    export let markers = []; // [{lat: 0, lng: 0, popup: 'Jeep 1'}, ...]
    export let myPosition = null; // { lat, lng } or null

    let map;
    let mapElement; // Reference to div container for the map
    let markerLayer; // We'll store a LayerGroup for easy updates
    let meMarker;

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

        // L.tileLayer.provider('CartoDB.Positron').addTo(map);

        // L.tileLayer.provider('Esri.WorldImagery').addTo(map);

        // L.tileLayer.provider('OpenTopoMap').addTo(map);

        // L.tileLayer.provider('Stadia.StamenWatercolor').addTo(map);

        // L.tileLayer('https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png', {
        //     maxZoom: 20,
        //     attribution: '<a href="https://github.com/cyclosm/cyclosm-cartocss-style/releases" title="CyclOSM - Open Bicycle render">CyclOSM</a> | Map data: &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        // }).addTo(map);

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
            
            // label ng jeep
            if (popup) {
                marker.bindPopup(popup);
            }
            
            // add label
            marker.bindTooltip(popup, {
                permanent: true,
                direction: 'top',
                offset: [0, -10],
                className: 'jeep-label'
            });

            // add the marker to the map
            marker.addTo(markerLayer);
        })
    };

    $: if (map && myPosition) {
        const { lat, lng } = myPosition;
        
        if (meMarker) {
            // just move it
            meMarker.setLatLng([lat,lng]);
        } else {
            // create it once
            meMarker = L.marker([lat, lng], { icon: meIcon })

            // add popout sticking out
            meMarker.bindTooltip("You are here!", {
                permanent: true,
                direction: 'top',
                offset: [0, -10],
            });
            
            meMarker.addTo(map);
        }
    }
</script>

<style>
    /* Make the map container take some space */
    .map-container {
        width: 100%;
        height: 90vh;
    }
</style>

<div bind:this={mapElement} class="map-container"></div>