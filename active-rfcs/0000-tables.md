- Start Date: 2024-07-13
- Referenced Issues: https://github.com/excalidraw/excalidraw/issues/4847
- Implementation PR: (leave this empty)

# Summary

Once this feature is implemented the users will be able to use tables in excalidraw. The `table` will be added to the shapes toolbar similar to other tools.

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

Ideally the `first` `row` and `column` should be preserved for the `row` and `column` `header`. But do we need a separate distinction for headers ? (Eg showing header in diff background color or some highlighter how miro does it).

I think the users will be able to style the headers differently once we have support for wyswyg editor, so till then lets keep it simple.

## Reordering rows and columns

This is a powerful feature and good to have. Users can drag the rows and columns and reorder them on the UI.
However I think in the first release we might not want to support this and add this as an enhancement in future releases.

## Alternatives

### Virtual Text Elements

In the above approach the text elements with `containerId` as `cell id` will exists in the `json`. This means there will be 4 separate `text` elements in the `json`.

Here is an alternative version - Having `cells` as virtual text elements instead of actual text elements.

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
      "text":"Cell 1",
      "fontSize": 16;
      "fontFamily": 1;

    },
    {
      "id": "cell-2",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "text":"Cell 2",
      "fontSize": 16;
      "fontFamily": 1;
    },
    {
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "row": 1,
      "column": 0,
      "width": 120,
      "height": 60,
      "text":"Cell 3",
      "fontSize": 16;
      "fontFamily": 1;
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "row": 1,
      "column": 1,
      "width": 130,
      "height": 60,
      "text":"Cell 4",
      "fontSize": 16;
      "fontFamily": 1;
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

As you can see above the `cells` will contain all the attributes (only some of text attributes are shown above) and at the time of rendering these text elements will be drawn, however we won't be storing the text elements as separate elements.
This would simplify the data structure and reduce the hierarchy, thus helping in better maintainance

#### Questions

When `cells` are virtual elements which are just rendered on canvas but physically they don't exist as separate text elements? This will definately simplify the data structure, however there are some unknowns listed below which needs to be evaluated before we move with this approach.

- With above approach of virtual elements, how will impact the overall performance of interacting with tables since we heavily rely on actual elements ?
- Need to verify if this approach is feasable with current state of TextWYSIWYG
- How will this impact collaboration ? We may need a custom logic for tables?

Yes we will need write custom logic to handle reconcilation in `tables`, mainly attaching `timestamp` to each cell and writing a custom algorithm to resolve the conflicts and ensuring latest changes are applied to all the collaborators.

### Storing cells with respect to rows to ease reordering of rows and columns

In the suggested approach, reordering might get difficult as we will have to keep track of which cells should be swapped - lets say if we swap row 1 with row 3, we will have to iterate through all the cells and check which cells have `row` attribute set to `row 1` and swap them with cells with `row` attribute set to `row 3`.

Here is an alternate approach to ease the above process.

```js
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "rows": [{
    "id": "row-1",
    "cells": [{
      "id": "cell-1",
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
    }
  ]},
  {
    "id": "row-2",
    "cells:[{
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "boundElements": [{id: "text3", type: "text"}]
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "boundElements": [{id: "text-4", type: "text"}]
    }
  }],
  "columns:": [{
    "id": "column-1",
    "title": "Column 1",
  },
  {
    "id": "column-2",
    "title": "Column 2",
  }],
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

As you can see now all the data related to rows stays together and whenever the rows are swapped, we can swapped the entire row between `source` and `destination` index.

Whenever the columns are swapped, we will need to swap the cells of the `source` index in each `row` with the `destination` index in each `row`.

This way the swapping becomes easier since we need not filter the cells based on row / column positions.

#### Questions

- This approach definately will ease the swapping and handling of rows and columns. How do we track changes when columns or rows are reordered ?

We can add an `order` attribute which helps in maintaining the correct sequence of `rows` and `columns` and `lastUpdated` helps in resolving conflicts in reordering.

```js
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "rows": [{
    "id": "row-1",
    "order": 1,
    "lastUpdated": 1633257617000
    "cells": [{
      "id": "cell-1",
      "x": 100,
      "y": 100,
      "width": 100,
      "height": 50,
      "boundElements": [{id: "text-1", type: "text"}],
    },
    {
      "id": "cell-2",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "boundElements": [{id: "text-2", type: "text"}],
    }
  ]},
  {
    "id": "row-2",
    "order": 2,
    "lastUpdated": 1633257617000
    "cells:[{
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "boundElements": [{id: "text3", type: "text"}],
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "boundElements": [{id: "text-4", type: "text"}],

    }
  }],
  "columns:": [{
    "id": "column-1",
    "order": 1,
    "title": "Column 1",
  },
  {
    "id": "column-2",
    "order": 2,
    "title": "Column 2",
  }],
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

We will need to write a custom reconcilation to reconcile correctly for this case.

When there are updates we will compare the order and if order is updated, it means rows and columns were reordered and compare the `lastUpdated` as well to ensure most recent update is applied to all the collaborators.

Additionally each cell will also have a `timestamp` attribute if we consider virtual text elements (as mentioned in previous approach) to resolve conflicts when content updated on same cell.

```js
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "rows": [{
    "id": "row-1",
    "order": 1,
    "lastUpdated": 1633257617000
    "cells": [{
      "id": "cell-1",
      "x": 100,
      "y": 100,
      "width": 100,
      "height": 50,
      "content": "Cell-11",
      "timestamp": 1633257618000
    },
    {
      "id": "cell-2",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "content": "Cell-12",
      "timestamp": 1633257619000
    }
  ]},
  {
    "id": "row-2",
    "order": 2,
    "lastUpdated": 1633257620000
    "cells:[{
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "content": "Cell-21",
      "timestamp": 1633257621000
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "content": "Cell-22",
      "timestamp": 1633257617000

    }
  }],
  "columns:": [{
    "id": "column-1",
    "order": 1,
    "title": "Column 1",
    "timestamp": 1633257617000
  },
  {
    "id": "column-2",
    "order": 2,
    "title": "Column 2",
    "timestamp": 1633257617000
  }],
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

- Will this work for multiplayer undo/redo ?

# Adoption strategy

- If we implement this proposal, how will existing Excalidraw users adopt it?

This is a new feature so adoption should be straight forward and they will be using it as needed.

- Is this a breaking change? If yes how are migrating the existing Excalidraw users ?

No this is not a breaking change (unless we change some existing implementation) so hopefully no migration will be needed.

```

```

```
