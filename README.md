# infinite-table
A high performance, virtual scrolling, lazy loading, editable data table of unlimited length.<br>Currently implemented as a Vue component.


>PLEASE NOTE<br>
This is a work in progress project! Most of the features described below are not yet implemented (and will likely change)!<br>
The npm package has not yet been published<br>
Please check back soon for updates!

## Installation

```
npm install @infinite-table/vue
```
## Usage

Import the component where you need it
```javascript
import InfiniteTable from '@infinite-table/vue'
```
For basic usage you can bind a 2D array of [field values](#fieldValues) to the `infinite-table` component
```html
<infinite-table v-model="myData"/>
```
Alternatively, we can provide a function to get the field values and subscribe to the table's events for updates. When the size of the table gets very large, this approach can help maintain high performing table by only keeping the visible data in memory.
```html
<infinite-table :get="myDataGetter"/>
```
In this example `myDataGetter` would need to be a function that is called with two arguments - the row and column index of the cell.

## Field values<a name="fieldValues"></a>
Each cell in an `infinite-table` is populated using a 'field value' which can be provided either as the value primitive or as an object with required `value` property and optional overrides of the column's `type`, `options`, `required` and/or `disabled` parameters.

## Headers
The `headers` prop allows column headers (and associated field's attributes) to be added and customised.
```html
<!-- default headers (A, B, C...) -->
<infinite-table />

<!-- custom headers -->
<infinite-table :headers="myHeaders" />

<!-- no headers -->
<infinite-table :headers="false" />
```
Where `myHeaders` is an array of header objects following the schema below.<br> All properties are optional.
|Property|Type|Default|
|---|---|---|---|
|`name`|String|Alphabetical (A, B, C etc)
|`type`|String|`"string"` (see [Supported types](#SupportedTypes))
|`width`|Number|`6` (units are em)
|`disabled`|Boolean|`false`
|`unit`|String|none
|`options`|Array|none (see [options](#Options))
|`children`|Array|Empty array of nested sub-header objects

## Supported types<a name="SupportedTypes"></a>
The following field value types are supported
* `"string"`
* `"number"`
* `"integer"`
* `"boolean"` (renders as checkbox)
* `"button"` (renders as button)

## Options<a name="options"></a>
If a field has an associated `options` property (either directly, or inherited from the column headers), it will be rendered with a `<select>` HTML component. Options can be passed as an array of strings (representing both the option name and value) or as an array of objects with `name` and `value` properties.

## Events
The `infinite-table` component will emit the following events which, if subscribed to, will override the default implementation. This can be useful for large datasets where the (potentially many) required calls to the tables data's `get` and `set` methods could become expensive.

### Event Payload Types
```javascript
CellReference = {
    row: rowIndex,  //zero based indeces
    column: columnIndex
}

CellRange = {
    start: CellReference,
    end: CellReference
}

SetValues = {
    range: CellRange,
    value: newValue
}
```

|Event name|Payload|
|---|---|
|`set`|SetValues
|`cut`|CellRange
|`copy`|CellRange
|`paste`|CellRange
|`undo`|none
|`redo`|none
|`button-click`|CellReference (cell where the button was clicked)
|`insert-row`|Insert row range `{row:#, count:#}`
|`delete-row`|Delete row range (as above)
|`insert-column`|Insert column range `{column:#, count:#}`
|`delete-column`|Delete column range (as above)