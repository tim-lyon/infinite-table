<template>
  <div class="table-container" tabindex="1" @keydown="OnTableKeypress">
    <InfiniteTableHeaders
      :headers="headers_"
      :selectedColumns="selectedColumns"
      @selectColumnRange="selectColumns"
    />
    <div class="table-body" ref="body" :style="{height:contentHeight+'px'}" @scroll="scrollTable">
      <div :style="{height: totalHeight+'px'}">
        <div :style="{position:'relative', top:topBufferHeight+'px'}">
          <InfiniteTableRow
          v-for="i in renderedRowCount"
          :index="i+startRow"
          :style="{background: (i+startRow)%2 ? 'rgba(0,0,0,0.05)' : ''}"
          @input="setData($event, i+startRow)"
          @cellMouseDown="onCellMouseDown"
          @cellMouseOver="onCellMouseOver"
          @rowMouseDown="onRowMouseDown(i+startRow)"
          @rowMouseOver="onRowMouseOver(i+startRow)"
          @created="getRowData(i+startRow)"
          :data="isDataInterface ? rowData[i+startRow] ? rowData[i+startRow] : ['...'] : getRowData(i+startRow)" 
          :key="i+startRow" />
          <div :style="underlayStyle"></div>
        </div>
      </div>
      
    </div>
    <div>
      <button v-for="(exportFunction, name) in computedExports" :key="name" @click="startExport(exportFunction)">
        Export {{name}}
      </button>
    </div>

    <br><br>Selection: {{selection}}
    <br>
  </div>

</template>

<script>
import InfiniteTableHeaders from './InfiniteTableHeaders.vue'
import InfiniteTableRow from './InfiniteTableRow.vue'

const ROW_HEIGHT = 30
const BUFFER_ROWS = 5
const ROW_COUNT = 12
const MAX_ROWS = 100000
export default {
  name: 'InfiniteTable',
  components: {
    InfiniteTableHeaders,
    InfiniteTableRow
  },
  props: {
    headers: {
      required: false
    },
    dataInterface: {
      type: Object,
      required: false
    },
    value: {
      type: Array,
      required: false
    },
    exports: {
      required: false
    }
  },
  mounted(){
    this.computeHeaders()
  },
  methods: {
    getRowData(iRow) {
      if(!this.isDataInterface){
        return this.value[iRow-1]
      }
      return new Promise(resolve => {
        this.dataInterface.get(iRow).then(rowData=>{
          this.$set(this.rowData, iRow, rowData)
          resolve()
        })
      })
    },
    computeHeaders() {
      if(Array.isArray(this.headers) || this.headers === undefined){
        this.headers_ = this.headers
      }
      else if(!this.isDataInterface) {
        this.headers_ = this.value[0].map((cell, iCol) => {
          return {name: this.defaultColumnName(iCol+1)}
        })
      }
      else{
        this.dataInterface.get(1).then(row=>{
          var data = row.map((cell, iCol) => {
            return {name: this.defaultColumnName(iCol+1)}
          })
          this.headers_ = data
        })
      }
    },
    startExport(exportFunction){
      this.getAllData().then(()=>{
        exportFunction()
      })
    },
    getAllData(){
      var promises = []
      for(var iRow = 1; iRow <= this.dataInterface.count; iRow++){
        promises.push(this.getRowData(iRow))
      }
      return Promise.all(promises)
    },
    exportCSV(){
      let csvFile = "data:text/csv;charset=utf-8,";
      for(var iRow = 1; iRow <= this.rowCount; iRow++){
        const rowArray = this.rowData[iRow].map(fieldValue => {
          var value = fieldValue.hasOwnProperty('value') ? fieldValue.value : fieldValue
          var string = value.toString().replace(/"/g, '""')
          if (string.search(/("|,|\n)/g) >= 0){
            string = '"' + string + '"'
          }
          return string
        })
        csvFile += rowArray.join(",") + "\r\n";
      }
      var blob = new Blob([csvFile], { type: 'text/csv;charset=utf-8;' });
      var url = URL.createObjectURL(blob);
      var link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", "export.csv");
      link.style.visibility = 'hidden';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    setData(data, row){
      if(this.isDataInterface){
        this.dataInterface.set(row, data.column, data.value)
      }
      else{
        this.$set(this.value[row-1], data.column, data.value)
        this.$emit('input', this.value)
      }
    },
    defaultColumnName(iCol) {
      for (var ret = '', a = 1, b = 26; (iCol -= a) >= 0; a = b, b *= 26) {
        ret = String.fromCharCode(parseInt((iCol % b) / a) + 65) + ret;
      }
      return ret;
    },
    OnTableKeypress(){
      switch(event.key){
        case 'ArrowDown':
          this.OnArrow(0,1)
          break
        case 'ArrowUp':
          this.OnArrow(0,-1)
          break
        case 'ArrowLeft':
          this.OnArrow(-1,0)
          break
        case 'ArrowRight':
          this.OnArrow(1,0)
          break
        case 'Tab':
          event.preventDefault()
          this.OnArrow(1,0)
          break
        case 'PageUp':
          this.OnArrow(0,-ROW_COUNT)
          break
        case 'PageDown':
          this.OnArrow(0,ROW_COUNT)
          break
      }
    },
    scrollToRow(iRow){
      const distanceFromReferenceRow = this.$refs.body.scrollTop-this.referenceRow.scrollTop
      const rowsFromReferenceRow = distanceFromReferenceRow/ROW_HEIGHT
      const firstFullyVisibleRow = Math.ceil(this.referenceRow.row+rowsFromReferenceRow)+1
      const offset = iRow < firstFullyVisibleRow ? 1 :
      iRow > firstFullyVisibleRow + ROW_COUNT - 2 ? ROW_COUNT : 0
      if(offset!=0){
        this.$refs.body.scrollTop = this.referenceRow.scrollTop +
                (iRow-this.referenceRow.row-offset)*ROW_HEIGHT
      }
    },
    OnArrow(dx, dy){
      if(event.shiftKey){
        this.selection.end.C += dx
        this.selection.end.R += dy
      }else{
        this.selection.start.C += dx
        this.selection.start.R += dy
        this.selection.end.C = this.selection.start.C
        this.selection.end.R = this.selection.start.R
      }
      this.clampSelection()
      this.scrollToRow(this.selection.end.R)
    },
    clampSelection(){
      if(this.selection.start.R < 1){
        this.selection.start.R = 1
      }
      else if(this.selection.start.R > this.rowCount){
        this.selection.start.R = this.rowCount
      }
      if(this.selection.end.R < 1){
        this.selection.end.R = 1
      }
      else if(this.selection.end.R > this.rowCount){
        this.selection.end.R = this.rowCount
      }
      if(this.selection.start.C < 0){
        this.selection.start.C = 0
      }
      if(this.selection.end.C < 0){
        this.selection.end.C = 0
      }
    },
    selectColumns(columns){
      if(!event.shiftKey){
        this.selection.start.C = columns.start - 1
      }
      this.selection.end.C = columns.end - 1
      this.selection.start.R = 1
      this.selection.end.R = this.rowCount
    },
    scrollTable(){
      const scrollTop = event.target.scrollTop
      if(this.rowCount > MAX_ROWS){
        const step = scrollTop - this.lastScrollTop
        if(Math.abs(step) > 0.01*MAX_ROWS){
          if(scrollTop < MAX_ROWS){
            //align with top
            this.referenceRow.row = 0
            this.referenceRow.scrollTop = 0
          }
          else if(event.target.scrollHeight-scrollTop < MAX_ROWS){
            // align with bottom
            this.referenceRow.row = this.rowCount
            this.referenceRow.scrollTop = event.target.scrollHeight
          }
          else{
            // align based on percentage
            this.referenceRow.row = Math.round(this.rowCount*scrollTop/event.target.scrollHeight)
            this.referenceRow.scrollTop = scrollTop
          }
        }

        const distanceFromReferenceRow = scrollTop - this.referenceRow.scrollTop
        const rowsFromReferenceRow = Math.round(distanceFromReferenceRow/30)
        const startRow = this.referenceRow.row + rowsFromReferenceRow
        this.startRow = Math.max(startRow - BUFFER_ROWS, 0)
        const lastRow = Math.min(this.startRow + ROW_COUNT+2*BUFFER_ROWS, this.rowCount)
        this.renderedRows = lastRow - this.startRow
        this.topBufferHeight = this.referenceRow.scrollTop + 30*rowsFromReferenceRow
        this.topBufferHeight -= 30*(startRow - this.startRow)
      }else{
        let firstVisibleRow = Math.round(scrollTop/30)
        this.startRow = Math.max(firstVisibleRow - BUFFER_ROWS, 0)
        const lastRow = Math.min(this.startRow + ROW_COUNT+2*BUFFER_ROWS, this.rowCount)
        this.renderedRows = lastRow - this.startRow
        this.topBufferHeight = this.startRow*ROW_HEIGHT
      }
      this.lastScrollTop = scrollTop
    },
    onCellMouseDown(iCol){
      this.selection.end.C = iCol
      if(!event.shiftKey){
        this.selection.start.C = iCol
      }
      window.getSelection().removeAllRanges();
    },
    onCellMouseOver(iCol){
      if(event.buttons == 1){
        this.selection.end.C = iCol
      }
    },
    onRowMouseDown(iRow){
      this.selection.end.R = iRow
      if(!event.shiftKey){
        this.selection.start.R = iRow
      }
    },
    onRowMouseOver(iRow){
      if(event.buttons == 1){
        this.selection.end.R = iRow
      }
    },
    rowTopOffset(iRow){
      let offset = 30*(iRow-1)
      if(this.rowCount > MAX_ROWS){
        offset += this.referenceRow.scrollTop - 30* this.referenceRow.row
      }
      return offset+'px'
    },
    columnLeftOffset(iCol){
      return 6*iCol+'em'
    }
  },
  data(){
    return {
      headers_: undefined,
      rowData: {},
      referenceRow: {row: 0, scrollTop: 0},
      lastScrollTop: 0,
      renderedRows: ROW_COUNT+2*BUFFER_ROWS,
      startRow: 0,
      topBufferHeight: 0,
      contentHeight: ROW_COUNT*ROW_HEIGHT,

      selection: {
        start: {
          R:0,
          C:0
        },
        end: {
          R:0,
          C:0
        }
      }
    }
  },
  computed: {
    isDataInterface(){
      return this.dataInterface !== undefined
    },
    computedExports(){
      return Array.isArray(this.exports) || this.exports === undefined ?
      this.exports : {CSV: this.exportCSV}
    },
    rowCount() {
      return this.isDataInterface ? this.dataInterface.count : this.value.length
    },
    renderedRowCount() {
      return Math.min(this.renderedRows, this.rowCount - this.startRow)
    },
    totalHeight() {
      return Math.min(MAX_ROWS, this.rowCount)*ROW_HEIGHT
    },
    selectedColumns(){
      if(Math.abs(this.selection.start.R -
      this.selection.end.R) == this.rowCount-1){
        return {
          start: Math.min(this.selection.start.C,this.selection.end.C),
          end: Math.max(this.selection.start.C,this.selection.end.C)
        }
      }
      return null
    },
    underlayStyle(){
      const lowestRow = Math.min(this.selection.start.R,this.selection.end.R)
      const highestRow = Math.max(this.selection.start.R,this.selection.end.R)
      let topOffset = 30*(lowestRow - this.startRow - 1)
      if(topOffset < 0){
        topOffset =0
      }
      if(topOffset>30*this.renderedRows){
        topOffset = 30*this.renderedRows
      }

      let bottomOffset = 30*(highestRow - this.startRow - 1)
      if(bottomOffset < 0){
        bottomOffset =0
      }
      if(bottomOffset>30*this.renderedRows){
        bottomOffset = 30*this.renderedRows
      }
      const height = 26+bottomOffset-topOffset
      
      return {
        position: 'absolute',
        top: topOffset+'px',
        height: height+'px',
        left: this.columnLeftOffset(Math.min(this.selection.start.C,this.selection.end.C)),
        background: 'rgb(240,240,255)',
        border: '0.1em solid rgb(180,180,255)',
        lineHeight: '30px',
        width: (6*(Math.abs(this.selection.start.C-this.selection.end.C)+1))-0.2+'em',
        zIndex: '-1'
      }
    }
  }
}
</script>

<style scoped>
.table-container{
  border:1px solid lime;
  padding-left:2em;
  outline:none;
}
.table-body{
  overflow-y: scroll;
  position: relative;
  cursor: cell;
}
</style>
