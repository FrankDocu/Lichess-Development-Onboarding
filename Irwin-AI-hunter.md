# Irwin AI Hunter

![Screenshot of Irwin Report](http://i.imgur.com/IqWdHPj.png)

Irwin analyses games by looking at individual move data, chunks of moves, and all moves played in mass.

## An individual game report

![An individual game report](http://i.imgur.com/QcCpH1q.png)

Each bar on the left represents Irwin's activation for that move. The larger and more red, the more likely cheating occurred. The activation for each move is calculated by 2 parts.

### Individual Move Data
- Move Number
- Rank (the PV rank)
- Winning Chances
- Winning Chances Loss
- Ambiguity (how many good moves are available in the position)
- Time Used (amount of time in seconds used to play the move)
- Move time consistency (is the amount of time spent on the move within 0.5 standard deviations of the average)
This data is sent through a neural network on a move-by-move basis to get an activation between 0 and 100.

### Chunk Move Data
This is similar to the individual move data, but it spans 10 moves. In a game of 30 moves, there will be 21 chunks as they over-lap each other.