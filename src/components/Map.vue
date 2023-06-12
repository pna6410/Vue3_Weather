<template>
  <div id="mapContainer">
    <div id="map"></div>
  </div>
</template>

<script>
import MapPositions from "~/assets/map-positions.json";

export default {
  mounted() {
    const API_KEY = "a9d013edd6c6d6fd54e7f5375e647243";

    if (window.kakao && window.kakao.maps) {
      this.initMap();
    } else {
      const script = document.createElement('script');
      /* global kakao */
      script.onload = () => kakao.maps.load(this.initMap);
      script.src = `http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${API_KEY}`;
      document.head.appendChild(script);
    }
  },
  methods: {
    initMap() {
      var mapContainer = document.getElementById('map'), // 지도를 표시할 div
        mapOption = {
          center: new kakao.maps.LatLng(37.73035, 127.967487), // 지도의 중심좌표
          level: 13, // 지도의 확대 레벨
      };
      const map = new kakao.maps.Map(mapContainer, mapOption);
      const positions = MapPositions.map((pos) => ({
        latlng: new kakao.maps.LatLng(...pos.latlng),
        cityName: pos.cityName,
      }))
      // 마커를 생성합니다.
      positions.forEach((pos) => {
        var marker = new kakao.maps.Marker({
          position: pos.latlng, // 마커의 위치
        });
        // 마커가 지도 위에 표시 되도록 설정합니다.
        marker.setMap(map);
        // 마커를 클릭했을 때, 클릭된 마커의 좌표를 Vuex Store에 전달하는 작업
        // 아래 코드는 카카오 디벨로퍼 공식문서에 그대로 나와있는 것을 활용
        kakao.maps.event.addListener(marker, "click", () => {
          // 클릭한 위도, 경도 정보를 불러온다.
          // watch로 따로 빼지 않고, 직접 Vuex Store에 할당
          this.$store.commit('openWeatherApi/SET_CITYNAME', pos.cityName);
          this.$store.commit('openWeatherApi/SET_LATLON', marker.getPosition());
          this.$store.dispatch('openWeatherApi/FETCH_OPENWEATHER_API');
        });
        // 아래 코드는 지도 위의 마커를 제거하는 코드입니다.
        // marker.setMap(null);
      });
    },
  },
}
</script>

<style lang="scss" scoped>
@import '~/scss/main.scss';

#mapContainer {
  @include center;
  width: 100%;
  height: 35%;

  #map {
    width: 80%;
    height: 90%;
    border-radius: 10px;
  }
}
</style>