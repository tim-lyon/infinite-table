<template>
  <div class="header-container" @mouseup="onMouseUp">
    <template v-for="(h, i) of headers">
      <div class="inner-container"
        :style="headerStyle(i)"
        v-if="h.hasOwnProperty('children')"
        @mousedown.stop="onColumnMouseDown(columnRange(i))"
        @mouseover.stop="onColumnMouseOver(columnRange(i))"
        :key=i
      >
        <div>{{h.name}}</div>
        <InfiniteTableHeaders
        :config="config"
        :headers="h.children"
        :startColumn="columnRange(i).start"
        :selectedColumns="selectedColumns"
        @selectColumnRange="selectColumnRange"
        @mouseOverColumns="onColumnMouseOver"/>
        
      </div>
      <div v-else class="header-item"
        :style="headerStyle(i)"
        @mousedown.stop="onColumnMouseDown(columnRange(i))"
        @mouseover.stop="onColumnMouseOver(columnRange(i))"
        :key="-i"
      >
        <div>
          {{h.name}}
          <span v-if="h.hasOwnProperty('unit')">({{h.unit}})</span>
        </div>
      </div>
      
    </template>
  </div>
</template>

<script>

var mouseDownColumns

export default {
  name: 'InfiniteTableHeaders',
  props: {
    headers: Array,
    startColumn: {
      type: Number,
      default: 1
    },
    selectedColumns: Object,
    config: Object,
  },
  data() {
    return {
      
    }
  },
  methods: {
    selectColumnRange(range){
      this.$emit('selectColumnRange', range)
    },
    onColumnMouseDown(range){
      mouseDownColumns = range
      this.selectColumnRange(range)
    },
    onColumnMouseOver(range){
      if(event.buttons != 1 || !mouseDownColumns) return
      if(range.start > mouseDownColumns.start){
        range.start = mouseDownColumns.start
      }
      if(range.end < mouseDownColumns.end){
        range.end = mouseDownColumns.end
      }
      this.$emit('selectColumnRange', range)
    },
    onMouseUp(){
      mouseDownColumns = false
    },
    columnRange(i){
      let iNow = 0
      let count = 0
      let childCount = 0
      for(var header of this.headers){
        if(i==iNow){
          childCount = this.numberOfColumns(header)
          break
        }
        ++iNow
        count += this.numberOfColumns(header)
      }
      return {
        start: count+this.startColumn,
        end: count+this.startColumn+childCount-1
      }
    },
    numberOfColumns(header){
      if(header.hasOwnProperty('children')){
        return header.children.reduce((count, h) => count + this.numberOfColumns(h), 0);
      }
      return 1
    },
    isColumnSelected(i){
      if(!this.selectedColumns) return false
      var range = this.columnRange(i)
      return range.start-1 >= this.selectedColumns.start &&
      range.end-1 <= this.selectedColumns.end
    },
    headerStyle(i){
      const isSelected = this.isColumnSelected(i)
      return {
          background: isSelected ? this.config.style.selection.border.color : 'none',
          borderColor: isSelected ? this.config.style.selection.border.color : '#aaa'
      }
    }
  }
}
</script>

<style scoped>
.header-container{
  display:flex;
  align-items: stretch;
  text-align: center;
  cursor:cell;
  color:rgba(0,0,0,0.54);
  user-select: none;
}

.inner-container{
  display:flex;
  flex-direction: column;
  border-top: 1px solid #aaa;
}
.inner-container .header-container{
  flex:1;
}
.header-item{
  min-width:6em;
  display: flex;
  align-items: center;
  border-bottom: 1px solid #aaa;
  border-top: 1px solid #aaa;
}
.header-item>div{
  flex:1;
}
</style>
