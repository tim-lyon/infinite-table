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
Alternatively, we can bind to a 'data interface' object, allowing a field's data to be fetched only when required. When the size of the table gets very large, this approach can help maintain high performing table by only keeping the visible data in memory.
```html
<infinite-table :data-interface="myDataInterface"/>
```
Where `myDataInterface` is an object with the following properties
|Property|Type|Description|
|---|---|---|---|
|`count`|Number|The number of data records/rows in the table
|`get`|Function|Returns a field's value given its position in the table
|optional
|`set`|Function|Called when a field's value is changed
|`destroy`|Function|Called when a field's data is no longer required

## Field values<a name="fieldValues"></a>
Each cell in an `infinite-table` is populated using a 'field value' which can be provided either as the value primitive or as an object with required `value` property and optional overrides of the column's `type`, `options`, `required` and/or `disabled` parameters.

## Headers
The `headers` prop allows column headers (and associated field's attributes) to be added and customised.
```html
<!-- default headers (A, B, C...) -->
<infinite-table headers />

<!-- custom headers -->
<infinite-table :headers="myHeaders" />
```
Where `myHeaders` is an array of header objects following the schema below.<br> All properties are optional.
|Property|Type|Default|
|---|---|---|---|
|`name`|String|Alphabetical (A, B, C etc)
|`type`|String|`"text"` (see [Supported types](#SupportedTypes))
|`width`|Number|`6` (units are em)
|`required`|Bollean|`false`
|`disabled`|Boolean|`false`
|`unit`|String|none
|`options`|Array|none (see [options](#Options))
|`children`|Array|Empty array of nested sub-header objects
|`validators`|Array of functions|none

## Supported types<a name="SupportedTypes"></a>
The following field value types are supported
* `"text"`
* `"integer"`
* `"decimal"`
* `"bool"` (renders as checkbox)

## Options<a name="options"></a>
If a field has an associated `options` property (either directly, or inherited from the column headers), it will be rendered with a `<select>` html component. Options can be passed as an array of strings (representing both the option name and value) or as an array of objects with `name` and `value` properties.

## Validators
An array of validation functions can be provided in the `headers` (or overriden in a field value). On change of a field's value, each of the validator functions will be called in sequence with the proposed value as the only parameter. If any of the validatos return `false` then the `infinite-table` will not mutate the value/emit a change event.

## Events
The `infinite-table` component will emit the following events which, if subscribed to, will override the default implementation. This can be useful for large datasets where the (potentially many) required calls to the data interface's `get` and `set` methods could become expensive.
|Event name|Payload|
|---|---|
|`copy`|Selected range `{start: {row:#, column:#}, end: {row:#, column:#}}`
|`paste`|Selected range
|`delete`|Selected range
|`insert-row`|Insert row range `{row:#, count:#}`
|`delete-row`|Delete row range (as above)
|`insert-column`|Insert column range `{column:#, count:#}`
|`delete-column`|Delete column range (as above)

## Export data
By default, `infinite-table` will include an "Export CSV" option. This can be disabled by adding the prop `exports="false"`. Alternatively, the default export functionality can be extended (and overriden) by passing an `exports` object with each property representing a custom export function. The function will be called without parameters - the implementation should export the entire data range.
```javascript
let myExports = {
    CSV: myCsvExportFunction,
    PDF: myPdfExportFunction
}
```
```html
<infinite-table :exports="myExports" />
```