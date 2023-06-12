<!-- eslint-disable vue/singleline-html-element-content-newline -->
<!-- eslint-disable vue/max-attributes-per-line -->
<!-- eslint-disable vue/html-indent -->
<template>
  <div class="rightContainer">
    <div id="cityNameBox">
      <div class="cityName">
        <p>{{ cityName }}</p>
        <p>{{ currentTime }}</p>
      </div>
    </div>
    <div id="contentsBox">
      <div class="buttonBox">
        <div class="buttonBackground">
            <button class="forecast">Forecast</button>
            <button class="airquality">Air Quality</button>
        </div>
      </div>
      <div class="weatherBox">
        <div class="airCondition">
            <p>{{ feeling }}</p>
        </div>
        <div class="detail">
            <div class="title">
                <p>🌈상세한 날씨 데이터🌞</p>
            </div>
            <div class="data" v-for="(detailData, index) in subWeatherData" :key="index">
                <div class="dataName">
                    <p></p>
                    <p>{{ detailData.name }}</p>
                </div>
                <div class="dataValue">
                    <p>{{ detailData.value }}<span></span></p>
                </div>
            </div>
        </div>
      </div>
    </div>
    <Map />
    <nav>
      <i class="fas fa-home"></i>
      <i class="fas fa-search-location"></i>
      <i class="fas fa-chart-line"></i>
      <i class="fas fa-cog"></i>
    </nav>
  </div>
</template>

<script>
import Map from "~/components/Map.vue";
import { computed, onMounted, ref, watchEffect } from "vue";
import { useStore } from "vuex";
import dayjs from "dayjs";
import "dayjs/locale/ko";
dayjs.locale("ko"); // global로 한국어 locale 사용

export default {
    components: {
        Map,
    },
    // Vue3 CompositionAPI 사용
    setup() {
        // 화면에 보여질 데이터
        let currentTime = dayjs().format("YYYY. MM. DD. ddd"); // 현재 시간

        let cityName = ref(""); // 도시 이름
        let feeling = ref(""); // 체감 온도
        let subWeatherData = ref([]); // 상세 날씨 데이터

        // 타임스탬프로 변환
        const Unix_timestamp = (dt) => {
            let date = new Date(dt * 1000);
            // padStart() 메서드는 현재 문자열의 시작을 다른 문자열로 채워, 주어진 길이를 만족하는 새로운 문자열을 반환합니다.
            // 채워넣기는 대상 문자열의 시작(좌측)부터 적용됩니다.
            let hour = date.getHours().toString().padStart(2, "0");
            return hour + "시";
        };

        // OpenWeatherAPI 호출 함수
        const store = useStore();
        const fetchOpenWeatherApi = async () => {
            // API 호출을 위한 필수 데이터
            try {
                await store.dispatch("openWeatherApi/FETCH_OPENWEATHER_API");
                const { currentFeelsLike, currentSunrise, currentSunset, currentVisibility } = store.state.openWeatherApi.currentWeather;

                let isInitialCityName = store.state.openWeatherApi.cityName; // 초기 도시이름 데이터
                let isFeelLikeTemp = computed(() => {
                    return currentFeelsLike;
                }); // 초기 체감온도 데이터
                let isTimeOfSunrise = computed(() => {
                    return currentSunrise;
                }); // 일출시간 데이터
                let isTimeOfSunset = computed(() => {
                    return currentSunset;
                }); // 일몰시간 데이터
                let isLineOfSight = computed(() => {
                    return currentVisibility;
                }); // 가시거리 데이터

                const tempPoints = [0, 10, 15, 20, 25, 30];
                const labels = ["매우추움", "추움", "쌀쌀함", "선선함", "보통", "더움", "매우더움"];

                let index = 0;
                for (const point of tempPoints) {
                    if (isFeelLikeTemp.value <= point) break;
                    index++;
                }
                feeling.value = labels[index];

                // 가공할 or 가공한 데이터를 가지고 새로운 배열을 생성
                // 우리가 새로운 배열을 만들어주는 이유는 template 부분에서 v-for를 좀 더 편하게 쓰기 위해서
                let isProcessedData = [
                    {name: "일출시간", value: Unix_timestamp(isTimeOfSunrise.value)},
                    {name: "일몰시간", value: Unix_timestamp(isTimeOfSunset.value)},
                    {name: "가시거리", value: isLineOfSight.value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") + "M"},
                ];

                // Composition API에서 AJAX 요청과 데이터 변경을 하려면 데이터.value로 접근해야 한다.
                cityName.value = isInitialCityName;
                subWeatherData.value = isProcessedData;
            } catch (error) {
                console.log(error);
            }
        };

        watchEffect(async() =>{
            await fetchOpenWeatherApi();
        })

        onMounted(() => {
            fetchOpenWeatherApi();            
        });

        return {
            currentTime,
            cityName,
            subWeatherData,
            feeling,
        }
    }
}
</script>

<style lang="scss" scoped>
@import '~/scss/main.scss';
@import '~/scss/subview.scss';
</style>