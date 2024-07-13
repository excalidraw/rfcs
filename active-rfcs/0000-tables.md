- Start Date: 2024-07-13
- Referenced Issues: https://github.com/excalidraw/excalidraw/issues/4847
- Implementation PR: (leave this empty)

# Summary

Once this feature is implemented the users will be able to use tables in excalidraw.

# Motivation

Tables are one of the most heavily requested features in Excalidraw for past few years. This will also be a very useful feature as well hence we should consider pushing it out soon.

# Detailed Design

Tables are nothing but a collection of Text Containers next to each other in form of a grid with `x` rows and `y` columns.

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
  "rows": 2,
  "columns": 2,
  "cells": [
    {
      "id": "cell-1",
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

A table will have a `title` as well and the count of `rows` and `columns` as well.

The table consists of `cells` defining the content of each cell.
Each cell has an `id`, `x`, `y` and dimensions. Since when resizing the width and height of cells can be altered hence recording the dimensions of each cell.

The `row` and `column` helps in identifying the row and column indexes of the cell.

Each cell has a `boundElements` to keep it in sync with text containers / labeled arrows, however we can surely rename it to `textId` to keep it simple.

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

Since the table is not just a single element but a collection of different elements, whenever there is an operation eg moving , resizing, we need to update all the children of the table (basically every cell).

## Supporting row and column headers

Ideally the first row and column should be preserved for the row and column header. But do we need a separate distinction for headers ? (Eg showing header in diff background color or some highlighter how miro does it).

I think the users will be able to style the headers differently once we have support for wyswyg editor, so till then lets keep it simple.

## Questions

- Should the `cells` in the above structure be actual text elements or virtual elements which are just rendered on canvas but physically they don't exist in json? This will definately simplify the data structure, however there are some unknowns listed below.

- With above approach of virtual elements, how will impact the overall performance of interacting with tables since we heavily rely on actual elements ?
- How will this impact collaboration ? We may need a custom logic for tables?

# Adoption strategy

- If we implement this proposal, how will existing Excalidraw users adopt it?

This is a new feature so adoption should be straight forward and they will be using it as needed.

- Is this a breaking change? If yes how are migrating the existing Excalidraw users ?

No this is not a breaking change (unless we change some existing implementation) so hopefully no migration will be needed.
