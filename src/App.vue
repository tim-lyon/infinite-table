<template>
  <div id="app">
    This is a table!
    <infinite-table :headers="headers" :rowCount="1000000" :dataGetter="getRowData" />
  </div>
</template>

<script>
import InfiniteTable from './components/InfiniteTable.vue'

export default {
  name: 'app',
  components: {
    InfiniteTable
  },
  data() {
    return{
      getRowData(i){
      return [i,'a','b','c']
    },
      headers: [
        {name: "name", type:"string", children: [
            {name: "area"},
            {name: "zip"}
          ]},
        {name: "age", type:"integer", unit:"years"},
        {name: "address", children: [
          {name: "house", type:"string"},
          {name: "street", type:"string"},
          {name: "city", type:"string", children: [
            {name: "area"},
            {name: "zip"}
          ]}
        ]},
        {name: "hair", type:"string"}
      ]
    }
  }
}
</script>

<style>
#app{
  font-family:calibri
}
</style>
