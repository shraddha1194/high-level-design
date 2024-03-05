# Design Uber

Steps 1: 100 Million places marked as grid cells,

If grid cells of equal size is created some grid will have higher density while some may be very spare.

**Flexible Grid Sizes:**

Assume the entire world is only one single cell.

Do you have more than 100 cells in this cell?
Yes - divide it in 4 equal cells of equal size
Then recursively divide each cell as needed.

At some part of world we will have very tiny cells and at some parts we will cells that stop dividing at very early stage.

>**Why the magic number 100?**
>
>The number should be big and small enough that we are able to perform the search anf filter like barber shops near me

All the locations are going to be stored on the leaf nodes.


|Place Id|Latitude|Longitude|Description|Title|lead_cell_id|
|--------|--------|---------|-----------|-----|------------|

### Finding grid Id for a particular point - which leaf cell is x,y point in quad tree?



