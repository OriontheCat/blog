## GridWorld
For my second project I had to make an intelligent agent that could follow walls. My teacher suggested we use an int matrix to represent the world, but I thought I would take advantage of Java's OOP and make a matrix of an abstract class called a block:
```java
abstract class Block {
    int x;
    int y;
    String pictureFile = "error.png";

    Block(int x, int y, String pictureFile) {
        this.x = x;
        this.y = y;
        this.pictureFile = pictureFile;
    }

    void draw() {
        StdDraw.picture(x * 10 + 5, y * 10 + 5, pictureFile, 10, 10);
    }
}
```
It could draw a picture wherever it was, but it had to have it's x and y properties changed whenever it was moved. Next I extended Block into two classes: Wall and Empty: 
```java
class Wall extends Block {
    Wall(int x, int y) {
        super(x, y, "naman.png");
    }
}

class Empty extends Block {
    Empty(int x, int y) {
        super(x, y, "hex.jpg");
    }
}
```
I used these to build a maze. To store the maze I decided on an int matrix, but those are a pain to handmake, so I made two functions: one to turn a string into an int matrix, and a function to write the int matrix to the Block matrix:
```java
int[][] convertStringToIntMatrix(String intMatrixString) {
        int[][] intMatrix = new int[xDim][yDim];
        String[] tempStringArray = intMatrixString.split("\n");
        String[][] tempStringMatrix = new String[xDim][yDim];
        for (int x = 0; x < xDim; x++) {
            tempStringMatrix[x] = tempStringArray[x].split("");
            for (int y = 0; y < xDim; y++) {
                intMatrix[x][y] = Integer.parseInt(tempStringMatrix[x][y]);
            }
        }
        return intMatrix;
    }

    Block[][] convertIntMatrixToBlockMatrix(int[][] intMatrix) {
        Block[][] blockMatrix = new Block[xDim][yDim];
        for (int x = 0; x < xDim; x++) {
            for (int y = 0; y < xDim; y++) {
                blockMatrix[x][y] = new Empty(x, y);
                if (intMatrix[x][y] == 1) {
                    blockMatrix[x][y] = new Wall(x, y);
                }
                if (intMatrix[x][y] == 2) {
                    blockMatrix[x][y] = new Player(x, y);
                }
                if (intMatrix[x][y] == 3) {
                    blockMatrix[x][y] = new Enemy(x, y);
                }
            }
        }
        return blockMatrix;
    }
```
My teacher pretty much gave us the wall following logic, so all I had to do was turn the logic into code (and I had to use predefined method names):
```java
boolean[] getSurroundingCellsVector() {
        boolean[] surroundingCellsVector = new boolean[8];
        Vector[] coordinates = { new Vector(-1, -1), new Vector(0, -1), new Vector(1, -1), new Vector(1, 0),
                new Vector(1, 1), new Vector(0, 1), new Vector(-1, 1), new Vector(-1, 0) };
        for (int i = 0; i < coordinates.length; i++) {
            try {
                if (Arrays.asList(Empty.class, Player.class).contains(
                        GridWorld.oldMatrix[this.x + coordinates[i].x][this.y + coordinates[i].y].getClass())) {
                    surroundingCellsVector[i] = true;
                }
            } catch (Exception e) {
                System.out.print(e);
            }
        }
        return surroundingCellsVector;
    }

    boolean[] sense() {
        return this.getFeatureVector();
    }

    Vector decideMovement() {
        // boolean[] surroundingCellsVector = getSurroundingCellsVector();
        // boolean[] surrounded = { true, true, true, true, true, true, true, true };
        boolean[] featureVector = sense();
        if (featureVector[0] && !featureVector[1]) {
            return new Vector(1, 0);
            // StdDraw.text(this.x * 10 + (5 + 10), this.y * 10 + 5, "e");
        } else if (featureVector[1] && !featureVector[2]) {
            return new Vector(0, 1);
            // StdDraw.text(this.x * 10 + 5, this.y * 10 + (5 + 10), "s");
        } else if (featureVector[2] && !featureVector[3]) {
            return new Vector(-1, 0);
            // StdDraw.text(this.x * 10 + (5 - 10), this.y * 10 + 5, "w");
        } else if (featureVector[3] && !featureVector[0]) {
            return new Vector(0, -1);
            // StdDraw.text(this.x * 10 + 5, this.y * 10 + (5 - 10), "n");
        }
        // else if (surroundingCellsVector == surrounded) {
        // return new Vector(0, 0);
        // }
        return new Vector(0, -1);
    }

    void move() {
        act();
    }

    Vector decide() {
        return this.decideMovement();
    }

    void act() {
        Vector direction = decide();
        GridWorld.matrix[this.x + direction.x][this.y + direction.y] = new Player(this.x + direction.x,
                this.y + direction.y);
        GridWorld.matrix[this.x][this.y] = new Empty(this.x, this.y);
    }
```
I then wanted to make an Enemy class that moved counterclockwise, so I extended the Player class into a new one with different movement logic. This time I had to actually think about how the logic worked, and it definately took me the longest out of all the code.
```java
 @Override
    void move() {
        Vector direction = decideMovement();
        GridWorld.matrix[this.x + direction.x][this.y + direction.y] = new Enemy(this.x + direction.x,
                this.y + direction.y);
        GridWorld.matrix[this.x][this.y] = new Empty(this.x, this.y);
    }

    @Override
    boolean[] getFeatureVector() {
        boolean[] featureVector = new boolean[4];
        boolean[] surroundingCellsVector = this.getSurroundingCellsVector();
        // StdDraw.setPenColor(StdDraw.BLACK);
        if (!surroundingCellsVector[0] || !surroundingCellsVector[1]) {
            featureVector[0] = true;
            // StdDraw.text(this.x * 10 + (5 + 5), this.y * 10 + (5 - 10), "empty");
        }
        if (!surroundingCellsVector[2] || !surroundingCellsVector[3]) {
            featureVector[1] = true;
            // StdDraw.text(this.x * 10 + (5 + 10), this.y * 10 + (5 + 5), "empty");
        }
        if (!surroundingCellsVector[4] || !surroundingCellsVector[5]) {
            featureVector[2] = true;
            // StdDraw.text(this.x * 10 + (5 - 5), this.y * 10 + (5 + 10), "empty");
        }
        if (!surroundingCellsVector[6] || !surroundingCellsVector[7]) {
            featureVector[3] = true;
            // StdDraw.text(this.x * 10 + (5 - 10), this.y * 10 + (5 - 5), "empty");
        }
        return featureVector;
    }

    @Override
    Vector decideMovement() {
        boolean[] featureVector = getFeatureVector();
        if (featureVector[2] && !featureVector[1]) {
            return new Vector(1, 0);
            // StdDraw.text(this.x * 10 + (5 + 10), this.y * 10 + 5, "e");
        } else if (featureVector[3] && !featureVector[2]) {
            return new Vector(0, 1);
            // StdDraw.text(this.x * 10 + 5, this.y * 10 + (5 + 10), "s");
        } else if (featureVector[0] && !featureVector[3]) {
            return new Vector(-1, 0);
            // StdDraw.text(this.x * 10 + (5 - 10), this.y * 10 + 5, "w");
        } else if (featureVector[1] && !featureVector[0]) {
            return new Vector(0, -1);
            // StdDraw.text(this.x * 10 + 5, this.y * 10 + (5 - 10), "n");
        }
        return new Vector(0, -1);
    }
```
Finally I wanted to go a step further and make a MazeEditor. It was fairly simple to make a mouse detection program, map that to an int array, turn it into a string, and write it to a ".maze" file. I also added a side menu that showed the hotkeys for the different "Blocks" and one that showed the current selected Block.
