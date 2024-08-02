- Start Date: 2024-07-13
- Referenced Issues: https://github.com/excalidraw/excalidraw/issues/4847
- Implementation PR: (leave this empty)

# Summary

Once this feature is implemented the users will be able to use tables in excalidraw. The `table` will be added to the shapes toolbar similar to other tools.

# Motivation

Tables are one of the most heavily requested features in Excalidraw for past few years. This will also be a very useful feature as well hence we should consider pushing it out soon.

# Detailed Design

Tables are nothing but a collection of Text Containers next to each other in form of a grid with `x` rows and `y` columns.

Let's see how the data structure of table element look like.

A `table` element will have `rows` and `columns` defining the grid. The `table` will also have `x` , `y` coordinates for positioning on canvas and also the `width` and `height` for dimensions.

The `table` element consists of multiple cells. Each cell will have an `id`, `x` and `y` coordinates and dimensions as well since each cell's `width/height` can be varied and some text. Each cell is a text container (rectangle with bound text).

Below is the sample JSON representation of a `table` element

```json
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "cells": {
    "cell-11": {
      "id": "cell-11",
      "content": "Cell-11",
      "rowId": "row-1",
      "columnId": "col-1", 
      "version": 1
    },
    "cell-12": {
      "id": "cell-12",
      "row": 0,
      "column": 1,
      "content": "Cell-12",
      "rowId": "row-1",
      "columnId": "col-2", 
      "version": 1
    },
    "cell-21": {
      "id": "cell-21",
      "content": "Cell-21",
      "rowId": "row-2",
      "columnId": "col-1",
      "version": 1
    },
    "cell-22": {
      "id": "cell-22",
      "width": 130,
      "height": 60,
      "content": "Cell-22",
      "rowId": "row-2",
      "columnId": "col-2", 
      "version": 1 
    }
  },
  "rows": {
    "row-1": {
      "id": "row-1",
      "index": 0,
      "cellIds": [
        "cell-11",
        "cell-12"
      ], 
      "version": 1,
      "isDeleted": false,
      "height": 50
    },
    "row-2": {
      "id": "row-2",
      "index": 1,
      "cellIds": [
        "cell-21",
        "cell-22"
      ],
      "version": 2,
      "isDeleted": false,
      "height": 60
    }
  },
  "columns:": {
    "column-1": {
      "id": "column-1",
      "index": 0,
      "title": "Column 1",
      "version": 2,
      "isDeleted": false,
      "width": 100
    },
    "column-2": {
      "id": "column-2",
      "index": 1,
      "title": "Column 2",
      "version": 1,
      "isDeleted": false,
      "width": 150
    }
  },
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
  "version": 2,
  "versionNonce": 19289929,
  "isDeleted": false,
  "boundElements": null,
  "link": null,
  "locked": false
}
```
* A table will have a `title` as well `x` and `y` coordinates as well.
* The table consists of `cells` which is an object, defining the content of each cell. These cells are virtual text elements. However, if we don't go ahead with virtual text elements then `boundTextElementIds` will be reintroduced.
  The `cells` is an object as the earlier approach was to keep it an array but as mentioned in the #alternatives` section, it was not a good idea to keep it as an array for performance reasons.

### cell
Each `cell` has the :point_down: attributes
* `id` - to uniquely identify the cell.
* `x` and `y` - to keep track of the position of the cell.
* `content` - the text content of the cell.
* `version` - to keep track of the version of the cell. This will be used in reconciliation to resolve conflicts when cells are updated.
* `rowId` and `columnId` - to keep track of the row and column it belongs to. This can be useful for quick lookup for updates, however will be removed if not needed.

### row
Each `row` has the :point_down: attributes
* `id` - to uniquely identify the row.
* `index` - To keep track of the index of the row. This will be used in reordering of rows. 
* `cellIds` - which is an array of cell ids. This helps in keeping track of the cells in the row.
* `version` - to keep track of the version of the row. This will be used in reconciliation to resolve conflicts when rows are updated / reordered.
* `isDeleted` - to keep track of whether the row is deleted or not.
* `height` - to keep track of the height of the row.

### column
Each `column` has the :point_down: attributes
* `id` - to uniquely identify the column.
* `index` - To keep track of the index of the column. This will be used in reordering of columns.
* `title` - the name of the column.
* `version` - to keep track of the version of the column. This will be used in reconciliation to resolve conflicts when columns are updated / reordered.
* `isDeleted` - to keep track of whether the column is deleted or not.
* `width` - to keep track of the width of the column.

- When resolving conflicts, we give preference to the client having highest `version`.
- In case multiple clients end up having the same `version`, we check which client has highest `versionNonce` and apply their updates.

## Adding and Deleting rows / columns

To start with we can keep it simple and just allow to add
3 x 3 grid table. Once table is added we can have "+" button to add row and add column button to increase the rows and columns similar to how **Miro** has.

![uploaded image](https://i.imgur.com/X21LzGR.png)

Similarly and entire `row` and `column` can be removed by right-clicking a cell. We just set the attribute `isDeleted` to true for all cells in that row / column and since we are storing the `row` and `column` index in each cell, it will be easy to identify which cells to remove.

## Interacting with tables

Similar to other shape tool, the table tool will be resizable, movable etc.

Whenever user clicks on the cell - show a text box to updated or add the text. The text will start wrapping as per the cell dimensions similar to how it works for text container.

Additionally, if a user clicks on the cell stroke (width / height) should be adjustable. So we need to do a hit test whether the cell stroke is being hit and allow the user to adjust dimensions.
But since this would be internally using rectangles so this should be possible without custom code.
What we need to support is allow the width / height of all cells in that row /column when any one of them is moved.

Since the table is not just a single element but a collection of different elements, whenever there is an operation eg moving , resizing, we need to update all the children of the table (basically every cell).

## Supporting row and column headers

Ideally the `first` `row` and `column` should be preserved for the `row` and `column` `header`. But do we need a separate distinction for headers ? (Eg showing header in diff background color or some highlighter how miro does it).

I think the users will be able to style the headers differently once we have support for wysiwyg editor, so till then lets keep it simple.

## Reordering rows and columns

This is a powerful feature and good to have. Users can drag the rows and columns and reorder them on the UI.
However, I think in the first release we might not want to support this and add this as an enhancement in future releases.
The `index` attribute in `row` and `column` will be used to resolve conflicts when reordering the rows and columns.

# Alternatives

## Cells as array of elements

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

### `Drawbacks`

This was the `first` approach however this would slow down the operations significantly during lookup since it's an array
hence not going ahead with this structure.

## `Storing cells with respect to rows to ease reordering of rows and columns`

In the suggested approach, reordering might get difficult as we will have to keep track of which cells should be swapped - lets say if we swap row 1 with row 3, we will have to iterate through all the cells and check which cells have `row` attribute set to `row 1` and swap them with cells with `row` attribute set to `row 3`.

Here is an alternate approach to ease the above process.

```json
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
      "boundElements": [{"id": "text-1", "type": "text"}]
    },
    {
      "id": "cell-2",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "boundElements": [{"id": "text-2", "type": "text"}]
    }
  ]},
  {
    "id": "row-2",
    "cells":[{
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "boundElements": [{"id": "text3", "type": "text"}]
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "boundElements": [{"id": "text-4", "type": "text"}]
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

As you can see now all the data related to rows stays together and whenever the rows are swapped, we can swap the entire row between `source` and `destination` index.

Whenever the columns are swapped, we will need to swap the cells of the `source` index in each `row` with the `destination` index in each `row`.

This way the swapping becomes easier since we need not filter the cells based on row / column positions.

### Questions

- This approach definitely will ease the swapping and handling of rows and columns. How do we track changes when columns or rows are reordered ?

We can add an `index` attribute which helps in maintaining the correct sequence of `rows` and `columns` and `lastUpdated` helps in resolving conflicts in reordering.

```json
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "rows": [{
    "id": "row-1",
    "index": 0,
    "lastUpdated": 1633257617000
    "cells": [{
      "id": "cell-1",
      "x": 100,
      "y": 100,
      "width": 100,
      "height": 50,
      "boundElements": [{"id": "text-1", "type": "text"}],
    },
    {
      "id": "cell-2",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "boundElements": [{"id": "text-2", "type": "text"}]
    }
  ]},
  {
    "id": "row-2",
    "index": 1,
    "lastUpdated": 1633257617000,
    "cells":[{
      "id": "cell-3",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "boundElements": [{"id": "text3", "type": "text"}]
    },
    {
      "id": "cell-4",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "boundElements": [{"id": "text-4", "type": "text"}]
    }
  }],
  "columns:": [{
    "id": "column-1",
    "index": 0,
    "title": "Column 1",
  },
  {
    "id": "column-2",
    "index": 1,
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

We will need to write a custom reconciliation to reconcile correctly for this case.

When there are updates we will compare the `index` and if `index` is updated, it means rows and columns were reordered and compare the `lastUpdated` as well to ensure most recent update is applied to all the collaborators.

Additionally, each cell will also have a `timestamp` attribute if we consider virtual text elements (as mentioned in previous approach) to resolve conflicts when content updated on same cell.

```json
{
  "id": "table-id",
  "type": "table",
  "x": 100,
  "y": 100,
  "title": "Table Title",
  "cells": {
    "cell-11": {
      "id": "cell-11",
      "x": 100,
      "y": 100,
      "width": 100,
      "height": 50,
      "content": "Cell-11",
      "timestamp": 1633257618000
    },
    "cell-12": {
      "id": "cell-12",
      "x": 200,
      "y": 100,
      "row": 0,
      "column": 1,
      "width": 150,
      "height": 50,
      "content": "Cell-12",
      "timestamp": 1633257619000
    },
    "cell-21":{
      "id": "cell-21",
      "x": 100,
      "y": 150,
      "width": 120,
      "height": 60,
      "content": "Cell-21",
      "timestamp": 1633257621000
    },
    "cell-22": {
      "id": "cell-22",
      "x": 220,
      "y": 150,
      "width": 130,
      "height": 60,
      "content": "Cell-22",
      "timestamp": 1633257617000
    }
  },
  "rows": {
    "row-1": {
    "id": "row-1",
    "index": 0,
    "cellIds":["cell-11", "cell-12"],
    "lastUpdated": 1633257617000
    },
    "row-2": {
    "id": "row-2",
    "index": 1,
    "cellIds":["cell-21", "cell-22"],
    "lastUpdated": 1633257620000,
    }
  },
  "columns": {
    "column-1": {
    "id": "column-1",
    "index": 0,
    "title": "Column 1",
    "timestamp": 1633257617000
  },
    "column-2": {
    "id": "column-2",
    "index": 1,
    "title": "Column 2",
    "timestamp": 1633257617000
    }
  },
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

### `Drawbacks`

This would slow down the operations significantly during lookup since it's still an array
hence not going ahead with this structure.

# Questions

1. If `cells` are virtual elements which are just rendered on canvas, but physically they don't exist as separate text elements. This will definitely simplify the data structure, however there are some unknowns listed below which needs to be evaluated before we move with this approach.

   - With above approach of virtual elements, how will impact the overall performance of interacting with tables since we heavily rely on actual elements ?
   - Need to verify if this approach is feasible with current state of TextWYSIWYG
   - How will this impact collaboration ? We may need a custom logic for tables?

    Yes we will need write custom logic to handle reconciliation in `tables`, mainly attaching `version` to each row/column and cell and using a combination with `versionNonce` as explained earlier and writing a custom algorithm to resolve the conflicts and ensuring latest changes are applied to all the collaborators.

2. Do we need `isDeleted` for each cell ?

   - Not needed as we can't just remove an individual cell, instead either an entire `row` or `column` can be removed.

3. Do we need per property syncing?

   - Yes we will need, to ensure that we don't ignore the changes made by other collaborators when cell properties updated eg font size, font family and later when we have support for wysiwyg editor, we need to ensure that the changes are in sync with all the collaborators.
   - However, for `tables` we might ignore text properties syncing and implement it later.
   - 
4. Do we need `width` and `height` for each cell?

   - Each cell can have different dimensions but when a dimension is updated, lets say if cell at (1,2) has increased width by `x`, this means the entire column width gets impacted by same amount `x`
   - Similarly, if cell at (1,2) has increased height by `y`, this means the entire row height gets impacted by same amount `y`
   - This means that `width` and `height` could be uplifted to the respective `column` and `row` attributes.
5. Are timestamps reliable for reconciliation ?

   - No, timestamps if generated on client are definitely not reliable for several factors, some of them mentioned :point_down:
      - Clock drifts - Different client may have different system times leading to inconsistencies
      - Timezone differences - Different clients may have different timezones leading to inconsistencies
      - Network latency - Different clients may have different network latencies leading to inconsistencies
   Hence client generated timestamps are not reliable for reconciliation. However, if we use server generated timestamps, they can also have network latency as well so not reliable as well.
   Hence, the way we do it now in production using `version` and `versionNonce` looks like the best way to handle reconciliation.
      - `version` - is the version of the element, and it increments for every update
      - `versionNonce` - is the nonce (random number) which is generated for every update
   Here is how it works
      - When a client updates an element, it increments the `version` and generates a new `versionNonce` for that element
      - When the client receives an update from the server, it compares the local `version` and the remote elements `version`, whichever is higher gets preference
      - If the `version` is same, then it compares the `versionNonce`, whichever is higher gets preference
      - In case of `version` being same, we end up randomly choosing the winner but that's ok as it works fine

# Adoption strategy

- If we implement this proposal, how will existing Excalidraw users adopt it?

This is a new feature so adoption should be straight forward, and they will be using it as needed.

- Is this a breaking change? If yes how are migrating the existing Excalidraw users ?

No this is not a breaking change (unless we change some existing implementation) so hopefully no migration will be needed.
However, for virtual text elements some existing code might need to be updated with the way wysiwyg editor is being used and hence that could introduce some changes we need to test out before pushing this out.
