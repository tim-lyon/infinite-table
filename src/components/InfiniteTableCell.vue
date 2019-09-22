<template>
  <div class="cell"
  :style="cellStyle"
  @mousedown="$emit('cellMouseDown')"
  @mouseover="$emit('cellMouseOver')">
    {{value}}
  </div>
</template>

<script>

export default {
  name: 'InfiniteTableCell',
  props: [
    'config',
    'value',
    'selectedRange',
    'isActive',
    'rowIndex',
    'columnIndex'
  ],
  computed: {
    cellStyle(){
      return {
        background: this.isSelected && !this.isActive ? this.config.style.selection.fill.color : 'none',
        borderStyle: 'solid',
        borderColor: this.config.style.selection.border.color,
        borderWidth: this.borderWidths,
        padding: this.paddingWidths
      }
    },
    borderWidths(){
      return (this.isTop ? '2px' : '0') +
      (this.isRight ? ' 2px' : ' 0') +
      (this.isBottom ? ' 2px' : ' 0') +
      (this.isLeft ? ' 2px' : ' 0')
    },
    paddingWidths(){
      return (this.isTop ? '0' : '2px') +
      (this.isRight ? ' 0' : ' 2px') +
      (this.isBottom ? ' 0' : ' 2px') +
      (this.isLeft ? ' 0' : ' 2px')
    },
    isLeft(){
      return this.isSelected &&
      this.columnIndex === this.selectedRange.start.C;
    },
    isRight(){
      return this.isSelected &&
      this.columnIndex === this.selectedRange.end.C;
    },
    isTop(){
      return this.isSelected &&
      this.rowIndex === this.selectedRange.start.R;
    },
    isBottom(){
      return this.isSelected &&
      this.rowIndex === this.selectedRange.end.R;
    },
    isSelected(){
      return this.rowIndex >= this.selectedRange.start.R &&
      this.rowIndex <= this.selectedRange.end.R &&
      this.columnIndex >= this.selectedRange.start.C &&
      this.columnIndex <= this.selectedRange.end.C
    }
  }
}
</script>

<style scoped>
.cell{
  width:6em;
  box-sizing: border-box;
}

::selection {
  background: none;
}
</style>
