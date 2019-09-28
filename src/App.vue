<template>
  <div id="app">
    This is a table!
    <infinite-table
      :headers="headers"
      :row-count="10000"
      :column-count="4"
      :tableData="{rows: 10000, columns: 4, get: cellValue, set: setCellValue}"
    />

    <infinite-table
      :headers="headers"
      :row-count="10000"
      :column-count="4"
      v-model="sampleData"
    />
  </div>
</template>

<script>
import InfiniteTable from './components/InfiniteTable.vue'

var d = {}
for(var i = 0; i < 10000; ++i){
  d[i] = [i,"a","b","c"]
}

export default {
  name: 'app',
  components: {
    InfiniteTable
  },
  data() {
    return{
      sampleData: d,
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
  },
  methods: {
    inputRange(payload) {
      console.log('inputRange')
      console.log(payload)
    },
    cellValue(row, column){
      return d[row][column]
    },
    setCellValue(row, column, value){
      d[row][column] = value
    }
  }
}
</script>

<style>
#app{
  font-family:calibri
}
</style>
