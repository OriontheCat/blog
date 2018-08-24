##Conway's Game of Life

For my first Honor's AI project, I had to create my own copy of Conway's Game of Life. I used Java and [StdDraw](https://introcs.cs.princeton.edu/java/stdlib/javadoc/StdDraw.html) only because I thought it was required to stick to Java. If I had realized this wasn't the case, I probably would have used [P5.js](https://p5js.org/) or [DartLang](https://dartlang.org/)'s [Flutter](https://flutter.io/). First I wanted to get the graphics done because it would be hard to test the logic with a printout of the boolean matrix. I created three methods for drawing the game: `drawGrid()`, `drawCell()`, and `drawAllCells()`. My grid drawing method simply drew a bunch of rectangles based on some variables:
```java
void drawGrid() {
    StdDraw.setPenColor(StdDraw.BLACK);
    StdDraw.setPenRadius(0.002);
    for (int x = 0; x <= xDim; x++) {
        for (int y = 0; y < yDim; y++) {
            StdDraw.rectangle(x * 10 + 5, y * 10 + 5, 5, 5);
        }
    }
    StdDraw.show();
}
```
Then I made a `drawCell()` method to draw a single cell. It would draw a different color based on a boolean input.
```java
void drawCell(int x, int y, boolean state) {
    if (state) {
        StdDraw.setPenColor(aliveColor);
    } else {
        StdDraw.setPenColor(deadColor);
    }
    StdDraw.filledRectangle(x * 10 + 5, y * 10 + 5, 4.5, 4.5);
    // Code for displaying surrounding alive cells
    // StdDraw.setPenColor(StdDraw.BLACK);
    // StdDraw.text(x * xDim + xDim / 2, y * yDim + yDim / 2,
    // Integer.toString(getSurroundingCellsSum(x, y)));
}
```
Then I made a method to loop over a boolean matrix and run the `drawCell()` method for each value:
```java
void drawAllCells() {
    for (int x = 0; x < xDim; x++) {
        for (int y = 0; y < yDim; y++) {
            drawCell(x, y, cellMatrix[x][y]);
        }
    }
    StdDraw.show();
}
```
Finally I implemented the logic by creating 4 methods: `doCellLogic()`, `doAllCellLogic()`, `getSurroundingCells()`, and `copyBooleanMatrix()` (damn refrences.) First, I copied the current game state to `oldCellMatrix` and cleared the `cellMatrix`. Then `doCellLogic()` took the old cells and decided it's new state using CGoL's rules:
```java
void doCellLogic(int x, int y) {
        int surroundingCells = getSurroundingCellsSum(x, y);
        if (oldCellMatrix[x][y] == true && surroundingCells < 2) {
            cellMatrix[x][y] = false;
        } else if (oldCellMatrix[x][y] == true && (surroundingCells == 2 || surroundingCells == 3)) {
            cellMatrix[x][y] = true;
        } else if (oldCellMatrix[x][y] == true && surroundingCells > 3) {
            cellMatrix[x][y] = false;
        } else if (oldCellMatrix[x][y] == false && surroundingCells == 3) {
            cellMatrix[x][y] = true;
        }
    }   
```
The second method `doAllCellLogic()` ran `doCellLogic()` for all the cells (duh):
```java
void doAllCellLogic() {
    oldCellMatrix = copyBooleanMatrix(cellMatrix);
    for (int x = 0; x < xDim; x++) {
        for (int y = 0; y < yDim; y++) {
            doCellLogic(x, y);
        }
    }
}
```
I had the most trouble with `getSurroundingCellsSum()`. It wasn't that I didn't know how to implement it, I just forgot to write an important piece of code. I always solve "get surrounding values" problems by using nested for loops with -1 to 1 and add them to the desired coordinates, but I forgot to make it not count the center value. This completely changed the game's logic, causing blinkers and gliders to fail. After rewriting `doCellLogic()` two times, I discovered my error and was very sad that I wasted 10 minutes rewriting boolean logic.
```java
int getSurroundingCellsSum(int x, int y) {
        int sum = 0;
        for (int i = -1; i <= 1; i++) {
            for (int j = -1; j <= 1; j++) {
                //I forgot to include this
                if (i != 0 || j != 0) {
                    try {
                        if (oldCellMatrix[x + i][y + j] == true) {
                            sum++;
                        }
                    } catch (Exception e) {
                    }
                }
            }
        }
        return sum;
    }
```
After using StdDraw's proprietary mouse and keyboard methods I was done. I feel this project was a great chance for me to divide my code into small functions.
