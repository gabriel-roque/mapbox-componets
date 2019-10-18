<template>
    <div id="map"></div>
</template>

<script>
    import mapboxgl from 'mapbox-gl'
    export default {
        name: "mapbox-full",
        data() {
            return {
                token: 'pk.eyJ1IjoiZ2FicmllbHJtcyIsImEiOiJjazFzbDBubjAwaDZoM2JwNGRxcmxjY25yIn0.dhif7znIWxyQLfbAGkfFIw',
                map: {},
                geocoder: {},
                localizacao: {
                    longitude: 0,
                    latitude: 0
                }
            }
        },

        methods: {
            startMap() {
                mapboxgl.accessToken = this.token;

                this.map = new mapboxgl.Map({
                    container: document.querySelector('#map'),
                    style: 'mapbox://styles/mapbox/streets-v11',
                    center: [-47.8823, -15.7934], // Brasília-DF
                    zoom: 12
                });

                this.configureGeolocation();
                this.configureControlsMap();
            },

            configureGeolocation(){
                this.geocoder = new MapboxGeocoder({
                    accessToken: mapboxgl.accessToken,
                    mapboxgl: mapboxgl,
                    placeholder: 'Pesquise Ex: Brasil ...'
                });
            },

            configureControlsMap(){
                this.map.addControl(this.geocoder, 'top-left');
                this.map.addControl(new mapboxgl.NavigationControl());
                this.map.addControl(new mapboxgl.GeolocateControl({
                    positionOptions: {
                        enableHighAccuracy: true
                    },
                    trackUserLocation: true
                }))
            },

            resultSearch(){
                this.map.on('load', () => {
                    this.geocoder.on('result', (ev) => {
                        // console.log(ev.result.center[0]);
                        // console.log(ev.result.center[1]);

                        let marker = {
                            longitude: ev.result.center[0],
                            latitude: ev.result.center[1]
                        };
                        this.clearMakers();
                        this.addMakerMap(marker);



                        this.localizacao.longitude = ev.result.center[0];
                        this.localizacao.latitude = ev.result.center[1];
                    });
                });
            },

            markerLocationWithClick(){
                this.map.on('click', (e) => {
                    let marker = {
                        longitude: (e.lngLat.lng).toFixed(3),
                        latitude: (e.lngLat.lat).toFixed(3)
                    };

                    Object.assign(this.localizacao, marker);

                    this.clearMakers();
                    this.addMakerMap(marker);
                });
            },

            clearMakers() {

                let currentMaker = document.querySelectorAll('.mapboxgl-marker');
                console.log(currentMaker)
                console.log(currentMaker.length)

                if (currentMaker.length > 0) {
                    currentMaker.forEach((marker, index) => {
                        currentMaker[index].remove()
                    });
                }
            },

            addMakerMap({longitude, latitude}){
                let newMarker = new mapboxgl.Marker()
                        .setLngLat([longitude, latitude])
                        .addTo(this.map);
                // console.log(longitude, latitude);
            }
        },


        mounted() {
            this.startMap();
            this.resultSearch();
            this.markerLocationWithClick();
        }
    }
</script>

<style scoped>

    #map {
        top: 0;
        bottom: 0;
        width: 100%;
        height: 500px;
    }

    .map-overlay {
        position: absolute;
        width: 150px;
        top: 0;
        left: 0;
        padding: 10px;
        margin-top: 55px;
    }

    .map-overlay .map-control-cep {
        background-color: #fff;
        box-shadow: 0 2px 2px rgba(0, 0, 0, 0.10);
        border-radius: 5px;
    }

    .map-input-cep {
        width: 75%;
        border: 0px;
        background-color: transparent;
        margin: 0;
        height: 25px;
        color: rgba(0, 0, 0, 0.75);
        padding: 6px 20px;
        outline: none !important;
    }

    .map-control-local {
        background: white;
        border-radius: 4px;
        margin-top: 15px;
        box-shadow: 0 2px 2px rgba(0, 0, 0, 0.10);
        width: 100px !important;
        display: inline-block !important;
    }

    .map-control-local > button {
        width: 100px;
        height: 30px;
        display: block;
        padding: 0;
        outline: none;
        border: 0;
        color: blue;
        box-sizing: border-box;
        background-color: transparent;
        cursor: pointer;
    }

    .mapboxgl-popup {
        max-width: 200px;
    }
    .mapboxgl-popup-content {
        text-align: center;
        font-family: 'Open Sans', sans-serif;
    }

</style>