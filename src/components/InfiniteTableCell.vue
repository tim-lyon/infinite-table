<template>
  <div
    :class="cellClass"
    @mousedown="$emit('mousedown', $event)"
    @mouseover="$emit('mouseover', $event)"
    @dblclick="$emit('dblclick')"
  >
    <slot />
  </div>
</template>

<script>
export default {
  name: "InfiniteTableCell",
  props: [
    "isTableActive",
    "selectedRange",
    "activeCell",
    "rowIndex",
    "columnIndex"
  ],
  computed: {
    isActive() {
      return (
        this.rowIndex == this.activeCell.R &&
        this.columnIndex == this.activeCell.C
      );
    },
    isInSelectedRowRange() {
      return (
        this.rowIndex >= this.selectedRange.start.R &&
        this.rowIndex <= this.selectedRange.end.R
      );
    },
    isInSelectedColumnRange() {
      return (
        this.columnIndex >= this.selectedRange.start.C &&
        this.columnIndex <= this.selectedRange.end.C
      );
    },
    isSelected() {
      return this.isInSelectedRowRange && this.isInSelectedColumnRange;
    },
    isLeft() {
      return (
        this.isInSelectedRowRange &&
        this.columnIndex === this.selectedRange.start.C
      );
    },
    isRight() {
      return (
        this.isInSelectedRowRange &&
        this.columnIndex === this.selectedRange.end.C
      );
    },
    isTop() {
      return (
        this.isInSelectedColumnRange &&
        this.rowIndex === this.selectedRange.start.R
      );
    },
    isBottom() {
      return (
        this.isInSelectedColumnRange &&
        this.rowIndex === this.selectedRange.end.R
      );
    },
    cellClass() {
      return {
        cell: true,
        highlight: this.isTableActive && this.isSelected,
        active: this.isTableActive && this.isActive
      };
    }
  }
};
</script>

<style>
.cell {
  box-sizing: border-box;
  background: none;
  overflow: hidden;
  white-space: nowrap;
  position: relative;
  color: rgba(0, 0, 0, 0.87);
  border-left: 1px solid #e0e0e0;
  border-bottom: 1px solid #e0e0e0;
}

.cell.highlight {
  background: rgba(0, 135, 189, 0.1);
}
.cell.active {
  background: none;
}
</style>
