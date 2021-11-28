<template>
  <div class="container-fuild mx-md-5">
    <div class="row g-5">
      <!-- 選擇路線 -->
      <div class="col-lg-4">
        <div class="p-6 bg-dark rounded">
          <!-- 搜尋欄位 -->
          <div class="d-flex mb-4">
            <select
              class="form-select bg-dark border-focus border-2 text-white rounded-pill me-5"
              @change="getCityRouteData($event.target.value)"
            >
              <option value="請選擇縣市" selected :hidden="nowCity !== '請選擇縣市'">
                請選擇縣市
              </option>
              <option v-for="item in cities" :key="item.nameEn" :value="item.nameEn">
                {{ item.name }}
              </option>
            </select>
            <input
              v-model="cacheSearch"
              type="text"
              class="form-control bg-dark border-focus border-2 text-white rounded-pill"
              :placeholder="`請輸入路線名稱`"
            >
          </div>
          <!-- 路線列表 -->
          <ul
            class="scrollbar list-unstyled bg-primary rounded-3 p-3 vh-100 vh-lg-71 overflow-auto"
          >
            <template v-if="cityRouteData[0]">
              <li
                v-for="item in filterCityRouteData"
                :key="item.RouteUID"
                class="border border-focus bg-primary px-2 py-1 mb-4 rounded-3"
                @click="
                  getCityRouterStopsData(
                    item.RouteName.Zh_tw,
                    item.DepartureStopNameZh,
                    item.DestinationStopNameZh,
                  )
                "
              >
                <h2 class="mb-2 text-white">
                  {{ item.RouteName.Zh_tw }}
                </h2>
                <p class="text-white d-flex">
                  {{ item.DepartureStopNameZh }} - {{ item.DestinationStopNameZh }}
                  <span class="text-focus ms-auto">{{
                    cities.filter((item) => item.nameEn === nowCity)[0].name
                  }}</span>
                </p>
              </li>
            </template>
            <li v-else class="border border-focus bg-primary px-2 py-5 rounded-3">
              <p class="text-white text-center">
                請先選擇縣市
              </p>
            </li>
          </ul>
        </div>
      </div>
      <!-- 顯示站點 -->
      <div v-if="nowCity !== '請選擇縣市'" class="col-lg-4 d-none d-lg-block">
        <div class="d-flex">
          <input
            type="button"
            :value="`往${
              stopsData.departureStopNameZh !== undefined ? stopsData.departureStopNameZh : ''
            }`"
            class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
            :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 1 }"
            @click="changeDirection(1)"
          >
          <input
            type="button"
            :value="`往${
              stopsData.destinationStopNameZh !== undefined ? stopsData.destinationStopNameZh : ''
            }`"
            class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
            :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 0 }"
            @click="changeDirection(0)"
          >
        </div>
        <ul class="scrollbar list-unstyled py-4 px-6 overflow-auto bg-dark vh-md-80">
          <template v-if="directionData[0]">
            <li
              v-for="item in directionData"
              :key="item.StopUID"
              class="d-flex align-items-center text-white mb-4"
            >
              <div class="position-relative w-10 me-4 magic-height-54">
                <img
                  src="~/static/gery.png"
                  alt="geryImg"
                  class="position-absolute top-0 start-0"
                >
                <img
                  src="~/static/Rectangle-1.png"
                  alt="Rectangle-1"
                  class="position-absolute top-70 start-50 translate-middle magic-height-12"
                >
                <p
                  class="
                    position-absolute
                    top-20
                    start-50
                    translate-middle
                    fz-smaller
                    text-center
                    bg-primary
                    w-100
                    py-1
                    rounded--top
                    text-nowrap
                  "
                >
                  {{ formatEstimateTime(item.EstimateTime) }}
                </p>
              </div>
              <p
                class="w-50"
                :class="`${item.StopName.Zh_tw.length > 6 ? 'fz-normal' : 'fz-larger'}`"
              >
                {{ item.StopName.Zh_tw }}
              </p>
              <div
                v-if="
                  item.PlateNumb !== undefined && item.PlateNumb !== '-1' && item.EstimateTime < 60
                "
                class="ms-auto d-flex align-items-center"
              >
                <img src="~/assets/icon/bus.png" alt="bus" class="me-2">
                <span class="fz-smaller">{{ item.PlateNumb }}</span>
              </div>
              <span
                class="
                  line--after
                  border border-focus border-3
                  ms-auto
                  rounded-circle
                  icon-larger
                  d-flex
                  justify-content-center
                  align-items-center
                  fz-small
                "
                :class="`${
                  item.PlateNumb !== undefined && item.PlateNumb !== '-1' && item.EstimateTime < 60
                    ? 'bg-focus'
                    : 'bg-dark'
                }`"
              >{{ item.Price }}</span>
            </li>
          </template>
          <li v-else class="text-white">
            <p class="fz-larger">
              請選擇路線或是該路線為單向迴圈。
            </p>
          </li>
        </ul>
      </div>
      <!-- 顯示地圖與對應資訊 -->
      <div class="col-lg-4 d-none d-lg-block">
        <div id="pcMap" class="vh-lg-85" />
      </div>
    </div>

    <!-- stopsModal -->
    <div
      ref="stopsModal"
      class="modal fade"
      tabindex="-1"
      aria-labelledby="stopsModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-fullscreen">
        <div class="modal-content">
          <div class="modal-header bg-primary">
            <p class="modal-title fz-larger text-white">
              {{ stopsData.busRouteName }}
            </p>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" />
          </div>
          <div class="modal-body p-0">
            <!-- 顯示站點 -->
            <div v-if="nowCity !== '請選擇縣市'" class="col-lg-4 vh-85">
              <div class="d-flex sticky-top">
                <input
                  type="button"
                  :value="`往${
                    stopsData.departureStopNameZh !== undefined ? stopsData.departureStopNameZh : ''
                  }`"
                  class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
                  :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 1 }"
                  @click="changeDirection(1)"
                >
                <input
                  type="button"
                  :value="`往${
                    stopsData.destinationStopNameZh !== undefined
                      ? stopsData.destinationStopNameZh
                      : ''
                  }`"
                  class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
                  :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 0 }"
                  @click="changeDirection(0)"
                >
              </div>
              <ul class="scrollbar list-unstyled py-4 px-6 overflow-auto bg-dark">
                <template v-if="directionData[0]">
                  <li
                    v-for="item in directionData"
                    :key="item.StopUID"
                    class="d-flex align-items-center text-white mb-4"
                  >
                    <div class="position-relative w-10 me-4 magic-height-54">
                      <img
                        src="~/static/gery.png"
                        alt="geryImg"
                        class="position-absolute top-0 start-0"
                      >
                      <img
                        src="~/static/Rectangle-1.png"
                        alt="Rectangle-1"
                        class="position-absolute top-70 start-50 translate-middle magic-height-12"
                      >
                      <p
                        class="
                          position-absolute
                          top-20
                          start-50
                          translate-middle
                          fz-smaller
                          text-center
                          bg-primary
                          w-100
                          py-1
                          rounded--top
                          text-nowrap
                        "
                      >
                        {{ formatEstimateTime(item.EstimateTime) }}
                      </p>
                    </div>
                    <p
                      class="w-50"
                      :class="`${item.StopName.Zh_tw.length > 6 ? 'fz-normal' : 'fz-larger'}`"
                    >
                      {{ item.StopName.Zh_tw }}
                    </p>
                    <div
                      v-if="
                        item.PlateNumb !== undefined &&
                          item.PlateNumb !== '-1' &&
                          item.EstimateTime < 60
                      "
                      class="ms-auto d-flex align-items-center"
                    >
                      <img src="~/assets/icon/bus.png" alt="bus" class="me-2">
                      <span class="fz-smaller">{{ item.PlateNumb }}</span>
                    </div>
                    <span
                      class="
                        line--after
                        border border-focus border-3
                        ms-auto
                        rounded-circle
                        icon-larger
                        d-flex
                        justify-content-center
                        align-items-center
                        fz-small
                      "
                      :class="`${
                        item.PlateNumb !== undefined &&
                        item.PlateNumb !== '-1' &&
                        item.EstimateTime < 60
                          ? 'bg-focus'
                          : 'bg-dark'
                      }`"
                    >{{ item.Price }}</span>
                  </li>
                </template>
                <li v-else class="text-white">
                  <p class="fz-larger vh-100">
                    請選擇路線或是該路線為單向迴圈。
                  </p>
                </li>
              </ul>
            </div>
          </div>
          <div class="modal-footer bg-dark">
            <button type="button" class="btn btn-primary" data-bs-dismiss="modal">
              Close
            </button>
            <button type="button" class="btn btn-primary" @click="changeModal">
              顯示地圖
            </button>
          </div>
        </div>
      </div>
    </div>
    <!-- mapModal -->
    <div
      ref="mapModal"
      class="modal fade"
      tabindex="-1"
      aria-labelledby="mapModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-fullscreen">
        <div class="modal-content">
          <div class="modal-header bg-primary">
            <p class="modal-title fz-larger text-white">
              {{ stopsData.busRouteName }}
            </p>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" />
          </div>
          <div class="modal-body p-0">
            <div id="mobileMap" ref="mapID" class="vh-85" />
          </div>
          <div class="modal-footer bg-dark">
            <button type="button" class="btn btn-primary" data-bs-dismiss="modal">
              Close
            </button>
            <button type="button" class="btn btn-primary" @click="changeModal">
              顯示站點
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import JsSHA from 'jssha';
import Wicket from 'wicket';

export default {
  data() {
    return {
      cities: [
        {
          name: '臺北市',
          nameEn: 'Taipei',
        },
        {
          name: '新北市',
          nameEn: 'NewTaipei',
        },
        {
          name: '桃園市',
          nameEn: 'Taoyuan',
        },
        {
          name: '臺中市',
          nameEn: 'Taichung',
        },
        {
          name: '臺南市',
          nameEn: 'Tainan',
        },
        {
          name: '高雄市',
          nameEn: 'Kaohsiung',
        },
        {
          name: '基隆市',
          nameEn: 'Keelung',
        },
        {
          name: '新竹市',
          nameEn: 'Hsinchu',
        },
        {
          name: '新竹縣',
          nameEn: 'HsinchuCounty',
        },
        {
          name: '苗栗縣',
          nameEn: 'MiaoliCounty',
        },
        {
          name: '彰化縣',
          nameEn: 'ChanghuaCounty',
        },
        {
          name: '南投縣',
          nameEn: 'NantouCounty',
        },
        {
          name: '雲林縣',
          nameEn: 'YunlinCounty',
        },
        {
          name: '嘉義縣',
          nameEn: 'ChiayiCounty',
        },
        {
          name: '嘉義市',
          nameEn: 'Chiayi',
        },
        {
          name: '屏東縣',
          nameEn: 'PingtungCounty',
        },
        {
          name: '宜蘭縣',
          nameEn: 'YilanCounty',
        },
        {
          name: '花蓮縣',
          nameEn: 'HualienCounty',
        },
        {
          name: '臺東縣',
          nameEn: 'TaitungCounty',
        },
        {
          name: '金門縣',
          nameEn: 'KinmenCounty',
        },
        {
          name: '澎湖縣',
          nameEn: 'PenghuCounty',
        },
        {
          name: '連江縣',
          nameEn: 'LienchiangCounty',
        },
      ],
      nowCity: '請選擇縣市',
      cacheSearch: '',
      cityRouteData: [],
      cityRouterStopsData: [],
      estimatedTimeOfArrivalData: [],
      cityRouterFaresData: [],
      busShapeData: [],
      busScheduleData: [],
      nowBusMarks: [],
      busRealTimeMarks: [],
      busRealTimeData: [],
      stopsData: {},
      directionData: [],
    };
  },
  computed: {
    filterCityRouteData() {
      return this.cityRouteData.filter((item) => item.RouteName.Zh_tw.match(this.cacheSearch));
    },
  },
  mounted() {
    this.displayMap();
    const Modal = this.modal;
    this.stopsModal = new Modal(this.$refs.stopsModal);
    this.mapModal = new Modal(this.$refs.mapModal);
    this.$refs.mapModal.addEventListener('shown.bs.modal', () => {
      setTimeout(() => {
        this.myMap.invalidateSize();
      }, 1);
    });
  },
  created() {
    if (process.client) {
      this.leaflet = require('leaflet');
      this.modal = require('bootstrap/js/dist/modal');
      const markerCluster = require('leaflet.markercluster');
      this.leaflet = { ...this.leaflet, ...markerCluster };
    }
  },
  methods: {
    // 選擇縣市後，取得公車路線資料
    getCityRouteData(city) {
      this.nowCity = city;
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/Route/City/${city}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.cityRouteData = res.data;
        });
    },
    // 取得公車路線停靠站點資料，為「主要」啟動後續動作
    getCityRouterStopsData(routeName, departureStopName, destinationStopName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/StopOfRoute/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.cityRouterStopsData = res.data;
          this.stopsData = {
            busRouteName: routeName,
            departureStopNameZh: departureStopName,
            destinationStopNameZh: destinationStopName,
            nowDirection: 1,
          };
          this.getEstimatedTimeOfArrivalData(routeName);
        });
    },
    // 取得公車站點預估到站時間資料
    getEstimatedTimeOfArrivalData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/EstimatedTimeOfArrival/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.estimatedTimeOfArrivalData = res.data;
          this.getCityRouterFaresData(routeName);
          this.getBusShapeData(routeName);
          this.getBusRealTimeData(routeName);
          this.getBusScheduleData(routeName);
        });
    },
    // 取得站點票價資料
    getCityRouterFaresData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/RouteFare/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.cityRouterFaresData = res.data;
          this.changeDirection(this.stopsData.nowDirection);
        })
        .catch(() => {
          this.cityRouterFaresData = [];
          this.changeDirection(this.stopsData.nowDirection);
        });
    },
    // 取得公車路線軌跡資料
    getBusShapeData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/Shape/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.busShapeData = res.data;
        });
    },
    // 取得公車定時資料 - 公車 GPS
    getBusRealTimeData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/RealTimeByFrequency/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.busRealTimeData = res.data;
        });
    },
    // 取得公車時刻表資料
    getBusScheduleData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/Schedule/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.busScheduleData = res.data;
        });
    },
    // 繪製路線圖
    polyLine(geo) {
      const wicket = new Wicket.Wkt();
      const geojsonFeature = wicket.read(geo).toJson();
      const myStyle = {
        color: '#C0A7C4',
        weight: 5,
        opacity: 1,
      };
      if (this.myLayer) {
        this.myMap.removeLayer(this.myLayer);
      }
      this.myLayer = this.leaflet
        .geoJSON(geojsonFeature, {
          style: myStyle,
        })
        .addTo(this.myMap);
      this.myLayer.addData(geojsonFeature);
      // zoom the map to the layer
      // 開啟 modal 會遇到找不到地圖路徑問題
      if (window.screen.width < 992) {
        setTimeout(() => {
          this.myMap.invalidateSize();
        }, 200);
        setTimeout(() => {
          this.myMap.fitBounds(this.myLayer.getBounds());
        }, 500);
      } else {
        this.myMap.fitBounds(this.myLayer.getBounds());
      }
    },
    // 變更方向資料
    changeDirection(value) {
      //  [0:'去程',1:'返程',2:'迴圈',255:'未知']
      this.stopsData.nowDirection = value;
      // 篩選方向
      // 可能多筆資料，因為有多台公車與不同營運業者
      // 站點座標可能在不同去程與返程上
      const tempData = this.cityRouterStopsData.filter(
        (item) => item.Direction === value && item.Stops[0].StopPosition !== undefined,
      )[0];
      // 匯入「動態預估時間」與「票價」
      if (tempData) {
        tempData.Stops.forEach((item, index) => {
          this.estimatedTimeOfArrivalData.forEach((arrivalItem, arrivalIndex) => {
            if (arrivalItem.StopUID === item.StopUID) {
              tempData.Stops[index] = {
                ...tempData.Stops[index],
                ...this.estimatedTimeOfArrivalData[arrivalIndex],
                Price: this.searchFares(tempData.Stops[0].StopName.Zh_tw, item.StopName.Zh_tw),
              };
            }
          });
        });
        this.directionData = tempData.Stops;
      } else {
        this.directionData = [];
      }
      if (window.screen.width < 992) {
        this.stopsModal.show();
      } else {
        this.polyLine(this.busShapeData[0].Geometry);
        this.displayStopsMark();
        this.displayBusRealTimeMark();
      }
    },
    changeModal() {
      this.stopsModal.toggle();
      this.mapModal.toggle();
      // 可能有很多不同營運業者，先取一筆圖資資料
      this.polyLine(this.busShapeData[0].Geometry);
      this.displayStopsMark();
      this.displayBusRealTimeMark();
    },
    // 判斷預估時間回報狀態
    formatEstimateTime(value) {
      if (value === 0) {
        return '已到站';
      }
      if (value > 0 && value <= 60) {
        return '即進站';
      }
      if (value === -1) {
        return '未行駛';
      }
      if (value) {
        return `${Math.ceil(value / 60)} 分`;
      }
      return '已過站';
    },
    getAuthorizationHeader() {
      const AppID = '3209d3c409014e8cb42b5e83f861c102';
      const AppKey = 'qd4_Nh2b30-edGu3vXmbDzaWTFU';
      const GMTString = new Date().toGMTString();
      const ShaObj = new JsSHA('SHA-1', 'TEXT');
      ShaObj.setHMACKey(AppKey, 'TEXT');
      ShaObj.update(`x-date: ${GMTString}`);
      const HMAC = ShaObj.getHMAC('B64');
      const Authorization = `hmac username="${AppID}", algorithm="hmac-sha1", headers="x-date", signature="${HMAC}"`;
      return { Authorization, 'X-Date': GMTString };
    },
    // 顯示基底地圖
    displayMap() {
      let map = 'pcMap';
      if (window.screen.width < 992) {
        map = 'mobileMap';
      }
      this.myMap = this.leaflet.map(map, {
        center: [25.0462, 121.5174],
        zoom: 13,
      });
      // 加入圖層
      this.leaflet
        .tileLayer(
          'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}',
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: 'mapbox/streets-v11',
            tileSize: 512,
            zoomOffset: -1,
            accessToken:
              'pk.eyJ1IjoidmljMzMzIiwiYSI6ImNrdXRqNHBjbTVwa3Myb21uc3hrajJkeHEifQ.TeelvbEBo9SIQjYpUdv70g',
          },
        )
        .addTo(this.myMap);
    },
    // 顯示路線上各站點
    displayStopsMark() {
      if (this.busStopsMark) {
        this.myMap.removeLayer(this.busStopsMark);
      }
      if (this.nowBusMarks[0]) {
        this.nowBusMarks.forEach((item) => this.myMap.removeLayer(item));
      }
      this.busStopsMark = this.leaflet.markerClusterGroup();
      this.directionData.forEach((item) => {
        // 加入站點座標
        this.busStopsMark.addLayer(
          this.leaflet
            .marker([item.StopPosition.PositionLat, item.StopPosition.PositionLon], {
              icon: this.setIcon(item.Price),
            })
            .bindPopup(`<p>${item.StopName.Zh_tw}</p>`),
        );
      });
      this.myMap.addLayer(this.busStopsMark);
      this.nowBusMarks.forEach((item) => item.addTo(this.myMap));
    },
    // 顯示目前公車 GPS 位置
    displayBusRealTimeMark() {
      if (this.busRealTimeMarks[0]) {
        this.busRealTimeMarks.forEach((item) => this.myMap.removeLayer(item));
      }
      this.busRealTimeData.forEach((item) => {
        if (item.Direction === this.stopsData.nowDirection) {
          this.busRealTimeMarks.push(
            this.leaflet
              .marker([item.BusPosition.PositionLat, item.BusPosition.PositionLon], {
                icon: this.setIcon(item.PlateNumb),
              })
              .bindPopup(
                `<p>車牌號碼: ${item.PlateNumb}</p>
              <p>目的地: ${
  item.Direction === 1
    ? this.stopsData.departureStopNameZh
    : this.stopsData.destinationStopNameZh
}</p>`,
              )
              .addTo(this.myMap),
          );
        }
      });
    },
    setIcon(value) {
      let icon = '';
      if (Number.isInteger(value) || value === '?') {
        icon = new this.leaflet.DivIcon({
          html: `<p class="icon icon__station line__station--after rounded-circle bg-white fw-bolder border border-dark border-3 fz-smaller">
          ${value}
          </p>`,
        });
      } else {
        icon = new this.leaflet.DivIcon({
          html: `<p class="icon icon__bus rounded-circle bg-third text-white fw-bolder">${value}</p>`,
        });
      }
      return icon;
    },
    // 對比起始站與終點站之票價
    searchFares(startStation, endStation) {
      // 票價放在 ODFares 內，資料為站點之排列組合
      // 只需要取得「起始站」與「目的地停靠站」，就有各站價格
      // RouteFare 資料可能只有一筆，相反路線則反向排序
      // 部分城市可能有不同的規則
      if (startStation === endStation) {
        return 0;
      }
      if (this.cityRouterFaresData[0]) {
        return this.cityRouterFaresData[0].ODFares.filter(
          (item) => item.OriginStop.StopName === startStation
            && item.DestinationStop.StopName === endStation,
        )[0].Fares[0].Price;
      }
      return '?';
    },
  },
};
</script>
