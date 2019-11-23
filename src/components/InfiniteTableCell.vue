<template>
  <td
    :class="cellClass"
    @mousedown="$emit('mousedown', $event)"
    @mouseover="$emit('mouseover', $event)"
    @dblclick="$emit('dblclick')"
  >
    <slot />
    <div :class="overlayClass" v-if="showOverlay"></div>
  </td>
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
    showOverlay() {
      return (
        this.isTableActive &&
        (this.isTop || this.isBottom || this.isLeft || this.isRight)
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
    },
    overlayClass() {
      return {
        overlay: true,
        top: this.isTop,
        right: this.isRight,
        bottom: this.isBottom,
        left: this.isLeft
      };
    }
  }
};
</script>

<style>
.cell {
  box-sizing: border-box;
  background: none;
  height: 30px;
  overflow: hidden;
  white-space: nowrap;
  position: relative;
  color: rgba(0, 0, 0, 0.87);
}

.cell.highlight {
  background: rgba(0, 135, 189, 0.1);
  border: 1px solid rgb(0, 135, 189);
}
.cell.active {
  background: none;
}

.overlay {
  position: absolute;
  top: 0px;
  left: 0px;
  right: 0px;
  bottom: 0px;
  border: 0px solid rgb(0, 135, 189);
  pointer-events: none;
}
.overlay.top {
  border-top-width: 1px;
}
.overlay.left {
  border-left-width: 1px;
}
.overlay.right {
  border-right-width: 1px;
}
.overlay.bottom {
  border-bottom-width: 1px;
}
</style>
