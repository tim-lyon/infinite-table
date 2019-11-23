<template>
  <div id="app">
    <h1>When using Infinite Table you have two options</h1>
    <h2>1) Bind to an array of data</h2>
    <p>This is the easiest way to get started - simply use v-bind to bind data to the table. You can select, edit, copy, paste, undo etc. This approach should work just nicely until the quantity of data starts to take too long to fetch from a remote resource or occupy too much space in memory.</p>

    <InfiniteTable
      v-model="sampleData2"
      :headers="headers"
      :editable="editable"
      :selectable="selectable"
    />

    <h2>2) Bind to a data function</h2>
    <p>
      This is where the infinite power begins! Just specify the number of rows and a function to fetch your data.
      Here we have an example table with 1 million rows. You can still edit the table just as above - try it out, for example "Ctrl+A" to select all 4 million cells, then delete their contents. You can undo with "Ctrl+Z".
    </p>
    <InfiniteTable
      :row-count="1e6"
      :column-count="5"
      :headers="headers"
      :editable="editable"
      :selectable="selectable"
      :get="getCellValue"
      @set="setRangeValue"
      @undo="undo"
      @redo="redo"
      @cut="cut"
      @copy="copy"
      @paste="paste"
    />Table options
    <div class="options-container">
      <div>
        Headers
        <br />
        <input type="radio" id="default" value="default" v-model="headerOption" />
        <label for="default">Default</label>
        <br />
        <input type="radio" id="custom" value="custom" v-model="headerOption" />
        <label for="custom">Custom</label>
        <br />
        <input type="radio" id="none" value="none" v-model="headerOption" />
        <label for="none">None</label>
      </div>
      <div>
        Editing
        <br />
        <input type="radio" id="edit" value="edit" v-model="editOption" />
        <label for="edit">Editable</label>
        <br />
        <input type="radio" id="select" value="select" v-model="editOption" />
        <label for="select">Selectable</label>
        <br />
        <input type="radio" id="neither" value="neither" v-model="editOption" />
        <label for="neither">Neither</label>
      </div>
    </div>
  </div>
</template>

<script>
import InfiniteTable from "./components/InfiniteTable.vue";

export default {
  name: "app",
  components: {
    InfiniteTable
  },
  data() {
    return {
      headerOption: "default",
      editOption: "edit",
      customHeaders: [
        {
          name: "Text field inputs",
          children: [
            { name: "String", type: "string" },
            { name: "Numeric", type: "number" }
          ]
        },
        { name: "Button", type: "button", unit: "years" },
        {
          name: "Select",
          type: "string",
          options: ["Option 1", "Option 2", "Option 3"]
        },
        { name: "Boolean", type: "boolean", unit: "years" }
      ],
      sampleData1: [
        ["Some", "sample", "data!"],
        ["You", "can", "select"],
        ["edit", "and", "delete,"],
        ["cut", "copy and", "paste,"],
        ["undo", "redo", "etc..."]
      ],
      sampleData2: [
        ["string 1", 123, "Button 1", "Option 1", true],
        ["string 2", 0, "Button 2", "Option 2", true],
        ["string 3", null, "Button 3", "", false],
        ["string 4", 888, "Button 4", "Option 1", true]
      ],
      sampleData3: [
        ["This", "table", "has", "no", "headers"],
        ["and", "selecting", "and", "editing", "of"],
        ["data", "has", "been", "disabled", ""]
      ],
      buttonClick: cell =>
        alert("Buttton clicked! row: " + cell.row + " column: " + cell.column),
      setBuffer: [],
      undoBuffer: [],
      contentToCopy: ""
    };
  },
  computed: {
    headers() {
      return this.headerOption == "custom"
        ? this.customHeaders
        : this.headerOption === "default";
    },
    editable() {
      return this.editOption === "edit";
    },
    selectable() {
      return this.editOption !== "none";
    }
  },
  methods: {
    getCellValue(row, column) {
      let value = {
        value: "R" + row + "C" + column
      };
      for (var set of this.setBuffer) {
        if (
          !value.disabled &&
          row >= set.range.start.R &&
          row <= set.range.end.R &&
          column >= set.range.start.C &&
          column <= set.range.end.C
        ) {
          value.value = set.value;
        }
      }
      return value;
    },
    setRangeValue(payload) {
      this.setBuffer.push(payload);
      this.undoBuffer.length = 0;
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
              value,
              isGroup: true
            });
          });
        });
      });
    },
    undo() {
      if (this.setBuffer.length == 0) {
        return;
      }
      const isGroup = this.setBuffer[this.setBuffer.length - 1].isGroup;
      if (isGroup) {
        while (this.setBuffer[this.setBuffer.length - 1].isGroup) {
          this.undoBuffer.push(this.setBuffer[this.setBuffer.length - 1]);
          this.setBuffer.pop();
        }
      } else {
        this.undoBuffer.push(this.setBuffer[this.setBuffer.length - 1]);
        this.setBuffer.pop();
      }
    },
    redo() {
      if (this.undoBuffer.length === 0) {
        return;
      }
      const isGroup = this.undoBuffer[this.undoBuffer.length - 1].isGroup;
      if (isGroup) {
        while (this.undoBuffer[this.undoBuffer.length - 1].isGroup) {
          this.setBuffer.push(this.undoBuffer[this.undoBuffer.length - 1]);
          this.undoBuffer.pop();
        }
      } else {
        this.setBuffer.push(this.undoBuffer[this.undoBuffer.length - 1]);
        this.undoBuffer.pop();
      }
    }
  }
};
</script>

<style>
#app {
  font-family: calibri;
}
.options-container {
  display: flex;
}
</style>
