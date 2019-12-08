<template>
  <table :style="{width: totalWidth + 'px'}">
    <tr v-for="row of rowCount" :key="row">
      <td
        v-for="(cell, index) of headerCells[row-1]"
        :key="index"
        :colspan="cell.colspan"
        :rowspan="cell.rowspan"
        :class="{selected: isColumnSelected(cell), active: isColumnActive(cell)}"
        :style="{position:'relative',width: columnWidths[cell.column] + 'px',minWidth: columnWidths[cell.column] + 'px',maxWidth: columnWidths[cell.column] + 'px'}"
        @mousedown.stop="onColumnMouseDown({start: cell.column, end:cell.column + cell.colspan-1})"
        @mouseover.stop="onColumnMouseOver({start: cell.column, end:cell.column + cell.colspan-1})"
      >
        {{cell.name}}
        <div v-if="isColumnActive(cell) && cell.isLastRow" class="header-overlay" />
        <div
          class="column-width-handle"
          @mousedown.stop="startChangeColumnWidth(cell.column+cell.colspan-1)"
        />
      </td>
    </tr>
  </table>
</template>

<script>
var mouseDownColumns;

export default {
  name: "InfiniteTableHeaders",
  props: {
    headers: Array,
    columnCount: Number,
    isTableActive: Boolean,
    selectedColumns: Object,
    allRowsSelected: Boolean
  },
  data() {
    return {
      adjustingColumnWidth: 0,
      adjustingColumn: -1,
      adjustStartX: 0,
      columnWidths: []
    };
  },
  mounted() {
    let columnWidths = [];
    columnWidths.length = this.columnCount;
    columnWidths.fill(100);
    this.columnWidths = columnWidths;
  },
  computed: {
    totalWidth() {
      return this.columnWidths.reduce((a, b) => a + b, 0);
    },
    headerCells() {
      var headers = {};
      this.getHeaderRow(this.headers, 0, 0, headers);
      const rowCount = Object.keys(headers).length;
      let columnDetails = new Map();
      for (var row = 0; row < rowCount; row++) {
        headers[row].forEach(cell => {
          cell.rowspan = 1 + rowCount - cell.depth - row;
          cell.rowspan = cell.depth == 1 ? rowCount - row : 1;
          cell.isLastRow = row + cell.rowspan === rowCount;
          if (cell.isLastRow) {
            cell.width = this.columnWidths[cell.column];
            columnDetails.set(cell.column - 1, cell);
          }
        });
      }
      const sortedMap = new Map([...columnDetails.entries()].sort());
      this.$emit("columnDetails", [...sortedMap.values()]);
      return headers;
    },
    rowCount() {
      return Object.keys(this.headerCells).length;
    }
  },
  methods: {
    getHeaderRow(header, row, column, result) {
      header.forEach(h => {
        if (!result.hasOwnProperty(row)) {
          result[row] = [];
        }
        const colspan = this.numberOfColumns(h);
        let cell = {
          column: column,
          colspan: colspan,
          depth: this.depthOf(h)
        };
        Object.assign(cell, h);
        result[row].push(cell);

        if (h.hasOwnProperty("children")) {
          this.getHeaderRow(h.children, row + 1, column, result);
        }
        column += colspan;
      });
    },
    depthOf(header) {
      let depth = 1;
      if (header.hasOwnProperty("children")) {
        depth += header.children.reduce((a, b) =>
          Math.max(this.depthOf(a), this.depthOf(b))
        );
      }
      return depth;
    },
    selectColumnRange(range) {
      this.$emit("selectColumnRange", range);
    },
    onColumnMouseDown(range) {
      mouseDownColumns = range;
      this.selectColumnRange(range);
    },
    onColumnMouseOver(range) {
      if (event.buttons != 1 || !mouseDownColumns) return;
      if (range.start > mouseDownColumns.start) {
        range.start = mouseDownColumns.start;
      }
      if (range.end < mouseDownColumns.end) {
        range.end = mouseDownColumns.end;
      }
      this.$emit("selectColumnRange", range);
    },
    onMouseUp() {
      mouseDownColumns = false;
    },
    numberOfColumns(header) {
      if (header.hasOwnProperty("children")) {
        return header.children.reduce(
          (count, h) => count + this.numberOfColumns(h),
          0
        );
      }
      return 1;
    },
    isColumnActive(cell) {
      if (!this.isTableActive || !this.selectedColumns) return false;
      return (
        cell.column >= this.selectedColumns.start &&
        cell.column + cell.colspan - 1 <= this.selectedColumns.end
      );
    },
    isColumnSelected(cell) {
      if (!this.isTableActive || !this.allRowsSelected || !this.selectedColumns)
        return false;
      return (
        cell.column >= this.selectedColumns.start &&
        cell.column + cell.colspan - 1 <= this.selectedColumns.end
      );
    },
    startChangeColumnWidth(column) {
      this.adjustingColumn = column;
      this.adjustingColumnWidth = this.columnWidths[column];
      this.adjustStartX = event.clientX;
      window.addEventListener("mousemove", this.changeColumnWidth);
      window.addEventListener("mouseup", this.endChangeColumnWidth);
      this.$emit("startAdjustingColumnWidths");
    },
    changeColumnWidth(event) {
      event.stopPropagation();
      let newWidth =
        this.adjustingColumnWidth + event.clientX - this.adjustStartX;
      this.$set(
        this.columnWidths,
        this.adjustingColumn,
        Math.max(newWidth, 20)
      );
    },
    endChangeColumnWidth() {
      window.removeEventListener("mousemove", this.changeColumnWidth);
      window.removeEventListener("mouseup", this.endChangeColumnWidth);
      this.$emit("endAdjustingColumnWidths");
    }
  }
};
</script>

<style scoped>
.header-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 0.5em;
  background-image: linear-gradient(
    rgba(0, 135, 189, 0),
    rgba(68, 175, 218, 0.2)
  );
  z-index: 1000;
}
.selected {
  background: rgba(0, 135, 189, 0.1);
}
table {
  border-spacing: 0;
  border-collapse: collapse;
}

td {
  box-sizing: border-box;
  color: rgba(0, 0, 0, 0.54);
  border: 1px solid #e0e0e0;
  border-bottom: none;
  text-align: center;
  line-height: 1.3em;
  padding: 0;
  cursor: cell;
}
::selection {
  background: none;
}
.column-width-handle {
  position: absolute;
  right: -0.5em;
  top: -1px;
  bottom: -1px;
  width: 1em;
  background: none;
  cursor: ew-resize;
  z-index: 1001;
}
</style>
