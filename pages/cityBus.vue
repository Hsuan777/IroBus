<template>
  <div class="container-fuild">
    <div class="row g-5">
      <div class="col-md-4">
        <div class="p-6 ms-5 bg-dark rounded">
          <!-- 搜尋欄位 -->
          <div class="d-flex mb-4">
            <input type="button" value="<" class="btn btn-link text-white d-md-none">
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
            class="scrollbar list-unstyled bg-primary rounded-3 p-3 magic-height-388 overflow-auto"
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
      <div class="col-md-4">
        <div class="d-flex">
          <input
            type="button"
            :value="`往${stopsData.departureStopNameZh}`"
            class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
            :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 1 }"
            @click="changeDirection(1)"
          >
          <input
            type="button"
            :value="`往${stopsData.destinationStopNameZh}`"
            class="btn btn-primary text-focus border-0 rounded-0 d-block w-100"
            :class="{ 'border-bottom border-white text-white': stopsData.nowDirection === 0 }"
            @click="changeDirection(0)"
          >
        </div>
        <ul class="scrollbar list-unstyled py-4 px-6 overflow-auto bg-dark vh-85">
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
                  "
                >
                  {{ formatEstimateTime(item.EstimateTime) }}
                </p>
              </div>
              <p class="fz-larger">
                {{ item.StopName.Zh_tw }}
              </p>
              <span
                class="
                  line--after
                  bg-dark
                  border border-focus border-3
                  ms-auto ms-8
                  rounded-circle
                  py-1
                  px-2
                  fz-small
                "
              >20</span>
              <template
                v-if="
                  item.PlateNumb !== undefined && item.PlateNumb !== '-1' && item.EstimateTime < 60
                "
              >
                <img src="~/assets/icon/bus.png" alt="bus" class="ms-auto me-2">
                <span class="fz-smaller">{{ item.PlateNumb }}</span>
              </template>
            </li>
          </template>
          <li v-else class="text-white">
            <p class="fz-larger">
              無站點資料
            </p>
          </li>
        </ul>
      </div>
      <div class="col-md-4">
        <div id="mapID" class="vh-85" />
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
      nowBusMarks: [],
      stopsData: {
        departureStopNameZh: '',
        destinationStopNameZh: '',
        nowDirection: 1,
      },
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
  },
  created() {
    if (process.client) {
      this.leaflet = require('leaflet');
      const markerCluster = require('leaflet.markercluster');
      this.leaflet = { ...this.leaflet, ...markerCluster };
    }
  },
  methods: {
    // 取得公車路線資料
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
    // 取得公車路線停靠站點資料
    getCityRouterStopsData(routeName, departureStopNameZh, destinationStopNameZh) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/StopOfRoute/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.cityRouterStopsData = res.data;
          this.stopsData.departureStopNameZh = departureStopNameZh;
          this.stopsData.destinationStopNameZh = destinationStopNameZh;
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
          // this.getCityRouterFaresData(routeName);
          this.changeDirection(this.stopsData.nowDirection);
          this.getBusShapeData(routeName);
        });
    },
    // 取得站點票價資料
    // getCityRouterFaresData(routeName) {
    //   const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/RouteFare/City/${this.nowCity}/${routeName}?$format=JSON`;
    //   this.$axios
    //     .get(apiUrl, {
    //       headers: this.getAuthorizationHeader(),
    //     })
    //     .then((res) => {
    //       this.cityRouterFaresData = res.data;
    //       this.changeDirection(this.stopsData.nowDirection);
    //     });
    // },
    // 取得公車路線軌跡資料
    getBusShapeData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/Shape/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          // 可能有很多不同營運業者，先取一筆圖資資料
          this.busShapeData = res.data;
          this.polyLine(res.data[0].Geometry);
          this.displayStopsMark();
        });
    },
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
      this.myMap.fitBounds(this.myLayer.getBounds());
    },
    changeDirection(value) {
      //  [0:'去程',1:'返程',2:'迴圈',255:'未知']
      this.stopsData.nowDirection = value;
      // 篩選方向
      // 可能多筆資料，因為有多台公車與不同營運業者
      // 站點座標可能在不同去程與返程上
      const tempData = this.cityRouterStopsData.filter(
        (item) => item.Direction === value && item.Stops[0].StopPosition !== undefined,
      )[0];
      // 匯入動態資料
      if (tempData) {
        tempData.Stops.forEach((item, index) => {
          this.estimatedTimeOfArrivalData.forEach((arrivalItem, arrivalIndex) => {
            if (arrivalItem.StopUID === item.StopUID) {
              tempData.Stops[index] = {
                ...tempData.Stops[index],
                ...this.estimatedTimeOfArrivalData[arrivalIndex],
              };
            }
          });
        });
        this.directionData = tempData.Stops;
      } else {
        this.directionData = [];
      }
    },
    formatEstimateTime(value) {
      if (value === 0) {
        return '已到站';
      }
      if (value > 0 && value <= 60) {
        return '即將進站';
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
    // 加入地圖
    displayMap() {
      this.myMap = this.leaflet.map('mapID', {
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
              icon: this.setIcon(20),
            })
            .bindPopup(`<p>${item.StopName.Zh_tw}</p>`),
        );
        // 加入公車目前位置(站點位置)
        if (item.PlateNumb !== undefined && item.PlateNumb !== '-1' && item.EstimateTime < 60) {
          this.nowBusMarks.push(
            this.leaflet
              .marker([item.StopPosition.PositionLat, item.StopPosition.PositionLon], {
                icon: this.setIcon(item.PlateNumb),
              })
              .bindPopup(`<p>${item.PlateNumb}</p>`),
          );
        }
      });
      this.myMap.addLayer(this.busStopsMark);
      this.nowBusMarks.forEach((item) => item.addTo(this.myMap));
    },
    setIcon(value) {
      let icon = '';
      if (Number.isInteger(value)) {
        icon = new this.leaflet.Icon({
          iconUrl: 'https://imgur.com/djnOfPC.png',
          iconSize: [34, 47],
          iconAnchor: [12, 41],
          popupAnchor: [1, -34],
          shadowSize: [41, 41],
        });
      } else {
        icon = new this.leaflet.DivIcon({
          html: '<p class="icon-larger d-flex justify-content-center align-items-center rounded-circle bg-focus text-white shadow fw-bolder">Bus</p>',
        });
      }
      return icon;
    },
  },
};
</script>
