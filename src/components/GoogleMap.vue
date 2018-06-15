<template>
  <div>
    <div>
      <h2>Search and add a pin</h2>
      <label>
        <gmap-autocomplete
          @place_changed="setPlace">
        </gmap-autocomplete>
        <button @click="addMarker">Add</button>
      </label>
      <br/>

    </div>
    <br>
    <gmap-map ref="mapRef" 
      :center="center"
      :zoom="18"
      style="width:100%;  height: 600px;"
    >
    <gmap-info-window :options="infoOptions" :position="infoWindowPos" :opened="infoWinOpen" @closeclick="infoWinOpen=false">
        {{infoContent}}
      </gmap-info-window>
      <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :icon = m.icon
        @click="toggleInfoWindow(m,index)"
      ></gmap-marker>
    </gmap-map>
  </div>
</template>

<script>
var map;
export default {
  name: "GoogleMap",
  data() {
    return {
      /**default to Montreal to keep it simple */
      center: { lat: 45.508, lng: -73.587 },
      markers: [],
      places: [],
      icon: [],
      currentPlace: null,
      infoContent: "",
      currentMidx: -1,
      infoWindowPos: {
        lat: 0,
        lng: 0
      },
      infoWinOpen: false,
      infoOptions: {
        maxWidth: 250,
        pixelOffset: {
          width: 10,
          height: 15
        }
      }
    };
  },
  mounted() {
    this.geolocate();
  },

  methods: {
    /** receives a place object via the autocomplete component */
    setPlace(place) {
      this.currentPlace = place;
    },

    /**adding marker to places */
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
        this.markers.push({ position: marker });
        this.places.push(this.currentPlace);
        this.center = marker;
        this.currentPlace = null;
      }
    },

    toggleInfoWindow(marker, index) {
      //if (index != 0) {
        this.infoWindowPos = marker.position;
        this.infoContent = marker.placeName + "\n" + marker.vicinity;
        //check if its the same marker that was selected if yes toggle
        if (this.currentMidx == index) {
          this.infoWinOpen = !this.infoWinOpen;
        } else {
          //if different marker set infowindow to open and reset current marker index
          this.infoWinOpen = true;
          this.currentMidx = index;
        }
      //}
    },

    /**getting current location */
    geolocate: function() {
      navigator.geolocation.getCurrentPosition(position => {
        this.center = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
        

         this.markers.push(new google.maps.Marker({
            position: this.center,
            map: this.$refs.mapRef.$mapObject,
            placeName: 'Current Location',
            vicinity: ''
          }));
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "hospital"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "atm"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "bank"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "pharmacy"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "train_station"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "bus_station"
        );
        this.addNearByHospitals(
          position.coords.latitude,
          position.coords.longitude,
          "airport"
        );
      });
    },

    addNearByHospitals: function(lat, lang, placeName) {
      var presentLocation = new google.maps.LatLng(lat, lang);
      var request = {
        location: presentLocation,
        radius: 1000,
        types: [placeName] // this is where you set the map to get the hospitals and health related places
      };
      var service = new google.maps.places.PlacesService(
        this.$refs.mapRef.$mapObject
      );
      service.nearbySearch(request, this.callback);
    },
    callback: function(results, status) {
      if (status == google.maps.places.PlacesServiceStatus.OK) {
        for (var i = 0; i < results.length; i++) {
          this.createMarker(results[i]);
        }
      }
    },
    createMarker: function(place) {
      var icon = {
        url: place.icon, // url
        scaledSize: new google.maps.Size(20, 20), // scaled size
        origin: new google.maps.Point(0, 0), // origin
        anchor: new google.maps.Point(0, 0) // anchor
      };
      var newMarker = new google.maps.Marker({
        position: place.geometry.location,
        map: this.$refs.mapRef.$mapObject,
        icon: icon,
        placeName: place.name,
        vicinity: place.vicinity
      });
      this.markers.push(newMarker);
    }
  }
};
</script>