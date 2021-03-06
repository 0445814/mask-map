<template>
  <div id="app" class="w-100 h-100">
    <!-- Modal -->
    <Modal>
      <h5 slot="modal-title">注意事項</h5>
      <div slot="modal-content">
        <p><i class="fal fa-stars mr-1"></i>目前地圖僅提供簡單查詢功能</p>
        <p>
          <i class="fal fa-stars mr-1"></i>口罩數量請以
          <span class="font-weight-bold">實際藥局存量為主</span>
        </p>
        <p><i class="fal fa-stars mr-1"></i>為了方便查詢，請開啟存取位置權限</p>
        <p>
          <i class="fal fa-stars mr-1"></i>若需要更多資訊請參考
          <a href="https://g0v.hackmd.io/gGrOI4_aTsmpoMfLP1OU4A" target="_blank"
            >口罩供需資訊平台
          </a>
        </p>
      </div>
      <div slot="modal-content-img">
        <img
          class="d-block"
          src="@/assets/images/smartphone_map_app_woman.png"
          width="150"
          alt=""
        />
      </div>
    </Modal>

    <!-- Loading -->
    <Loading v-if="isLoading">
      <p class="text-white m-0 h3">資料讀取中 ...</p>
    </Loading>
    <!--    導覽列     -->
    <div class="board position-absolute bg-white" :class="{ active: isActive }" v-else>
      <button
        class="board-btn position-absolute btn text-white d-flex justify-content-center"
        @click="isActive = !isActive"
      >
        <i class="far fa-angle-left"></i>
      </button>
      <!--    導覽列 header     -->
      <div class="board-header bg-mask-primary">
        <div class="board-header__info d-flex justify-content-around align-items-center pt-2 pb-2">
          <div class="header-header__date text-white">
            <p>{{ today.date }}</p>
            <p class="h3">{{ today.weekDay }}</p>
          </div>

          <div class="board-header__avatar">
            <img
              src="@/assets/images/medical_mask07_businessman.png"
              width="70"
              alt="man with mask"
            />
            <img
              class="pt-3"
              src="@/assets/images/medical_mask02_girl.png"
              width="60"
              alt="girl with mask"
            />
          </div>
        </div>

        <div class="board-header__restriction text-center text-white mb-3">
          <p class="m-0" v-if="today.weekDayNum % 2 === 0 && today.weekDayNum !== 0">
            身分證末碼為 <span class="h3">2,4,6,8,0</span> 可購買
          </p>
          <p class="m-0" v-else-if="today.weekDayNum % 2 !== 0">
            身分證末碼為 <span class="h3">1,3,5,7,9</span> 可購買
          </p>
          <p class="m-0" v-else>週日不限定身份證字號，<span class="h3">皆可購買</span></p>
        </div>

        <div class="board-header__search pb-3 px-3">
          <div class="d-flex justify-content-between mb-2">
            <select
              class="board-header__select form-control"
              v-model="selectedCity"
              @change="changePosition('選擇城市')"
            >
              <option :value="city.CityName" v-for="city in county" :key="city.CityName">
                {{ city.CityName }}
              </option>
            </select>
            <select
              class="board-header__select form-control"
              v-model="selectedDistrict"
              @change="changePosition('選擇行政區')"
            >
              <option
                :value="district.AreaName"
                v-for="district in filteredDistricts"
                :key="district.AreaName"
                >{{ district.AreaName }}</option
              >
            </select>
          </div>

          <div class="position-relative mb-2">
            <input
              type="text"
              class="form-control mx-auto"
              placeholder="搜索區域, 地址, 藥局"
              @focus="isFocus = true"
              @blur="focusOut"
              @input="searchWord = $event.target.value"
            />
            <ul class="list-group position-absolute w-100 list-unstyled bg-white" v-show="isFocus">
              <li class="list-group-item p-0">
                <div class="search-list p-2 mt-2" @click="relocate">
                  <p class="mb-0"><i class="fal fa-location fa-lg mr-1"></i>重新定位您的位置</p>
                </div>
              </li>
              <li
                class="list-group-item p-0"
                v-for="item in searchPharmacies"
                :key="item.properties.id"
              >
                <div class="search-list p-2" @click="changePosition('選擇藥局', item)">
                  <p class="search-list-name mb-0 h5">{{ item.properties.name }}</p>
                  <p class="search-list-address text-secondary mb-0">
                    {{ item.properties.address }}
                  </p>
                </div>
              </li>
            </ul>
          </div>

          <div class="d-flex justify-content-between text-white">
            <span class="mr-2">僅顯示仍有存量</span>
            <div class="custom-control custom-checkbox custom-control-inline mr-0">
              <input
                type="checkbox"
                class="custom-control-input"
                id="adult-stock"
                value="mask_adult"
                v-model="stock"
              />
              <label class="custom-control-label" for="adult-stock">成人</label>
            </div>

            <div class="custom-control custom-checkbox custom-control-inline mr-0">
              <input
                type="checkbox"
                class="custom-control-input"
                id="child-stock"
                value="mask_child"
                v-model="stock"
              />
              <label class="custom-control-label" for="child-stock">兒童</label>
            </div>
          </div>
        </div>
      </div>

      <div class="board-body" v-if="filteredPharmacies.length">
        <div
          class="board-body__item p-2 border-bottom"
          v-for="item in filteredPharmacies"
          :key="item.id"
          @click="changePosition('選擇藥局', item)"
        >
          <h2 class="pharmacy-title h4">{{ item.properties.name }}</h2>
          <p class="text-secondary">{{ item.properties.address }}</p>
          <p class="text-secondary">{{ item.properties.phone }}</p>
          <div class="d-flex justify-content-around mb-2">
            <div
              class="mask-num mask-num__adult text-white
              d-flex justify-content-around align-items-center mr-1"
              :class="item.properties.mask_adult > 0 ? 'bg-mask-primary' : 'bg-mask-none'"
            >
              <img
                class="d-inline-block position-relative"
                :src="getAvatar('adult', item.properties.mask_adult)"
                width="40"
                alt="woman-avatar"
                style="top: 2px;"
              />
              {{ item.properties.mask_adult }}
            </div>
            <div
              class="mask-num mask-num__child  text-white
              d-flex justify-content-around align-items-center ml-1"
              :class="item.properties.mask_child > 0 ? 'bg-mask-secondary' : 'bg-mask-none'"
            >
              <img
                class="d-inline-block position-relative"
                :src="getAvatar('child', item.properties.mask_child)"
                width="40"
                alt="baby-avatar"
                style="top: 6px;"
              />
              {{ item.properties.mask_child }}
            </div>
          </div>
        </div>
      </div>
      <div class="d-flex flex-column align-items-center" v-else>
        <img
          class="d-inline-block pt-2 mt-5 mb-3"
          src="@/assets/images/no_masks.png"
          width="120"
          alt="dog-avatar"
        />
        <p class="h5">抱歉，這區域乎沒有藥局 ...</p>
        <p class="h5">試試搜尋別的地方吧 :D</p>
      </div>
    </div>
    <div id="mask-map" class="w-100 h-100" :class="{ active: isActive }"></div>
  </div>
</template>

<script>
/* eslint-disable global-require */
/* eslint-disable import/no-unresolved */

import L from 'leaflet';
import 'leaflet.markercluster';
import { format, getDay } from 'date-fns';

export default {
  data() {
    return {
      // 讀取畫面
      isLoading: true,
      // 是否啟用左方導覽版
      isActive: true,
      // 搜尋欄位是否 focus
      isFocus: false,
      // 人物圖片
      avatar: {
        adult: {
          laugh: require('@/assets/images/face_smile_woman4.png'),
          smile: require('@/assets/images/face_smile_woman2.png'),
          shock: require('@/assets/images/necchusyou_face_girl2.png'),
          cry: require('@/assets/images/necchusyou_face_girl4.png'),
        },
        child: {
          laugh: require('@/assets/images/baby_boy04_laugh.png'),
          smile: require('@/assets/images/baby_boy01_smile.png'),
          shock: require('@/assets/images/baby_boy06_shock.png'),
          cry: require('@/assets/images/baby_boy03_cry.png'),
        },
      },
      // 藥局原始資料
      pharmacies: [],
      // 台灣縣市名稱原始資料
      county: [],
      /**
       * 選擇的城市名稱與行政區，預設為臺北市中正區
       */
      selectedCity: '臺北市',
      selectedDistrict: '中正區',
      /**
       * date: 日期 YYYY/MM/DD
       * weekDayNum: 星期幾 數字
       * weekDay: 星期幾 文字
       */
      today: {
        date: '',
        weekDayNum: null,
        weekDay: '',
      },
      // 搜尋關鍵字
      searchWord: '',
      // 口罩存量選擇
      stock: [],
    };
  },
  components: {
    Loading: () => import('@/components/Loading'),
    Modal: () => import('@/components/Modal.vue'),
  },
  computed: {
    filteredDistricts() {
      const vm = this;
      let districts = [];
      vm.county.forEach((item) => {
        if (vm.selectedCity === item.CityName) {
          districts = item.AreaList;
        }
      });

      return districts;
    },
    filteredPharmacies() {
      const vm = this;
      let pharmacies = [];
      if (vm.stock.length === 0) {
        pharmacies = vm.pharmacies.filter(
          (i) => i.properties.county === vm.selectedCity
          && i.properties.town === vm.selectedDistrict,
        );
      } else if (vm.stock.length === 1) {
        pharmacies = vm.pharmacies.filter(
          (i) => i.properties.county === vm.selectedCity
          && i.properties.town === vm.selectedDistrict
          && i.properties[vm.stock[0]] > 0,
        );
      } else {
        pharmacies = vm.pharmacies.filter(
          (i) => i.properties.county === vm.selectedCity
          && i.properties.town === vm.selectedDistrict
          && i.properties[vm.stock[0]] > 0
          && i.properties[vm.stock[1]] > 0,
        );
      }

      return pharmacies;
    },
    searchPharmacies() {
      const vm = this;
      return vm.searchWord === ''
        ? []
        : vm.pharmacies
          .filter(
            (item) => item.properties.name.includes(vm.searchWord)
                || item.properties.address.includes(vm.searchWord)
                || item.properties.town.includes(vm.searchWord),
          )
          .splice(0, 10);
    },
  },
  methods: {
    getAvatar(type, maskNum) {
      const vm = this;
      let avatar;
      if (maskNum >= 500) {
        avatar = vm.avatar[type].laugh;
      } else if (maskNum < 500 && maskNum >= 200) {
        avatar = vm.avatar[type].smile;
      } else if (maskNum < 200 && maskNum > 0) {
        avatar = vm.avatar[type].shock;
      } else if (maskNum === 0) {
        avatar = vm.avatar[type].cry;
      }
      return avatar;
    },
    /**
     * 取得當天的日期
     */
    getToday() {
      const vm = this;
      const date = format(new Date(), 'yyyy-MM-dd');
      vm.today.date = date;

      // 得到當天星期的數字判斷中文日期
      const weekDayNum = getDay(new Date());
      vm.today.weekDayNum = weekDayNum;

      switch (weekDayNum) {
        case 1:
          vm.today.weekDay = '星期一';
          break;
        case 2:
          vm.today.weekDay = '星期二';
          break;
        case 3:
          vm.today.weekDay = '星期三';
          break;
        case 4:
          vm.today.weekDay = '星期四';
          break;
        case 5:
          vm.today.weekDay = '星期五';
          break;
        case 6:
          vm.today.weekDay = '星期六';
          break;
        default:
          vm.today.weekDay = '星期天';
          break;
      }
    },

    /**
     * 取得台灣縣市鎮名稱資料
     */
    getCounty() {
      /**
       * * 等之後開 API 再來用
       */
      // const vm = this;

      // vm.axios.get(`${process.env.VUE_APP_APIPATH}`).then((res) => {
      //   vm.county = res.data;
      // });

      // * 偷懶先用這招
      this.county = require('../public/taiwanCounty.json');
    },

    // 取得藥局圖標與判斷顏色
    getIcon(maskSum) {
      /**
       * 成人口罩與兒童口罩總數判斷座標顏色
       *
       * sum >= 500 --> Green
       * 200 <= sum < 500 --> Gold
       * 0 < sum < 200 --> Red
       * sum == 0 --> Black
       */
      let iconColor;
      if (maskSum >= 200 && maskSum < 500) {
        iconColor = 'gold';
      } else if (maskSum > 0 && maskSum < 200) {
        iconColor = 'red';
      } else if (maskSum === 0) {
        iconColor = 'grey';
      } else {
        iconColor = 'green';
      }
      // 創建藥局位置圖標
      return new L.Icon({
        iconUrl: `https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-${iconColor}.png`,
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });
    },
    // 取得藥局 Popup 資訊
    getPharmacyPopup(pharmacy) {
      return `
            <h1 class="h5 font-weight-bold">${pharmacy.properties.name}</h1>
            <p class="text-secondary mt-0 mb-2"><i class="fas fa-map-marker-alt pr-2" style="padding-left: 2.5px;"></i>${
  pharmacy.properties.address
}</p>
            <p class="text-secondary mt-0 mb-2"><i class="fal fa-phone-rotary pr-2" style="padding-left: 1.5px;"></i>${
  pharmacy.properties.phone
}</p>
            <div class="d-flex justify-content-around mb-2">
              <div class="mask-num mask-num__adult text-white d-flex justify-content-center align-items-center mr-1 ${
  pharmacy.properties.mask_adult > 0 ? 'bg-mask-primary' : 'bg-mask-none'
}">
                成人口罩: ${pharmacy.properties.mask_adult}
              </div>
              <div class="mask-num mask-num__child text-white d-flex justify-content-center align-items-center ml-1 ${
  pharmacy.properties.mask_child > 0 ? 'bg-mask-secondary' : 'bg-mask-none'
}">
                兒童口罩: ${pharmacy.properties.mask_child}
              </div>
            </div>
            <p class="text-secondary mt-0 mb-2"><i class="fal fa-cloud-upload pr-2"></i>${
  pharmacy.properties.updated !== ''
    ? pharmacy.properties.updated
    : '<span class="text-danger">抱歉，未取得資料 <i class="fal fa-sad-tear"></i></span>'
}</p>`;
    },

    /**
     * 改變地圖位置至選擇的藥局的地點
     */
    changePosition(type, selectedPharmacy) {
      const vm = this;
      let pharmacyPlace;
      /**
       * ! 當選擇的城市改變時，會默認將 filteredDistricts 行政區陣列資料的第一筆資料賦值到 selectedDistrict，否則，行政區的選項第一個會是空白。
       * * 點擊左方藥局欄時，跳出 Popup 視窗。
       */
      if (type === '選擇城市') {
        vm.selectedDistrict = vm.filteredDistricts[0].AreaName;
        [pharmacyPlace] = vm.filteredPharmacies;
      } else if (type === '選擇行政區') {
        [pharmacyPlace] = vm.filteredPharmacies;
      } else {
        pharmacyPlace = selectedPharmacy;

        // iPad 大小以下會收起左方導覽列
        if (window.document.body.clientWidth <= 768) {
          vm.isActive = false;
        }
        const maskSum = pharmacyPlace.properties.mask_adult + pharmacyPlace.properties.mask_child;
        const marker = L.marker(
          [pharmacyPlace.geometry.coordinates[1], pharmacyPlace.geometry.coordinates[0]],
          vm.getIcon(maskSum),
        ).bindPopup(vm.getPharmacyPopup(pharmacyPlace), {
          maxWidth: 999,
        });

        // 跳出 Popup 後，移除
        marker.addTo(window.map).openPopup();
        marker.on('popupclose', () => {
          window.map.removeLayer(marker);
        });
      }

      // 如果選擇的行政區沒有藥局，則不移動
      if (vm.filteredPharmacies.length === 0) return;

      window.map.setView(
        [pharmacyPlace.geometry.coordinates[1], pharmacyPlace.geometry.coordinates[0]],
        16,
      );
    },

    /**
     * 如果使用者拒絕提供地理位置，則默認顯示為第一筆藥局位置
     */
    firstLocate() {
      window.navigator.geolocation.watchPosition(() => this.relocate());
    },
    relocate() {
      window.navigator.geolocation.getCurrentPosition((position) => {
        window.map.setView([position.coords.latitude, position.coords.longitude], 16);
      });
    },
    /**
     * 點擊搜尋欄出現下拉選單，再點擊選擇的藥局時，由於 blur 會比 click 還早觸發
     * 所以加 setTimeout 來延遲
     */
    focusOut() {
      const vm = this;
      setTimeout(() => {
        vm.isFocus = false;
      }, 150);
    },
  },
  created() {
    const vm = this;
    vm.getToday();
    vm.getCounty();
  },
  // updated() {
  //   const vm = this;
  //   setTimeout(() => {
  //     vm.isLoading = false;
  //   }, 2000);
  // },
  async mounted() {
    const vm = this;

    // 取得藥局資料
    await vm.axios
      .get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
      .then((res) => {
        vm.pharmacies = res.data.features;
      });

    const map = L.map('mask-map', {
      center: [25.058709, 121.558489],
      zoom: 7,
      minZoom: 7,
      maxBounds: L.latLngBounds(L.latLng(28, 115), L.latLng(20, 127)),
      zoomControl: false,
    });

    vm.relocate();
    // 將 map 設置成全域變數
    window.map = map;

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(map);

    const markers = new L.MarkerClusterGroup().addTo(map);

    // 匯入至地圖中
    for (let i = 0; i < vm.pharmacies.length; i += 1) {
      const maskAdult = vm.pharmacies[i].properties.mask_adult;
      const maskChild = vm.pharmacies[i].properties.mask_child;
      const maskSum = maskAdult + maskChild;
      // 取得 藥局位置 icon
      const icon = vm.getIcon(maskSum);
      // vm.setAvatar(maskAdult, maskChild);
      markers.addLayer(
        L.marker(
          [vm.pharmacies[i].geometry.coordinates[1], vm.pharmacies[i].geometry.coordinates[0]],
          {
            icon,
          },
        )
          .bindPopup(vm.getPharmacyPopup(vm.pharmacies[i]), {
            maxWidth: 999,
          })
          .openPopup(),
      );
    }

    vm.isLoading = false;
  },
};
</script>

<style lang="scss">
html,
body {
  width: 100%;
  height: 100vh;
  // 避免放大縮小破版
  overflow: hidden;
  font-family: Microsoft JhengHei;
}

$mask-primary: #73c0d8;
$mask-secondary: #ffa573;
$mask-none: #a5a5a5;

.bg-mask-primary {
  background: $mask-primary;
}

.bg-mask-secondary {
  background: $mask-secondary;
}

.bg-mask-none {
  background: $mask-none;
}

#mask-map {
  transition: all 1s;
  &.active {
    transform: translateX(300px);
    transition: all 0.5s;
  }
}

.board {
  width: 300px;
  height: 100vh;
  box-shadow: 1px 1px 7px 0px rgba(0, 0, 0, 0.5);
  transition: all 1s;
  left: 0;
  top: 0;
  z-index: 999;
  transform: translateX(-300px);
  &.active {
    transform: translateX(-0%);
    transition: all 0.5s;
  }

  &-header {
    // height: 100px;

    &__restriction {
      span {
        color: yellow;
      }
    }

    &__search {
      input {
        // border-radius: 20px;
        // width: 270px;
        height: 30px;
        &::placeholder {
          color: rgba(128, 128, 128, 0.6);
          font-size: 14px;
        }
      }

      .list-group {
        // border-radius: 0 0 20px 20px;
        z-index: 10;
        .list-group-item {
          top: -5px;
          // border-top: 0;
          // border-color: 1px solid #ced4da;
          // border: 0px 1px 1px 1px solid #ced4da !important;
          border-width: 0px 1px 1px 1px;
          border-style: solid;
          border-color: #ced4da;
          // border-bottom: 0;
          &:first-child {
            border-top-left-radius: 0;
            border-top-right-radius: 0;
          }
          .search-list {
            cursor: pointer;

            &:hover {
              background: rgba(115, 192, 216, 0.3);
            }

            &-address {
              font-size: 12px;
            }
          }
        }
      }
    }

    &__select {
      width: 100px;
      height: 35px;
    }
  }

  &-body {
    // 這樣才會出現下拉卷軸
    height: 100%;
    overflow: auto;

    &__item {
      cursor: pointer;
      &:hover {
        background: rgba(0, 0, 0, 0.05);
      }
    }
  }
}
::-webkit-scrollbar {
  display: none;
}

// 隱藏地圖控制鈕
.leaflet-control-container {
  display: none;
}

.mask-num {
  width: 120px;
  min-height: 36px;
  border-radius: 16px;
}

.board-btn {
  width: 17px;
  height: 50px;
  background: $mask-primary;
  border-radius: 0px 5px 5px 0px;
  box-shadow: 2px 3px 6px #00000029;
  right: -26px;
  top: 281px;
}

input[type="text"]:focus {
  box-shadow: none;
  border: 1px solid #ced4da;
}
</style>
