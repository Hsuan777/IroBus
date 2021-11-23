<template>
  <div class="container-fuild">
    <div class="row">
      <div class="col-md-4">
        <div class="mx-6 p-6 bg-dark rounded">
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
                  py-1 px-2
                  fz-small
                "
              >20</span>
              <template v-if="item.PlateNumb">
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
        地圖
      </div>
    </div>
  </div>
</template>
<script>
import JsSHA from 'jssha';

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
  methods: {
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
    getEstimatedTimeOfArrivalData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/EstimatedTimeOfArrival/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.estimatedTimeOfArrivalData = res.data;
          this.getCityRouterFaresData(routeName);
        });
    },
    getCityRouterFaresData(routeName) {
      const apiUrl = `https://ptx.transportdata.tw/MOTC/v2/Bus/RouteFare/City/${this.nowCity}/${routeName}?$format=JSON`;
      this.$axios
        .get(apiUrl, {
          headers: this.getAuthorizationHeader(),
        })
        .then((res) => {
          this.cityRouterFaresData = res.data;
          console.log(this.cityRouterFaresData);
          this.changeDirection(this.stopsData.nowDirection);
        });
    },
    changeDirection(value) {
      //  [0:'去程',1:'返程',2:'迴圈',255:'未知']
      this.stopsData.nowDirection = value;
      // 篩選方向
      const tempData = this.cityRouterStopsData.filter((item) => item.Direction === value)[0];
      // 匯入動態資料
      if (tempData) {
        tempData.Stops.forEach((item, index) => {
          this.estimatedTimeOfArrivalData.forEach((arrivalItem, arrivalIndex) => {
            if (arrivalItem.StopUID === item.StopUID) {
              tempData.Stops[index] = {
                ...tempData[index],
                ...this.estimatedTimeOfArrivalData[arrivalIndex],
              };
            }
          });
        });
        this.directionData = tempData.Stops;
        console.log(this.directionData);
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
      return '已離站';
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
  },
};
</script>
