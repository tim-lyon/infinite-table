<template>
  <div id="app">
    This is an editable table with 1 million rows
    <infinite-table
      ref="table"
      :headers="headers"
      :row-count="1e6"
      :column-count="7"
      :get="getCellValue"
      @set="setRangeValue"
      @cut="cut"
      @copy="copy"
      @paste="paste"
      @undo="undo"
      @redo="redo"
      @buttonClick="buttonClick"
    />
    <textarea ref="copyText"/>
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
            {name: "area", type: "boolean"},
            {name: "zip", type: "number"}
          ]},
        {name: "age", type:"button", unit:"years"},
        {name: "address", children: [
          {name: "house", type:"string", disabled: true},
          {name: "street", type:"boolean"},
          {name: "city", type:"string", children: [
            {name: "area", options: ['One', 'Two', 'Three']},
            {name: "zip"}
          ]}
        ]}
      ],
      buttonClick: cell => alert('Buttton clicked! row: ' + cell.row + ' column: ' + cell.column),
      setBuffer: [],
      undoBuffer: [],
      contentToCopy: ''
    }
  },
  methods: {
    inputRange(payload) {
      console.log('inputRange')
      console.log(payload)
    },
    getCellValue(row, column){
      let value = {
        value: 'R'+row+'C'+column
      }
      if(row == 6){
        value.value = true
        value.type = 'boolean'
        value.disabled = true
      }
      if(row == 5 && column == 1){
        value.options = [
          {name: 'Car', value: 2},
          'Horse',
          'Tree',
          'Badger'
        ]
      }
      for(var set of this.setBuffer){
        if(!value.disabled && row >= set.range.start.R && row <= set.range.end.R &&
          column >= set.range.start.C && column <= set.range.end.C) {
          value.value = set.value
        }
      }
      return value
    },
    setRangeValue(payload) {
      this.setBuffer.push(payload)
      this.undoBuffer.length = 0
    },
    cut(range){
      this.copy(range)
      this.setRangeValue({range, value:null})
    },
    copyDataToClipboard(event) {
      event.preventDefault()
      event.clipboardData.setData("text/plain", this.contentToCopy)
    },
    copy(range){
      let rows = []
      for(var row = range.start.R; row <= range.end.R; row++){
        let values = []
        for(var column = range.start.C; column <= range.end.C; column++){
          const value = this.getCellValue(row, column)
          values.push(typeof value === 'object' ? value.value : value)
        }
        rows.push(values.join("\t"))
      }
      this.contentToCopy = rows.join("\r\n")
      document.addEventListener("copy", this.copyDataToClipboard);
      try {
        document.execCommand("copy")
      }
      catch(exception){
        alert(exception)
      }finally {
        document.removeEventListener("copy", this.copyDataToClipboard);
      }
    },
    paste(range){
      navigator.clipboard.readText().then(text => {
        text.split('\r\n').forEach((row, iRow) => {
          row.split('\t').forEach((value, iColumn) => {
            const cell = {
              R: range.start.R + iRow,
              C: range.start.C + iColumn
            }
            this.setRangeValue({
              range: {
                start: cell,
                end: cell
              },
              value
            })
          })
        })
      })
    },
    undo(){
      const setCount = this.setBuffer.length
      if(setCount > 0){
        this.undoBuffer.push(this.setBuffer[setCount - 1])
        this.setBuffer.pop()
      }
    },
    redo(){
      const undoCount = this.undoBuffer.length
      if(undoCount > 0){
        this.setBuffer.push(this.undoBuffer[undoCount - 1])
        this.undoBuffer.pop()
      }
    }
  }
}
</script>

<style>
#app{
  font-family:calibri
}
textarea {
position:absolute;
top:-10000px;
}
</style>
