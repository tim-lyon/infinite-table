<template>
  <div id="app">
    This is a table!
    <infinite-table :headers="headers" exports :data-interface="dataInterface" />
    <infinite-table headers v-model="sampleData" />
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
        ["a","b","c"],
        ["a","b","c"],
        ["a","b","c"],
        ["a","b","c"]
      ],
      dataInterface: {
        count: 500,
        get(row) {
          return new Promise(function(resolve, reject) {
            setTimeout(() => {
              resolve(["R" + row, "a\"b", "\"b\"", "c,ello"])
            }, 1000);
            
          })
        },
        set(row, column, value) {
          alert("set R" + row + "C" + column + " to " + value)
        }
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
