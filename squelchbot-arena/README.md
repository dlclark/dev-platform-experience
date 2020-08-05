# Squelchbot Arena Spec
The arena is a coordination app that coordinates and adjudicates squelch games between bots.  


## Required Inputs
You are allowed to have other command line parameters or StdIn support for debugging, logging verbosity, or different modes of operation.

### Required command line parameters
 Arg | Description 
-----|------------
 url | The URL of a bot. A repeatable arg, provided once per bot.
 c   | The number of games to play

###  Required StdIn
None


## Outputs
### StdOut
On Startup the header should be output:
```
[p1Name] at [p1]
[p2Name] at [p2]
...
Playing [c] games
```

Every 1 second the progress of the games should be output in the following format:

```
[DateTime] [finishedGameCount] / [c], [p1WinCount] to [p2WinCount] to [p3WinCount] to...
```

### StdErr
If a bot returns an invalid response or times out a message should be output on StdErr.  


## Operation
PseudoCode:
```
Ask each bot for a name, output header (format above).

startPlayerIndex = -1
start the match with each bot

for [c] games (each game may be concurrent):
    start the game with each bot
    increment startBotIndex, wrapping to 0 if overflow
    currentBotIndex = startBotIndex
    until game over:
        [game rules]

    increment bot winner count
    end the game with each bot

end the match with each bot
```

TODO: finish pseudocode

Allow for a 1 second timeout for each API call.