<template>
  <InfiniteTableCore
    ref="table"
    :headers="headers"
    :row-count="value.length"
    :column-count="value[0].length"
    :get="getCellValue"
    :editable="editable"
    :selectable="selectable"
    @set="setRangeValue"
    @cut="cut"
    @copy="copy"
    @paste="paste"
    @undo="undo"
    @redo="redo"
    @buttonClick="buttonClick"
  />
</template>

<script>
import InfiniteTableCore from "./InfiniteTableCore.vue";

export default {
  name: "InfiniteTableWrapper",
  props: {
    value: Array,
    headers: {
      type: [Array, Boolean],
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
    }
  },
  components: {
    InfiniteTableCore
  },
  data() {
    return {
      buttonClick: cell =>
        alert("Buttton clicked! row: " + cell.row + " column: " + cell.column),
      setBuffer: [],
      undoBuffer: [],
      contentToCopy: ""
    };
  },
  methods: {
    getCellValue(row, column) {
      return this.value[row][column];
    },
    setRangeValue(payload) {
      this.setBuffer.push(payload);
      this.undoBuffer.length = 0;

      for (var row = payload.range.start.R; row <= payload.range.end.R; row++) {
        for (
          var column = payload.range.start.C;
          column <= payload.range.end.C;
          column++
        ) {
          let value = this.value[row][column];
          if (typeof value === "object" && value !== null) {
            if (value.type != "button") {
              value.value = payload.value;
            }
          } else {
            this.$set(this.value[row], column, payload.value);
          }
        }
      }
      this.$emit("update", this.value);
    },
    cut(range) {
      this.copy(range);
      this.setRangeValue({ range, value: null });
    },
    copyDataToClipboard(event) {
      event.preventDefault();
      event.clipboardData.setData("text/plain", this.contentToCopy);
    },
    copy(range) {
      let rows = [];
      for (var row = range.start.R; row <= range.end.R; row++) {
        let values = [];
        for (var column = range.start.C; column <= range.end.C; column++) {
          const value = this.getCellValue(row, column);
          values.push(typeof value === "object" ? value.value : value);
        }
        rows.push(values.join("\t"));
      }
      this.contentToCopy = rows.join("\r\n");
      document.addEventListener("copy", this.copyDataToClipboard);
      try {
        document.execCommand("copy");
      } catch (exception) {
        alert(exception);
      } finally {
        document.removeEventListener("copy", this.copyDataToClipboard);
      }
    },
    paste(range) {
      navigator.clipboard.readText().then(text => {
        text.split("\r\n").forEach((row, iRow) => {
          row.split("\t").forEach((value, iColumn) => {
            const cell = {
              R: range.start.R + iRow,
              C: range.start.C + iColumn
            };
            this.setRangeValue({
              range: {
                start: cell,
                end: cell
              },
              value
            });
          });
        });
      });
    },
    undo() {
      const setCount = this.setBuffer.length;
      if (setCount > 0) {
        this.undoBuffer.push(this.setBuffer[setCount - 1]);
        this.setBuffer.pop();
      }
    },
    redo() {
      const undoCount = this.undoBuffer.length;
      if (undoCount > 0) {
        this.setBuffer.push(this.undoBuffer[undoCount - 1]);
        this.undoBuffer.pop();
      }
    }
  }
};
</script>

<style scoped>
</style>
