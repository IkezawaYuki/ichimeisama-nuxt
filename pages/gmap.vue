<template>
  
  <v-layout wrap>
   <GmapMap class="map-panel" map-type-id="roadmap" 
        style="width: 100%; height: 100%"
        :draggable="true"
        :center="maplocation"
        :zoom="15"
        ref="mapRef">
        <GmapMarker v-for="m in makers"
        :position="m.position"
        :title="m.title"
        :clickable="true"
        :draggable="false"
        :icon="m.icon"
        :key="m.id">
      </GmapMarker>
    </GmapMap>
  </v-layout>
</template>

<script>
export default {
  data() {
    return {
      makers:[],
      maplocation:{lng: 0, lat: 0}
    }
  },
  mounted(){
    this.$nextTick(() => {
      this.$nuxt.$loading.start()
      setTimeout(() => this.$nuxt.$loading.finish(), 500)
    })
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        function(position){
          let coords = position.coords;
          // 緯度経度を取得
          this.maplocation.lat = coords.latitude;
          this.maplocation.lng = coords.longitude;
          // 地図読み込み完了時のイベント
          // this.$gmapApiPromiseLazy().then(() => {
          //   google.maps.event.addListenerOnce(this.$refs.mapRef.$mapObject, 'idle',
          //     function() { this.setPlaceMakers() }.bind(this)
          //   );
          // });
        }.bind(this),
        function(error) {
          // エラーの場合は東京駅周辺に移動
          this.maplocation.lat = 35.6813092;
          this.maplocation.lng = 139.7677269;
        }
      );
    } else {
      // 現在地取得不可の場合は東京駅周辺に移動
      this.maplocation.lat = 35.6813092;
      this.maplocation.lng = 139.7677269;
    }
  },
  methods:{
    setPlaceMakers() {
      let map = this.$refs.mapRef.$mapObject
      let placeService = new google.maps.places.PlacesService(map);
      // Places APIのnearbySearchを使用する。
      placeService.nearbySearch(
        {
          location: new google.maps.LatLng(this.maplocation.lat, this.maplocation.lng),
          radius: 500,
          type: ['restaurant']
        },
        function(results, status) {
          if (status == google.maps.places.PlacesServiceStatus.OK) {
            results.forEach(place => {
              // デフォルトのアイコンが大きめなので縮小
              let icon = {
                url: place.icon, // url
                scaledSize: new google.maps.Size(30, 30), // scaled size
                origin: new google.maps.Point(0, 0), // origin
                anchor: new google.maps.Point(0, 0) // anchor
              };
              let maker = {
                position: place.geometry.location,
                icon: icon,
                title: place.name,
                id: place.place_id
              };
              this.makers.push(maker);
            });
          }
        }.bind(this)
      );
    }
  }
};

</script>
