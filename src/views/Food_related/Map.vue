<template>
    <div>
        <div id="dashboard_main">
            <div id="google_map_container">
                <gmap-map
                        :center="center"
                        :zoom="zoom"
                        style="width:100%;  height: 85vh;"
                        :options="mapOptions"
                        ref="myMap"
                >
                    <!--<gmap-marker-->
                    <!--:key="index"-->
                    <!--v-for="(m, index) in markers"-->
                    <!--:position="m.position"-->
                    <!--:clickable="true"-->
                    <!--@click="toggleInfo(m, index)"-->
                    <!--&gt;</gmap-marker>-->
                    <gmap-info-window :options="infoOptions" :position="infoPosition" :opened="infoOpened"
                                      @closeclick="infoOpened=false">
                        <p class="infoWindow_title">{{ location_selected }}</p>
                        <pre>{{ operating_hours }}</pre>
                        Address: <span>{{ address }}</span>
                        <br>
                        <span v-if="rating">Rating: {{ rating }}
                        <br></span>
                        <span v-if="number">Number: {{ number }}</span>

                    </gmap-info-window>

                    <!--<gmap-marker v-for="(item, key) in coordinates" :key="key" :position="getPosition(item)"-->
                    <!--:clickable="true"-->
                    <!--@click="toggleInfo(item, key)"/>-->


                    <!--</gmap-custom-marker>-->
                    <gmap-custom-marker v-for="(item, key) in canteen_info" :key="key"
                                        :marker="getCanteenPosition(item)"
                                        :clickable="true"
                                        @click.native="toggleInfo(item, key)"
                    >
                        <img :src="markercolour.yellow" v-if="Math.trunc(item.rating) === 4"/>
                        <img :src="markercolour.red" v-else-if="Math.trunc(item.rating) === 3"/>
                        <img :src="markercolour.blue" v-else-if="Math.trunc(item.rating) === 2"/>
                        <img :src="markercolour.grey" v-else/>
                    </gmap-custom-marker>

                </gmap-map>
            </div>
        </div>
    </div>
</template>

<script>
    import firebase from 'firebase';
    import Vue from 'vue';
    import GmapCustomMarker from 'vue2-gmap-custom-marker';
    import processFireBase from '@/mixins/processFireBase';
    import locationMethods from '@/mixins/locationMethods';
    import red_marker from "@/assets/markers/red.png";
    import grey_marker from "@/assets/markers/grey.png";
    import blue_marker from "@/assets/markers/blue.png";
    import yellow_marker from "@/assets/markers/yellow.png";


    export default {
        components: {
            'gmap-custom-marker': GmapCustomMarker
        },
        mixins: [processFireBase, locationMethods],
        data() {
            return {
                // All the data need for GoogleMaps
                center: {lat: 1.3483, lng: 103.6831}, // init centre of singapore
                streetViewControl: false,
                zoom: 15, // init zoom
                markers: [],
                map: null,
                infoOptions: {
                    pixelOffset: {
                        width: 0,
                        height: -35
                    }
                },
                infoPosition: null,
                infoOpened: false,
                infoCurrentKey: null,
                location_selected: null,
                operating_hours: null,
                address: null,
                rating: null,
                number: null,
                mapOptions: {
                    disableDefaultUI: true,
                    clickableIcons: false
                },
                markercolour: {
                    yellow: yellow_marker,
                    red: red_marker,
                    blue: blue_marker,
                    grey: grey_marker
                },
                canteen_info: {}
            }
        },
        mounted() {
            this.loadCanteensFromFirebase();
            // this.geolocate();
            // this.getCurrentPos();
        },
        methods: {
            // getPosition: function (marker) {
            //     return {
            //         lat: parseFloat(marker.lat),
            //         lng: parseFloat(marker.lng)
            //     }
            // },
            // addMarker(lat, lng, place_name) {
            //     const marker = {
            //         lat: lat,
            //         lng: lng
            //     };
            //     this.markers.push({
            //         position: marker,
            //         infoText: place_name
            //     });
            //     this.center = marker;
            // },
            // geolocate: function () {
            //     navigator.geolocation.getCurrentPosition(position => {
            //         this.center = {
            //             lat: position.coords.latitude,
            //             lng: position.coords.longitude
            //         };
            //     });
            // },
            toggleInfo: function (marker, idx) {
                // console.log(marker);
                this.infoPosition = this.getCanteenPosition(marker);
                this.location_selected = marker.canteenName;
                this.operating_hours = marker.hours;
                this.address = marker.info.address;
                this.rating = marker.rating;
                this.number = marker.info.international_phone_number;
                if (this.infoCurrentKey === idx) {
                    this.infoOpened = !this.infoOpened;
                } else {
                    this.infoOpened = true;
                    this.infoCurrentKey = idx;
                }
            },
            loadCanteensFromFirebase: function () {
                let self = this;
                this.getDocs("can_info").then(
                    function (res) {
                        for (let doc of res.docs) {
                            // console.log(doc.id);
                            let data = doc.data();
                            data.canteenName = doc.id;
                            data.rating = data.info.rating;
                            data.hours = data.hours.replace(/\\n/g, '\n');
                            Vue.set(self.canteen_info, doc.id, data);
                        }

                        // console.log(self.canteen_info);
                    }
                )
            },
            getCanteenPosition: function (marker) {
                return {
                    lat: parseFloat(marker.lat),
                    lng: parseFloat(marker.long)
                }
            },
        },
        updated() {
        },
        computed: {}
    }

</script>

<style scoped>

    .input-field > select {
        font-family: 'Calibri';
        font-weight: bold;
    }

    .infoWindow_title {
        font-family: 'Helvetica Rounded';
        font-weight: bold;
        font-size: 1.2rem;
        margin: 3%;
    }


</style>
