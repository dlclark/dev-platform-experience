# Squelchbot API Spec

Create a REST service that follows the spec below.  Each call MUST be idempotent and return the same result for the same request URL.  

Each squelch bot API MUST support playing multiple matches and multiple games of the same match concurrently.  Each match will be identified by a unique `matchId` and each game within the match will be identified by a unique `gameId`.  A `gameId` is unique within the context of a match and is allowed to repeat across matches.

### Game Rules
Squelch is a version of Farkle.  You can read the base rules and standard scoring method here:
https://en.wikipedia.org/wiki/Farkle

For this version we'll use this scoring table:
Dice Combination | Score
--- | ---
Each 1 | 100
Each 5 | 50
Three 1s | 1000
Three 2s | 200
Three 3s | 300
Three 4s | 400
Three 5s | 500
Three 6s | 600
Three pair | 750
123456 | 1500

The design is such that the Arena tells you the possible point values for each roll so you typically don't need to codify these rules.

## GET /bot/info
Return your bot info. `name` should be limited to 20 characters.

#### Request Body
None

#### Response
`200 OK`
```json
{ "name" : "somebot v1.0" }
```

## PUT /match/[matchId]/start
A match is a group of games with the same players and rules.  This tells you a match is starting and gives you the players and rules. Keep in mind that other games and matches may still be in progress at this point. 

The `dieCount` is typically 6 and `maxPoints` is typically 5000 for normal squelch games.  `yourBotIndex` is your index any time a botIndex is used throughout the game.
The `botNames` given are botIndexed strings and allow you to track which bots you're playing. 


#### Request Body
```json
{ 
    "dieCount" : <int>,
    "maxPoints" : <int>,
    "gameCount" : <int>,
    "yourBotIndex" : <int>,
    "botNames" : [ "name1", "name2" ]
}
```

#### Response
`200 OK`

## PUT /match/[matchId]/end
The match has concluded. Includes stats about the number of wins for each bot.

#### Request Body
```json
{ 
    "winsByBotIndex" : [<int>]
}
```

#### Response
`200 OK`

## PUT /match/[matchId]/game/[gameId]/start

Tells you a new game is starting.  Keep in mind that other games and matches may still be in progress at this point. 

#### Request Body
None

#### Response
`200 OK`

## PUT /match/[matchId]/game/[gameId]/end
After your final turn this is called to let you see the rolls of the players that went after you and to tell you which bot was the winner of the game. `finalPlayerTurns` will have the turn data from the final round of the game, including yours.

#### Request Body
```json
{
    "finalPlayerTurns" : [<BotTurn>],
    "winnerBotIndex" : <int>
}

```

#### Response
`200 OK`


## PUT /match/[matchId]/game/[gameId]/turn/[turnId]/start

Called at the start of your bots turn.  `otherPlayerTurns` gives a view of what the other bots did on their most recent turns and includes their current point totals.  If `isFinalRound` is true then this is the last turn your bot will have this game.


#### Request Body
```json
{ 
    "startPoints" : <int>,
    "otherPlayerTurns": [<BotTurn>],
    "isFinalRound" : <bool>
}
```

#### Response
`200 OK`

## PUT /match/[matchId]/game/[gameId]/turn/[turnId]/choose

After the arena roles the dice the bot must make a selection on which point options to choose.  If no point options were available then `GET /game/[gameId]/turn/squelch` is called instead.

`dieValues` represents the values of the dice rolled as a serialized string. The die values are sorted from lowest value to highest.  All die rolls are represented in this same format throughout the API.

The option `id` is a unique string within the context of the this turn.  It may be anywhere from 1 to 50 characters long.  The `options` array will be given in a sorted order based on the descending point value (highest point options first).  If points are equal between two options then the option with the fewest die will be first.

The bot should select which option it considers "best" and return the `id` and a bool to indicate if it wants to stop and collect the points for its turn (`"stay" : true`) or keep going and roll the remaining dice (`"stay" : false`).

#### Request Body
```json
{
    "dieValues" : "111222",
    "options" : [ 
        {
            "id" : "SADF",
            "dieValues" : "111",
            "points" : 1000
        }, {
            "id" : "QWER",
            "dieValues" : "222",
            "points" : 200
        }, {
            "id" : "ZXCV",
            "dieValues" : "111222",
            "points" : 1200
        }
    ]
}
```

#### Response
`200 OK`
```json
{ 
    "take" : "<optionId>",
    "stay" : <bool>
 }
```

## PUT /match/[matchId]/game/[gameId]/turn/[turnId]/squelch
Called when your turn is over because you didn't roll any points.  This is called instead of `PUT /game/[gameId]/turn/choose`.

#### Request Body
```json
{
    "dieValues" : "22346"
}
```

#### Response
`200 OK`

## IO

### BotTurn
```json
{ 
    "botIndex" : <int>,
    "startPoints" : <int>,
    "endPoints" : <int>,
    "rolls" : [
        {
            "roll" : "111345",
            "take" : "111",
            "points" : 1000
        },
        {
            "roll" : "125",
            "take" : "15",
            "points" : 150
        },
        {
            "roll" : "4",
            "take" : "",
            "points" : 0
        }
    ]
}
```