<template>
  <div class="component-container">
    <div
      class="table-container"
      ref="tableContainer"
      tabindex="1"
      @keydown.self="OnTableKeypress"
      @copy.prevent
      @mouseup="unselectNative"
      @focus="checkIsTableActive"
      @blur="checkIsTableActive"
    >
      <div v-if="computedHeaders !== false">
        <InfiniteTableHeaders
          style="margin-left:51px;"
          :headers="computedHeaders"
          :isTableActive="isTableActive"
          :selectedColumns="selectedColumns"
          :allRowsSelected="allRowsSelected"
          @selectColumnRange="selectColumns"
          @columnDetails="setColumnDetails"
        />
      </div>
      <div class="table-body-container">
        <div class="table-body" ref="body" @scroll="scrollTable">
          <div :style="tableBodyInnerStyle">
            <table :style="tableBodyVisibleStyle">
              <colgroup>
                <col v-for="i of columnCount+1" :key="i" />
              </colgroup>
              <tr v-for="row in renderedRowArray" :row-index="row" :key="row - startRow">
                <th
                  :class="tableRowHeaderStyle(row)"
                  v-if="headers !== false"
                  @mousedown="onCellMouseDown({R: row, C: -1}, $event)"
                  @mouseover="onCellMouseOver({R: row, C: -1}, $event)"
                >
                  {{row}}
                  <div
                    v-if="isTableActive && row >= selectedRows.start && row <= selectedRows.end"
                    class="row-number-overlay"
                  />
                </th>
                <td
                  is="InfiniteTableCell"
                  v-for="column in columnCount"
                  :key="column"
                  :is-table-active="isTableActive"
                  :selected-range="selectedRange"
                  :active-cell="active"
                  :row-index="row"
                  :column-index="column -1"
                  @mousedown="onCellMouseDown({R: row, C: column -1}, $event)"
                  @mouseover="onCellMouseOver({R: row, C: column -1}, $event)"
                  @dblclick="onCellDblClick"
                >
                  <div
                    v-for="(value, index) of [getCellValue(row, column -1)]"
                    :key="'v'+index"
                    :class="valueClass(value)"
                  >
                    <input
                      v-if="value.type === 'boolean'"
                      type="checkbox"
                      :disabled="value.disabled"
                      tabindex="-1"
                      @mousedown.stop
                      :checked="value.value"
                      @click="setCheckbox($event, row, column -1)"
                      @focus="checkIsTableActive"
                    />
                    <button
                      v-else-if="value.type === 'button'"
                      :disabled="value.disabled"
                      tabindex="-1"
                      @mousedown.stop
                      @click="clickButton(row, column-1)"
                      @focus="checkIsTableActive"
                    >{{displayString(value)}}</button>
                    <input
                      v-else-if="isEditing && active.R == row && active.C == column - 1"
                      :type="inputType(value)"
                      class="the-input"
                      v-model="inputValue"
                      ref="theinput"
                      @keydown="onInputKeypress"
                      @focus="checkIsTableActive"
                    />
                    <select
                      v-else-if="value.hasOptions && active.R == row && active.C == column - 1 && !value.disabled"
                      tabindex="-1"
                      @mousedown.stop
                      @change="selectValue"
                      @focus="checkIsTableActive"
                    >
                      <option
                        v-for="(option, index) of value.options"
                        :key="index"
                        :value="option.value"
                        :selected="option.value === value.value"
                      >{{option.name}}</option>
                      <option
                        v-if="!value.options.find(option => option.value == value.value)"
                        selected
                      >{{value.value}}</option>
                    </select>
                    <template v-else>{{displayString(value)}}</template>
                  </div>
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>

      <transition name="fade">
        <div class="table-busy" v-if="isBusy">
          <InfiniteTableLoader :message="isBusy" />
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import InfiniteTableHeaders from "./InfiniteTableHeaders.vue";
import InfiniteTableCell from "./InfiniteTableCell.vue";
import InfiniteTableLoader from "./InfiniteTableLoader.vue";

const ROW_HEIGHT = 30;
const BUFFER_ROWS = 5;
const ROW_COUNT = 12;
const MAX_ROWS = 100000;

const clamp = (value, min, max) =>
  value < min ? min : value > max ? max : value;

function CellReference() {
  this.R = -1;
  this.C = -1;
}

export default {
  name: "InfiniteTableCore",
  components: {
    InfiniteTableHeaders,
    InfiniteTableCell,
    InfiniteTableLoader
  },
  props: {
    columnCount: {
      type: Number,
      required: true
    },
    rowCount: {
      type: Number,
      required: true
    },
    headers: {
      required: false,
      default: true
    },
    selectable: {
      type: Boolean,
      required: false,
      default: true
    },
    editable: {
      type: Boolean,
      required: false,
      default: true
    },
    get: {
      type: Function,
      required: true
    }
  },
  data() {
    return {
      columnDetails: [],
      referenceRow: { row: 0, scrollTop: 0 },
      lastScrollTop: 0,
      renderedRows: ROW_COUNT + 2 * BUFFER_ROWS,
      startRow: 0,
      topBufferHeight: 0,
      isEditing: false,
      isDeepEditing: false,
      isSelectable: true,
      isEditable: true,
      inputValue: "",
      isEndKeyPressed: false,
      active: new CellReference(),
      selection: {
        start: new CellReference(),
        end: new CellReference()
      },
      isBusy: "",
      isTableActive: false
    };
  },
  created() {
    this.isEditable = this.editable;
    this.isSelectable = this.selectable || this.editable;
  },
  methods: {
    checkIsTableActive() {
      this.isTableActive = this.$el.contains(document.activeElement);
    },
    unselectNative() {
      if (document.activeElement == this.$refs.tableContainer) {
        window.getSelection().removeAllRanges();
      }
    },
    setColumnDetails(details) {
      this.columnDetails = details;
    },
    getCellValue(row, column) {
      let value = {};
      if (this.columnDetails[column]) {
        Object.assign(value, this.columnDetails[column]);
      }
      let v = this.get(row, column);
      if (typeof v === "object") {
        Object.assign(value, v);
      } else {
        value.value = v;
      }
      value.hasOptions = value.hasOwnProperty("options");
      if (value.hasOptions) {
        value.options = value.options.map(option => {
          const name = option.hasOwnProperty("name") ? option.name : option;
          const value = option.hasOwnProperty("value") ? option.value : name;
          return { name, value };
        });
      }
      if (!value.hasOwnProperty("type")) {
        value.type = typeof value.value;
      }
      if (value.type == "button" && !value.hasOwnProperty("onclick")) {
        value.onclick = new Function();
      }
      return value;
    },
    displayString(value) {
      let string = value.value;
      if (value.hasOptions) {
        let found = value.options.find(option => option.value == value.value);
        if (found) {
          string = found.name;
        }
      }
      return string;
    },
    valueClass(value) {
      return {
        centre: value.type == "boolean" || value.type == "button",
        disabled: value.disabled
      };
    },
    inputType(value) {
      switch (value.type) {
        case "number":
        case "integer":
          return "number";
      }
      return "text";
    },
    setRangeValue(range, value) {
      this.$emit("set", { range, value });
    },
    selectValue(event) {
      this.setRangeValue(
        this.cellRange(this.active.R, this.active.C),
        event.target.value
      );
      this.refocus();
    },
    setCheckbox(event, row, column) {
      this.setRangeValue(
        event.ctrlKey ? this.selectedRange : this.cellRange(row, column),
        event.target.checked
      );
      this.refocus();
    },
    clickButton(row, column) {
      this.$emit("buttonClick", { row, column });
      this.refocus();
    },
    cellRange(row, column) {
      return {
        start: {
          R: row,
          C: column
        },
        end: {
          R: row,
          C: column
        }
      };
    },
    defaultColumnName(iCol) {
      for (var ret = "", a = 1, b = 26; (iCol -= a) >= 0; a = b, b *= 26) {
        ret = String.fromCharCode(parseInt((iCol % b) / a) + 65) + ret;
      }
      return ret;
    },
    OnTableKeypress(event) {
      const rowShift = this.isEndKeyPressed ? this.rowCount : 1;
      const columnShift =
        this.isEndKeyPressed || event.key == "Home" ? this.columnCount : 1;

      // keyboard shortcuts
      if (event.ctrlKey) {
        switch (event.key.toLowerCase()) {
          case "a":
            event.preventDefault(); // stop selecting other page content
            this.selectAll();
            return;
          case "x":
            this.emitEvent("cut", this.selectedRange, "Cutting...");
            return;
          case "c":
            this.emitEvent("copy", this.selectedRange, "Copying...");
            return;
          case "v":
            this.emitEvent("paste", this.selectedRange, "Pasting...");
            return;
          case "y":
            this.emitEvent("redo", this.selectedRange, "Redoing...");
            return;
          case "z":
            this.emitEvent("undo", this.selectedRange, "Undoing...");
            return;
        }
      }

      switch (event.key) {
        case "Escape":
          this.endEdit();
          this.OnArrow(event, 0, 0);
          break;
        case "ArrowDown":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, 0, rowShift);
          break;
        case "ArrowUp":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, 0, -rowShift);
          break;
        case "ArrowLeft":
        case "Home":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, -columnShift, 0);
          break;
        case "ArrowRight":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, columnShift, 0);
          break;
        case "Tab":
          event.preventDefault(); // stop table losing focus
          this.OnTab(event);
          break;
        case "Enter":
          event.preventDefault(); // stop window scrolling
          this.OnEnter(event);
          break;
        case "PageUp":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, 0, -ROW_COUNT);
          break;
        case "PageDown":
          event.preventDefault(); // stop window scrolling
          this.OnArrow(event, 0, ROW_COUNT);
          break;
        case "Delete":
          if (this.isEditing) {
            return;
          }
          this.OnDelete();
          break;
        case "Backspace":
          if (this.isEditing) {
            return;
          }
          this.OnDelete();
          this.startEdit();
          break;
        case "F2":
          this.startEdit();
          break;
        default:
          if (!this.isEditing && event.key.length == 1) {
            this.startEdit(true);
          }
      }
      this.isEndKeyPressed = event.key == "End";
      if (this.isEndKeyPressed) {
        event.preventDefault(); // stop window scrolling
      }
    },
    emitEvent(name, payload, waitMessage) {
      this.isBusy = waitMessage;
      setTimeout(() => {
        this.$emit(name, payload);
        this.isBusy = false;
      }, 50);
    },
    onInputKeypress(event) {
      if (event.key === "Escape") {
        this.cancelEdit();
        this.OnArrow(0, 0);
      } else if (
        !this.isDeepEditing ||
        (event.key !== "ArrowLeft" && event.key !== "ArrowRight")
      ) {
        this.OnTableKeypress(event);
      }
    },
    scrollToRow(iRow) {
      const distanceFromReferenceRow =
        this.$refs.body.scrollTop - this.referenceRow.scrollTop;
      const rowsFromReferenceRow = distanceFromReferenceRow / ROW_HEIGHT;
      const firstFullyVisibleRow =
        Math.ceil(this.referenceRow.row + rowsFromReferenceRow) + 1;
      const offset =
        iRow < firstFullyVisibleRow
          ? 1
          : iRow > firstFullyVisibleRow + ROW_COUNT - 2
          ? ROW_COUNT
          : 0;
      if (offset != 0) {
        this.$refs.body.scrollTop =
          this.referenceRow.scrollTop +
          (iRow - this.referenceRow.row - offset) * ROW_HEIGHT;
      }
    },
    OnArrow(event, dColumn, dRow, ignoreShiftKey) {
      if (event.shiftKey && !ignoreShiftKey) {
        this.selection.end.C += dColumn;
        this.selection.end.R += dRow;
      } else {
        this.setActiveRow(this.active.R + dRow);
        this.setActiveColumn(this.active.C + dColumn);
        this.selection.start.R = this.active.R;
        this.selection.end.R = this.active.R;
        this.selection.start.C = this.active.C;
        this.selection.end.C = this.active.C;
      }
      this.clampSelection();
      this.scrollToRow(this.selection.end.R);
    },
    OnTab(event) {
      if (this.selectedCellCount == 1) {
        if (event.shiftKey) {
          const newRow = this.active.C === 0;
          if (newRow && this.active.R == 0) {
            return;
          }
          this.OnArrow(
            event,
            newRow ? this.columnCount : -1,
            newRow ? -1 : 0,
            true
          );
        } else {
          const newRow = this.active.C === this.columnCount - 1;
          if (newRow && this.active.R == this.rowCount - 1) {
            return;
          }
          this.OnArrow(
            event,
            newRow ? -this.columnCount : 1,
            newRow ? 1 : 0,
            true
          );
        }
        return;
        const rowShift = newRow ? (event.shiftKey ? -1 : 1) : 0;
        const columnShift = event.shiftKey ? -1 : 1;
        this.OnArrow(event, columnShift, rowShift, true);
        return;
      }
      const last = event.shiftKey
        ? this.selectedRange.start
        : this.selectedRange.end;
      const first = event.shiftKey
        ? this.selectedRange.end
        : this.selectedRange.start;
      const newRow = this.active.C === last.C;
      this.setActiveColumn(
        newRow ? first.C : this.active.C + (event.shiftKey ? -1 : 1)
      );
      if (newRow) {
        this.setActiveRow(
          this.active.R === last.R
            ? first.R
            : this.active.R + (event.shiftKey ? -1 : 1)
        );
      }
    },
    OnEnter(event) {
      if (this.isEditing) {
        this.endEdit(event.ctrlKey);
      }
      if (this.selectedCellCount == 1) {
        this.OnArrow(event, 0, event.shiftKey ? -1 : 1, true);
        return;
      }
      const last = event.shiftKey
        ? this.selectedRange.start
        : this.selectedRange.end;
      const first = event.shiftKey
        ? this.selectedRange.end
        : this.selectedRange.start;
      const newColumn = this.active.R === last.R;
      this.setActiveRow(
        newColumn ? first.R : this.active.R + (event.shiftKey ? -1 : 1)
      );
      if (newColumn) {
        this.setActiveColumn(
          this.active.C === last.C
            ? first.C
            : this.active.C + (event.shiftKey ? -1 : 1)
        );
      }
    },
    selectAll() {
      this.selection.start.R = 0;
      this.selection.end.R = this.rowCount - 1;
      this.selection.start.C = 0;
      this.selection.end.C = this.columnCount - 1;
    },
    clampSelection() {
      this.selection.start.R = clamp(
        this.selection.start.R,
        0,
        this.rowCount - 1
      );
      this.selection.end.R = clamp(this.selection.end.R, 0, this.rowCount - 1);
      this.selection.start.C = clamp(
        this.selection.start.C,
        0,
        this.columnCount - 1
      );
      this.selection.end.C = clamp(
        this.selection.end.C,
        0,
        this.columnCount - 1
      );
    },
    selectColumns(columns) {
      this.active.R = 0;
      this.active.C = columns.start;
      if (!event.shiftKey) {
        this.selection.start.C = columns.start;
      }
      this.selection.end.C = columns.end;
      this.selection.start.R = 0;
      this.selection.end.R = this.rowCount - 1;
    },
    scrollTable(event) {
      const scrollTop = event.target.scrollTop;
      if (this.rowCount > MAX_ROWS) {
        const step = scrollTop - this.lastScrollTop;
        if (Math.abs(step) > 0.01 * MAX_ROWS) {
          if (scrollTop < MAX_ROWS) {
            //align with top
            this.referenceRow.row = 0;
            this.referenceRow.scrollTop = 0;
          } else if (event.target.scrollHeight - scrollTop < MAX_ROWS) {
            // align with bottom
            this.referenceRow.row = this.rowCount;
            this.referenceRow.scrollTop = event.target.scrollHeight;
          } else {
            // align based on percentage
            this.referenceRow.row = Math.round(
              (this.rowCount * scrollTop) / event.target.scrollHeight
            );
            this.referenceRow.scrollTop = scrollTop;
          }
        }

        const distanceFromReferenceRow =
          scrollTop - this.referenceRow.scrollTop;
        const rowsFromReferenceRow = Math.round(distanceFromReferenceRow / 30);
        const startRow = this.referenceRow.row + rowsFromReferenceRow;
        this.startRow = Math.max(startRow - BUFFER_ROWS, 0);
        const lastRow = Math.min(
          this.startRow + ROW_COUNT + 2 * BUFFER_ROWS,
          this.rowCount
        );
        this.renderedRows = lastRow - this.startRow;
        this.topBufferHeight =
          this.referenceRow.scrollTop + 30 * rowsFromReferenceRow;
        this.topBufferHeight -= 30 * (startRow - this.startRow);
      } else {
        let firstVisibleRow = Math.round(scrollTop / 30);
        this.startRow = Math.max(firstVisibleRow - BUFFER_ROWS, 0);
        const lastRow = Math.min(
          this.startRow + ROW_COUNT + 2 * BUFFER_ROWS,
          this.rowCount
        );
        this.renderedRows = lastRow - this.startRow;
        this.topBufferHeight = this.startRow * ROW_HEIGHT;
      }
      this.lastScrollTop = scrollTop;
    },
    onCellMouseDown(cell, event) {
      if (this.isEditing) {
        if (cell.C === this.active.C && cell.R === this.active.R) {
          this.isDeepEditing = true;
        } else {
          this.endEdit();
        }
      }

      if (cell.C == -1) {
        this.selection.start.C = 0;
        this.selection.end.C = this.columnCount - 1;
      } else {
        this.selection.end.C = cell.C;
        if (!event.shiftKey) {
          this.selection.start.C = cell.C;
          this.setActiveColumn(cell.C);
        }
      }

      this.selection.end.R = cell.R;
      if (!event.shiftKey) {
        this.selection.start.R = cell.R;
        this.setActiveRow(cell.R);
      }
    },
    onCellMouseOver(cell, event) {
      if (event.buttons == 1) {
        if (cell.C >= 0) {
          this.selection.end.C = cell.C;
        }
        this.selection.end.R = cell.R;
      }
    },
    onCellDblClick() {
      this.startEdit();
    },
    setActiveColumn(iCol) {
      if (this.isEditing && iCol !== this.active.C) {
        this.endEdit();
      }
      this.active.C = clamp(iCol, 0, this.columnCount - 1);
    },
    setActiveRow(iRow) {
      if (this.isEditing && iRow !== this.active.R) {
        this.endEdit();
      }
      this.active.R = clamp(iRow, 0, this.rowCount - 1);
    },
    OnDelete() {
      if (this.isEditable) {
        this.setRangeValue(this.selectedRange, null);
      }
    },
    startEdit(resetValue) {
      if (!this.isEditable) return;
      let cell = this.getCellValue(this.active.R, this.active.C);
      if (cell.type === "boolean" || cell.type === "button") {
        return;
      }
      this.inputValue = resetValue ? "" : cell.value;
      this.isEditing = true;
      this.isDeepEditing = !resetValue;
      this.$nextTick(() => this.$refs.theinput[0].focus());
    },
    endEdit(fillSelection) {
      if (!this.isEditing) {
        return;
      }
      if (fillSelection) {
        this.setRangeValue(this.selectedRange, this.inputValue);
      } else {
        this.setRangeValue(
          this.cellRange(this.active.R, this.active.C),
          this.inputValue
        );
      }
      this.cancelEdit();
    },
    cancelEdit() {
      this.isEditing = false;
      this.isDeepEditing = false;
      this.refocus();
    },
    refocus() {
      this.$refs.tableContainer.focus();
    },
    tableRowHeaderStyle(row) {
      return {
        rowId: true,
        active: row >= this.selectedRows.start && row <= this.selectedRows.end,
        selected: this.isTableActive && this.allColumnsSelected
      };
    }
  },
  computed: {
    contentHeight() {
      return Math.min(ROW_COUNT, this.rowCount) * ROW_HEIGHT + 2;
    },
    selectedRange() {
      if (!this.isSelectable) {
        return {
          start: new CellReference(),
          end: new CellReference()
        };
      }
      return {
        start: {
          R: Math.min(this.selection.start.R, this.selection.end.R),
          C: Math.min(this.selection.start.C, this.selection.end.C)
        },
        end: {
          R: Math.max(this.selection.start.R, this.selection.end.R),
          C: Math.max(this.selection.start.C, this.selection.end.C)
        }
      };
    },
    selectedCellCount() {
      return (
        (1 + this.selectedRange.end.C - this.selectedRange.start.C) *
        (1 + this.selectedRange.end.R - this.selectedRange.start.R)
      );
    },
    computedHeaders() {
      if (
        Array.isArray(this.headers) ||
        this.headers == false ||
        this.headers == "false"
      ) {
        return this.headers;
      }
      let headers = [];
      for (var column = 1; column <= this.columnCount; column++) {
        headers.push({ name: this.defaultColumnName(column) });
      }
      return headers;
    },
    renderedRowCount() {
      return Math.min(this.renderedRows, this.rowCount - this.startRow);
    },
    renderedRowArray() {
      let renderedRows = [];
      var currentRow = this.startRow;
      for (var i = 0; i < this.renderedRowCount; i++) {
        renderedRows.push(currentRow++);
      }
      return renderedRows;
    },
    totalHeight() {
      return Math.min(MAX_ROWS, this.rowCount) * ROW_HEIGHT + 3;
    },
    allRowsSelected() {
      return (
        this.selection.start.R == 0 && this.selection.end.R == this.rowCount - 1
      );
    },
    allColumnsSelected() {
      return (
        this.selection.start.C == 0 &&
        this.selection.end.C == this.columnCount - 1
      );
    },
    selectedColumns() {
      return {
        start: this.selectedRange.start.C,
        end: this.selectedRange.end.C
      };
    },
    selectedRows() {
      return {
        start: this.selectedRange.start.R,
        end: this.selectedRange.end.R
      };
    },
    tableBodyInnerStyle() {
      return {
        height: this.totalHeight + "px"
      };
    },
    tableBodyVisibleStyle() {
      return {
        position: "relative",
        top: this.topBufferHeight + "px",
        borderCollapse: "collapse"
        //background: "blue"
      };
    }
  }
};
</script>

<style scoped>
.component-container {
  max-height: inherit;
  display: flex;
}
.table-container {
  outline: none;
  display: flex;
  flex-direction: column;
  max-height: inherit;
}

:not(input)::selection {
  background: none;
}

.table-body-container {
  overflow: hidden;
  flex: 1;
  display: flex;
  flex-direction: column;
}
.table-busy {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.54);
  display: flex;
  transition: all 1s;
}
.table-body {
  overflow-y: auto;
  cursor: cell;
  flex: 1;
}

.the-input {
  display: block;
  border: none;
  padding: 0;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
  width: 100%;
}
col {
  border-left: 1px solid #aaa;
}
tr {
  border: 1px solid #aaa;
  //border-top-width: 0px;
}

th {
  color: rgba(0, 0, 0, 0.54);
  font-weight: inherit;
}

.centre {
  text-align: center;
}
.disabled {
  color: rgb(150, 150, 150);
}
.rowId {
  width: 48px;
  min-width: 48px;
  max-width: 48px;
  position: relative;
}
.rowId.active.selected {
  background: rgba(0, 135, 189, 0.1);
}
.row-number-overlay {
  position: absolute;
  top: -1px;
  bottom: -1px;
  right: 0;
  width: 1em;
  background-image: linear-gradient(
    to right,
    rgba(0, 135, 189, 0),
    rgba(0, 135, 189, 0.2)
  );
  z-index: 1000;
}
input {
  outline: none;
}
select {
  display: block;
  background: none;
  width: 100%;
  border: none;
  font: inherit;
  padding: 0;
  outline: none;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
  transition-delay: 0.2s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>
