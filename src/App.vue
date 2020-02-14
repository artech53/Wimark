<template>
  <div id="app">
    
    <Table />
    <!--<HelloWorld msg="Welcome to Your 123 Vue.js App"/>-->
  </div>
</template>

<script>
import Table from './components/Table.vue';

export default {
  name: 'App',
  components: {
    Table,
    //HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
