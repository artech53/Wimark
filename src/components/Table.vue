<template>
  <div class="table">
    <b-table
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
        </div>
        <div class="text-center text-danger my-2">
          <strong>Загружаю данные...</strong>
        </div>
      </template>
    </b-table>

    <b-modal ref="graph-modal" hide-footer>
      <template
        v-slot:modal-title
      >{{ selected && selected[0].first_name }} {{ selected && selected[0].last_name }} WiFi</template>

      <b-container fluid>
        <b-row class="text-center" align-v="center">
          <b-col cols="1">
            <div>RSSI</div>
          </b-col>
          <b-col class="mt-3" ref="graph-container">
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
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";
import BarGraph from "vue-svg-charts/src/components/bar";

const API_URL = "https://rssi.wmrk.tk/";

// используемый сервер API не поддеривает кросдоменные запросы
// использование сторонних компонентов не принесло результата
// потому используем прослойку в виде сервиса от хероку
const WORK_API = `https://cors-anywhere.herokuapp.com/${API_URL}`;

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

      // параметры таблицы
      selectMode: "single",
      selected: [],

      // данные характеристик связи пользователя
      rssi: [],
      ts: [],

      // общий массив данных пользователя
      userInfo: [],
      isBusy: false,

      // параметры графиков вывода информации о связи
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
      if (items[0].signals) {
        const signals = items[0].signals;

        // найдём минимальные значения уровней для нормализации графиков
        const rssiBuffer = signals.map(signal => signal.rssi);
        const tsBuffer = signals.map(signal => signal.ts);
        // rssi имеет отрицательные значения
        // нас интересует самое маленькой по абсолютному значению
        // для отрицательных чисел это МАКСИМАЛЬНОЕ
        const rssiMin = (Math.max.apply(null, rssiBuffer) * 0.9).toFixed(0); // считаем за минимум значние на 10% меньше минимального
        const tsMin = (Math.min.apply(null, tsBuffer) * 0.999999).toFixed(0); // уменьшим минимальное на 0,001%, чтобы не допустить 0 значений в графике
        // создаём массивы данных сигнала для вывода в графике
        // одновременно производим нормализацию значений для лучшей визуализации на графиках
        signals.forEach(signal => {
          this.rssi.push(Math.abs(signal.rssi - rssiMin));
          this.ts.push(signal.ts - tsMin);
        });

        this.showModal();
      }
      else {
        console.log(`Parsing data ERROR: onRowSelected(item) => signals not find in item data`);
      }
    }
  },

  async mounted() {
    this.isBusy = true;

    try {
      const apiData = (await axios.get(WORK_API)).data;
      this.isBusy = false;
      this.userInfo = apiData && Object.values(apiData);
    } catch (error) {
      console.log(`mounted() => ${error}`);
    }
  }
};
</script>