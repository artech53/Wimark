<template>
  <div class="table">
    <b-table
      ref="data-table"
      selectable
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

    <b-modal ref="graph-modal" hide-footer @hidden="resetModal">
      <template
        v-slot:modal-title
      >{{ selected && selected[0].first_name }} {{ selected && selected[0].last_name }} WiFi</template>

      <apexchart type="line" height="350" :options="chartOptions" :series="series"></apexchart>
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";
import VueApexCharts from "vue-apexcharts";

const API_URL = "https://rssi.wmrk.tk/";

// используемый сервер API не поддеривает кросдоменные запросы
// использование сторонних компонентов не принесло результата
// потому используем прослойку в виде сервиса от хероку
const WORK_API = `https://cors-anywhere.herokuapp.com/${API_URL}`;

export default {
  name: "Table",
  components: { apexchart: VueApexCharts },
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

      // выделенная строка таблицы
      selected: [],

      // параметры графика вывода данных
      series: [
        {
          name: "RSSI",
          data: []
        }
      ],
      chartOptions: {
        chart: {
          height: 350,
          type: "line",
          zoom: {
            enabled: true
          }
        },
        xaxis: {
          type: "numeric",
          labels: {
            formatter: function(value) {
              // переводим время в универсальное для подписи строки координат
              const date = new Date(value);

              const hours = date.getUTCHours();
              const minutes = date.getUTCMinutes();
              const seconds = date.getUTCSeconds();
              const milliseconds = date.getUTCMilliseconds();

              return `${hours}:${minutes}:${seconds}'${milliseconds}`; // The formatter function overrides format property
            }
          }
        },
        tooltip: {
          x: {
            formatter: function(val) {
              return new Date(val);
            }
          }
        }
      },

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
    resetModal() {
      this.$refs["data-table"].clearSelected();
    },
    onRowSelected(items) {
      this.selected = items;

      // получаем данные сигнала выбранного элемента
      if (items.length > 0 && items[0].signals) {
        const signals = items[0].signals;
        this.series[0].data = [];
        // отсортируем данные сигнала по временной шкале
        signals.sort((a, b) => {
          if (a.ts > b.ts) return 1;
          if (a.ts == b.ts) return 0;
          if (a.ts < b.ts) return -1;
        });
        // создаём массивы данных сигнала для вывода в графике
        // одновременно производим нормализацию значений для лучшей визуализации на графиках
        signals.forEach(signal => {
          this.series[0].data.push([signal.ts, signal.rssi]);
          //this.chartOptions.xaxis.categories.push(new Date(signal.ts));
        });

        this.showModal();
      } else {
        console.log(
          `Parsing data ERROR: onRowSelected(item) => signals not find in item data`
        );
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