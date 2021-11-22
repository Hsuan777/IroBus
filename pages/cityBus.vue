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
          <ul class="list-unstyled bg-primary rounded-3 p-3 magic-height-388 overflow-auto">
            <li
              v-for="item in filterCityRouteData"
              :key="item.RouteUID"
              class="border border-focus bg-primary px-2 py-1 rounded-3"
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
          </ul>
        </div>
      </div>
      <div class="col-md-4">
        站序
      </div>
      <div class="col-md-4">
        地圖
      </div>
    </div>
  </div>
</template>
<script>
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
      this.$axios.get(apiUrl).then((res) => {
        this.cityRouteData = res.data;
      });
    },
  },
};
</script>
