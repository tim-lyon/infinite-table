<template>
  <div id="app">
    This is an editable table with 1 million rows
    <infinite-table
      :headers="headers"
      :row-count="1e6"
      :column-count="7"
      :tableData="{get: getCellValue, set: setRangeValue}"
    />
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
        ]}
      ],
      setRanges: []
    }
  },
  methods: {
    inputRange(payload) {
      console.log('inputRange')
      console.log(payload)
    },
    getCellValue(row, column){
      let value = 'R'+row+'C'+column
      for(var set of this.setRanges){
        if(row >= set.range.start.R && row <= set.range.end.R &&
          column >= set.range.start.C && column <= set.range.end.C) {
          value = set.value
        }
      }
      return value
    },
    setRangeValue(range, value) {
      this.setRanges.push({
        range,
        value
      })
    }
  }
}
</script>

<style>
#app{
  font-family:calibri
}
</style>
