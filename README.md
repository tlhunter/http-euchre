# HTTP Euchre

This is a simple project to teach myself Go and Rust.

I think for the first version the server will use simple req/res over HTTP. Future versions will use WebSockets to push the state of the game to players.

## `POST /v1/register`

Register a user.

### Request

```json
{
  "username": "tlhunter"
}
```

### Response

```json
{
  "username": "tlhunter",
  "user_id": "p-01D1Q8T5E8D5XG65N94N1KTWSW",
  "secret": "NA8C6T1SD3WZ7S19"
}
```

The `id` and `secret` attributes will then be used for authorizing future requests. These will be passed into the `Authorization` header. The header value is the word `Basic`, a space, then the result of the function `base64(user_id + ':' + secret)`.

```http
Authorization: Basic MDFEMVE4VDVFOEQ1WEc2NU45NE4xS1RXU1c6TkE4QzZUMVNEM1daN1MxOQ==
```

## `GET /v1/user/{user_id}`

Get information about the user.

```json
{
  "username": "tlhunter",
  "user_id": "p-01D1Q8T5E8D5XG65N94N1KTWSW",
  "state": "LOBBY",
  "game_id": null,
  "hand": []
}
```

A user can be in the following states:

- `IDLE`
- `LOBBY`: waiting to join a game
- `PLAYING`: in a game (`game_id` will be populated)

## `POST /v1/join`

Enter the lobby to join a game. Once you're in a lobby you can't leave until a game starts. Once four people join the lobby a game will start.

Once you're in the lobby you'll need to keep polling `GET /v1/user/{user_id}`. Once the payload returns a state of `PLAYING`, you'll be able to get the `game_id` and use it for the next request.

## `GET /v1/game/{game_id}`

Get information about a game.

### Response

```json
{
  "game_id": "g-01D1QA5ZHXBXSCH315AQKE5YKW",
  "state": "TRUMP",
  "teams": {
    "team1": {
      "score": 3,
      "players": [
        "p-01D1Q8T5E8D5XG65N94N1KTWSW",
        "p-01D1QA7XJJQYXBM8MXN6A75CEC"
      ]
    },
    "team2": {
      "score": 8,
      "players": [
        "p-01D1QA7BGXXQWMM4ECGNK7X3BR",
        "p-01D1QA875XNPA2JA9BJN88WNDG"
      ]
    }
  },
  "players": {
    "current": "p-01D1Q8T5E8D5XG65N94N1KTWSW",
    "list": [
      "p-01D1Q8T5E8D5XG65N94N1KTWSW",
      "p-01D1QA7BGXXQWMM4ECGNK7X3BR",
      "p-01D1QA7XJJQYXBM8MXN6A75CEC",
      "p-01D1QA875XNPA2JA9BJN88WNDG"
    ]
  },
  "round": {
    "number": 1,
    "trump": null,
    "dealer": "p-01D1QA875XNPA2JA9BJN88WNDG",
    "face_up": "9D"
  }
}
```

A game can be in the following states:

- `TRUMP-FACE`: players are deciding if face-up card is trump
- `TRUMP-CHOOSE`: players decide a new trump
- `PLAYING`: game is being played

## `POST /v1/play/{game_id}`

Plays a card for the current game.

### Request

```json
{
  "card": "JD"
}
```

### Response

```json
```

## Card Codes

- `N`: nine
- `T`: ten
- `J`: jack
- `Q`: queen
- `K`: king
- `A`: ace
- `D`: diamond
- `C`: club
- `S`: spade
- `H`: heart

```
ND NC NS NH
TD TC TS TH
JD JC JS JH
QD QC QS QH
KD KC KS KH
AD AC AS AH
```
