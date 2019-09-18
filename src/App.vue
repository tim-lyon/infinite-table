<template>
  <div id="app">
    This is a table!
    <infinite-table
      :headers="headers"
      :table-data="sampleData"
      @input="inputRange"
      @insert="insertRange"
      @delete="deleteRange"
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
      sampleData: [
        ["a","b","c"],
        ["a","b","c"],
        ["a","b",false],
        ["a","b","c"],
        ["a","b","c"],
        ["a","b","c"]
      ],
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
    insertRange(payload) {
      console.log('insertRange')
      console.log(payload)
    },
    deleteRange(payload) {
      console.log('deleteRange')
      console.log(payload)
    }
  }
}
</script>

<style>
#app{
  font-family:calibri
}
</style>
