## About

Python parser script for Texas Hold'em poker dataset taken from [here](https://web.archive.org/web/20110205042259/http://www.outflopped.com/questions/286/obfuscated-datamined-hand-histories)

***

Run with: <br>
```bash
python csvGenerator.py
```

or: <br>
```bash
python csvGenerator.py /absolute/path/to/dataset/folder outputResults.csv y
```
<br>
Where `/absolute/path/to/dataset/folder`  contains all the (unzipped) text data to mine, `outputResults.csv` is the CSV file that gets generated, and `y` (yes) will overwrite the CSV file if it already exists (`n`, no, will not overwrite the file and will prompt for a new name).



[Download the ZIP containing the script.](https://github.com/devedge/PokerDatasetParser/raw/master/data/PokerDatasetParser.zip) This has only been tested with Python >= 3.5.<br><br>


## Setup

The structure of the CSV file (rows and columns) needs to be manually defined in `csvGenerator.py`. None of the other files need to be changed. More information on how to edit the file is defined in the `csvGenerator.py` section below.<br><br>


## Program Logic

The general program logic is split up into three different Python files. The parser itself is `datafileParser.py`, the API for interacting with the parser is `parserAPI.py`, and the front-end that handles interaction with the user and creates the CSV files is `csvGenerator.py`.

<br>
<h4>`csvGenerator.py` [view file](https://github.com/devedge/PokerDatasetParser/blob/master/csvGenerator.py)</h4> <br>
The part of the script that needs to be edited is the first method in `writeCSV()` at the top of the file. 
<br><br>
The parser returns one game at a time (gameArray), and between the comment headers `#### ---- ####` is the code where each value is added to the current row. For each value you want to add to the current row, use `csvrow.append()`. These values are extracted using methods from `parserAPI.py`, which are all defined below in the `parserAPI.py` section.
<br><br>
For example in the code provided below:
 - `parserAPI.getGameID()` returns the current game's ID
 - `parserAPI.getDate()` returns the date of the game
 - `parserAPI.isGameWon()` returns 'true' if the game was won, and 'false' if otherwise
 - `parserAPI.getTotalPot()` returns the game's total pot amount
<br>
```python
# Writes the csv file from information gathered from the gameArray
def writeCSV(gameArray):
    global csvFile

    # Initialize the gameArray in parserAPI
    parserAPI.initGameArray(gameArray)

    # array of values taken from parserAPI.py using the gameArray
    csvrow =  []

    #### ---- ####  Edit this part so the values are appended to the array as a row

    # For each game, store the game ID, date, if the game was won, and what the total pot was
    csvrow.append(parserAPI.getGameID())
    csvrow.append(parserAPI.getDate())
    csvrow.append(parserAPI.isGameWon())
    csvrow.append(parserAPI.getTotalPot())

    #### ---- ####

    # Writes the current row to disk
    csvFile.writerow(csvrow)
```


<br>
The 'csvGenerator.py' script is in charge of getting user input, scanning every directory and subdirectory, passing each file to the datafileParser.py script, and writing the values to a CSV file.

<br><br>
<h4>`parserAPI.py` [view file](https://github.com/devedge/PokerDatasetParser/blob/master/parserAPI.py)</h4> <br>
This script contains easy to use methods for accessing fields from the game arrays that `datafileParser.py` returns.
<br><br>
<b>General Methods:</b>
<br><br>
* `getGameID()` <br>
    Returns the GameID as an Int
* `getDate()` <br>
    Returns the date as a String
* `getTime()` <br>
    Returns the time as a String
* `getTotalPot()` <br>
    Returns the total pot amount as an Int
* `isGameWon()` <br>
    Returns a boolean 'true' if the game was won, and 'false' otherwise
* `isGameCollected()` <br>
    Returns a boolean 'true' if the game was not won, but a user collected the entire pot. Returns 'false' otherwise.
* `isGameLost()` <br>
    Returns a boolean 'true' if the game was lost, and 'false' otherwise
* `isDealerDead()` <br>
    Returns a boolean 'true' if the dealer died (disappeared from the game), and 'false' otherwise
* `isBoardCardsDisplayed()` <br>
    Returns a boolean 'true' if the board cards were displayed, and 'false' otherwise
* `getBoardCards()` <br>
    Returns an array of Strings, each string containing one of the cards on the board
* `getNumberRounds()` <br>
    Returns the number of rounds played as an Int

<br><br><br>
<i>Variables used in the following methods:</i>
* `winnerIndex` <br>
    The game array index of the winner. This is '1' by default in the methods, so it does not have to be specified. If there are several winners though, `getNumberWinners()` can be used to iterate through all the winners (up to two).
* `playerIndex`  <br>
    The index of the player in the array, which is '1' by default in the methods. This can be found using `getLosingPlayerIndex()`, `getWinningPlayerIndex(winnerIndex)`, or by iterating through all the players with `getNumberPlayers()`.
* `play` <br>
    The index of the play in the game array, which is '1' by default in the methods. There are 2 to 6 plays in a poker game, which are "Pre-game actions", "Pocket Cards", "Flop", "Turn", "River", and "Showdown". The total number of plays in a game can be found with `getNumberRounds()`.
* `action` <br>
    The action of a user during a play. This defaults to '1' in the methods. The number of actions a user has taken during a play can be found with `getNumActions(playerIndex, play)`.
* `cardIDX` <br>
    The index of the cards played, and defaults to returning an array of Strings. The number of cards is always 5.

<br><br>
<b>General Player Info Methods:</b>
<br><br>
* `getNumberPlayers()` <br>
    Returns the number of players as an Int
* `getPlayerID(playerIndex)` <br>
    Returns the ID of the player in the game from their index. A player ID looks like `X+u4T/E5ANkyZLKm1YjqwQ`.
* `getPlayerChipsAmount(playerIndex)` <br>
    Returns the player's dollar amount in chips as a decimal amount (float).
* `getOnePlayerAction(playerIndex, play, action)` <br>
    Returns one player's action, given the player's index, the play index, and the action index at that play. This method is usually used by other methods defined in this API, instead of being called directly by the user.
* `getNumActions(playerIndex, play)` <br>
    Returns the number of actions a player has taken during a play in the game.
* `getAllPlayerActions(playerIndex, play)` <br>
    Returns all of a player's actions during a play as an array of Strings. There is usually only 1 action, but occasionally there are 2.
* `getUserActionCount(playerIndex)` <br>
    Returns an array of Integers that contains a count of the number of actions a player has taken during the game. The Integer values in the array represent: `[calls, bets, raises, all-in, shows, check, fold, muck, sitout]`

<br><br>
<b>Winner Specific Methods:</b>
<br><br>
* `getNumberWinners()` <br>
    Returns the number of winners as an Int. There is usually 1 winner, but occasionally there are 2.
* `getWinningPlayerIndex(winnerIndex)` <br>
    Returns the player index from the winnerIndex in the gameArray. The winnerIndex can be iterated over using `getNumberWinners()`
* `getWinningPlayerAmount(winnerIndex)` <br>
    Returns the amount of money the winner at the index has won.
* `getWinningHand(cardIDX, winnerIndex)` <br>
    Returns the winning hand of the winner at the winnerIndex as an array if the cardIDX is 0, and as a String if a card is specified.
* `getWinningCardsOnTable(cardIDX, winnerIndex)` <br>
    Returns the winning cards as an array of Strings on the table if cardIDX is 0, or the actual card String if an index of 1 to 5 is specified.
* `getWinningHandDescription(winnerIndex)` <br>
    Returns the winning hand description as a String.

<br><br>
<b>Loser Specific Methods:</b>
<br>
* `getLosingPlayerIndex()` <br>
    Returns the losing player's index.
* `getLosingHand(cardIDX)` <br>
    Returns the losing player's hand as an array of two Strings if no index is specified. Otherwise, if the cardIDX is 1 or 2 it returns the specified card as a String.
* `getLosingCardsOnTable(cardIDX)` <br>
    Returns the loser's cards as an array of Strings on the table if cardIDX is 0, or the actual card String if an index of 1 to 5 is specified.
* `getLosingHandDescription()` <br>
    Returns the losing hand description as a String.

<br><br>
<h4>`datafileParser.py` [view file](https://github.com/devedge/PokerDatasetParser/blob/master/datafileParser.py)</h4> <br>
This script parses a poker game text file. It saves each game in the file as a gameArray, and saves all of the games in one long gamesList, which is returned to `csvGenerator()`. [An example gameArray](https://github.com/devedge/PokerDatasetParser/blob/master/data/example%20gameArray.txt)

