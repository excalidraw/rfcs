* Start Date: 2024-07-13
* Referenced Issues: https://github.com/excalidraw/excalidraw/issues/4847
* Implementation PR: (leave this empty)

# Supporting Tables in Excalidraw

Tables is nothing but a collection of Text Containers next to each other in form of a grid with `x` rows and `y` columns.

Lets see how the data structure of table element look like.

A `table` element will have `rows` and `columns` defining the grid. The `table` will also have `x` , `y` coordinates for positioning on canvas and also the `width` and `height` for dimensions.

The `table` element consists of multiple cells. Each cell will have an `id`, `x` and `y` coordinates and dimensions as well since each cell's `width/height` can be varied and some text. Each cell is a text container (rectangle with bound text).

Below is the sample JSON representation of a `table` element

```js
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "cells": [
    {
      "id": "cell-1",
      "tableId":"table-id",
      "row": 0,
      "column": 0,
      "x": 100,
      "y": 100,
      "width": 100,
      "height": 50,
      "boundElements": [{id: "text-1", type: "text"}]
    },
    {
      "id": "cell-2",
      "tableId":"table-id",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
       "boundElements": [{id: "text-2", type: "text"}]
    },
    {
      "id": "cell-3",
      "tableId":"table-id",
      "x": 100,
      "y": 150,
      "row": 1,
      "column": 0,
      "width": 120,
      "height": 60,
      "boundElements": [{id: "text3", type: "text"}]
    },
    {
      "id": "cell-4",
      "tableId":"table-id",
      "x": 220,
      "y": 150,
      "row": 1,
      "column": 1,
      "width": 130,
      "height": 60,
      "boundElements": [{id: "text-4", type: "text"}]
    }
  ],
  "angle": 0,
  "strokeColor": "#000000",
  "backgroundColor": "transparent",
  "fillStyle": "hachure",
  "strokeWidth": 1,
  "strokeStyle": "solid",
  "roughness": 0,
  "opacity": 100,
  "groupIds": [],
  "seed": 1,
  "version": 1,
  "versionNonce": 1,
  "isDeleted": false,
  "boundElements": null,
  "link": null,
  "locked": false
}

```

A table will have a `title` as well.
Each cell has a `tableId` to identify the `table` which it is connected to when interacting with the cell eg resizing, typing text etc.

## Adding and Deleting rows / columns

To start with we can keep it simple and just allow to add
3 x 3 grid table. Once table is added we can have a add row and add column button to increase the rows and columns similar to how **Miro** has.

![uploaded image](https://i.imgur.com/X21LzGR.png)

Similarly and entire `row` and `column` can be removed by right clicking a cell. We just set the attribute `isDeleted` to true for all cells in that row / column and since we are storing the `row` and `column` index in each cell, it will be easy to identify which cells to remove.

## Interacting with tables

Similar to other shape tool, the table tool will be resizable, movable etc.

Whenever user clicks on the cell - show a text box to updated or add the text. The text will start wrapping as per the cell dimensions similar to how it works for text container.

Additionally if a user clicks on the cell stroke (width / height) should be adjustable. So we need to do a hit test whether the cell stroke is being hit and allow the user to adjust dimensions.
But since this would be internally using rectangles so this should be possible without custom code.
What we need to support is allow the width / height of all cells in that row /column when any one of them is moved.

Since the table is not just a single element but a collection of different elements, whenever there is an operation eg moving , resizing, we need to update all the children of the table (basically every cell). And all the cells can be identified with the `tableId`.

## Supporting row and column headers

Ideally the first row and column should be preserved for the row and column header. But do we need a separate distinction for headers ? (Eg showing header in diff background color or some highlighter how miro does it).

I think the users will be able to style the headers differently once we have support for wyswyg editor, so till then lets keep it simple
