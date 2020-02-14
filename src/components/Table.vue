<template>
  <div class="table">
    <b-table
      ref="selectableTable"
      selectable
      :select-mode="selectMode"
      :busy.sync="isBusy"
      :items="userInfo"
      :fields="fields"
      @row-selected="onRowSelected"
      responsive="sm"
    >
      <template v-slot:table-busy>
        <div class="text-center text-danger my-2">
          <b-spinner class="align-middle"></b-spinner>
          <strong>Загружаю данные...</strong>
        </div>
      </template>
    </b-table>

    <b-modal ref="graph-modal" hide-footer>
      <template
        v-slot:modal-title
      >{{ selected && selected[0].first_name }} {{ selected && selected[0].last_name }} WiFi</template>

      <b-container fluid >
        <b-row class="text-center" align-v="center">
          <b-col cols="1">
            <div>RSSI</div>
          </b-col>
          <b-col class="mt-3" ref="graph-container" >
            <bar-graph
              title="RSSI"
              animDuration="1s"
              :points="rssi"
              :width="graphW"
              :height="graphH"
            />
          </b-col>
        </b-row>
        <b-row class="text-center" align-v="center">
          <b-col cols="1">
            <div>TS</div>
          </b-col>
          <b-col class="mt-3">
            <bar-graph title="TS" animDuration="1s" :points="ts" :width="graphW" :height="graphH" />
          </b-col>
        </b-row>
      </b-container>
      <b-button class="mt-3" block @click="hideModal">Close Me</b-button>
    </b-modal>

    <div class="api-info">{{ userInfo }}</div>
  </div>
</template>

<script>
import axios from "axios";
import BarGraph from "vue-svg-charts/src/components/bar";
//import ky from "ky";

//const API_TEST = 'https://api.coindesk.com/v1/bpi/currentprice.json';
const API_TEST = "https://rssi.wmrk.tk/";

const WORK_API = `https://cors-anywhere.herokuapp.com/${API_TEST}`;

export default {
  name: "Table",
  components: { BarGraph },
  data: function() {
    return {
      fields: [
        {
          key: "first_name",
          label: "Имя",
          sortable: true
        },
        {
          key: "last_name",
          label: "Фамилия",
          sortable: true
        },
        {
          key: "mac",
          label: "MAC-адрес",
          sortable: true
        },
        {
          key: "phone",
          label: "Телефон",
          sortable: true
        }
      ],

      selectMode: "single",
      selected: [],

      rssi: [],
      ts: [],

      userInfo: [],
      isBusy: false,

      graphW: 256,
      graphH: 128
    };
  },
  methods: {
    showModal() {
      this.$refs["graph-modal"].show();
      
    },
    hideModal() {
      this.$refs["graph-modal"].hide();
    },
    onRowSelected(items) {
      this.rssi = [];
      this.ts = [];
      this.selected = items;

      // получаем данные сигнала выбранного элемента
      const signals = items[0].signals;

      // найдём минимальные значения уровней для нормализации графиков
      const rssi = signals.map(signal => signal.rssi);
      const ts = signals.map(signal => signal.ts);
      // rssi имеет отрицательные значения
      // нас интересует самое маленькой по абсолютному значению
      // для отрицательных чисел это МАКСИМАЛЬНОЕ
      const rssiMin = Math.max.apply(null, rssi) * 0.9; // считаем за минимум значние на 10% меньше минимального
      const tsMin = Math.min.apply(null, ts);
      console.log(`RSSI MIN = ${rssiMin}`);
      console.log(`TS MIN = ${tsMin}`);

      signals.forEach(signal => {
        this.rssi.push(Math.abs(signal.rssi - rssiMin));
        this.ts.push(signal.ts - tsMin);
      });

      this.showModal();
    }
  },

  mounted() {
    this.isBusy = true;
    axios.get(WORK_API).then(response => {
      this.isBusy = false;
      this.userInfo = Object.values(response.data);
    });
  }
};
</script>