<template>
  <div class="table">
    <b-table
      ref="selectableTable"
      selectable
      :select-mode="selectMode"
      :items="users"
      :fields="fields"
      @row-selected="onRowSelected"
      responsive="sm"
    >
      <!-- Example scoped slot for select state illustrative purposes -->
      <template v-slot:cell(selected)="{ rowSelected }">
        <template v-if="rowSelected">
          <span aria-hidden="true">&check;</span>
          <span class="sr-only">Selected</span>
        </template>
        <template v-else>
          <span aria-hidden="true">&nbsp;</span>
          <span class="sr-only">Not selected</span>
        </template>
      </template>
    </b-table>

    <div class="api-info">{{ userInfo }}</div>
  </div>
</template>

<script>
import axios from 'axios';

//const API_TEST = 'https://api.coindesk.com/v1/bpi/currentprice.json';
const API_TEST = 'https://rssi.wmrk.tk/';

const WORK_API = API_TEST;

export default {
  name: "Table",
  components: {},
  data: function() {
    return {
      fields: [
        {
          key: "id",
          sortable: true
        },
        {
          key: "fio",
          label: "ФИО",
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
      users: [
        {
          id: 0,
          fio: "FIO",
          mac: "23-df-34-45-56",
          phone: "+7-123-456-78-90"
        },
        {
          id: 1,
          fio: "FIO-123",
          mac: "23-df-34-45-56",
          phone: "+7-123-456-78-90"
        },
        {
          id: 2,
          fio: "FIO-3453",
          mac: "23-df-34-45-56",
          phone: "+7-123-456-78-90"
        }
      ],
      selectMode: "single",
      selected: null,

      userInfo: null
    };
  },
  methods: {
    onRowSelected(items) {
      this.selected = items;
    }
  },

  mounted() {
    axios
      .get(WORK_API)
      .then(response => (this.userInfo = response));
  }
};
</script>