![](assets/liteplay_logo.png)


<center> <font size="10">Integration API Specification   </font> </center>

</br>

<center>Version 1.0.0</center>

</br>

<center>Latest Update: March, 2025</center>





---

# Revision History

| Version | Date          | Name  | Description      |
| ------- | ------------- | ----- | ---------------- |
| 1.0.0   | 08 March 2025 | Sally | Initial document |



---


# API Overview

This document describes set of APIs which Operator needs to implement to integrate with TransferWallet API. <br>

The TransferWallet integration allows you to add LitePlay Provider games to your platform by deposit and withdraw credit into our System. You may optionally provide game configuration / parameters to LitePlay Provider.

Provider API and Operator Integration API uses signature to validate incoming/outgoing request integrity. The hash encryption algorithm is described in section [Hash Authenticate](#Hash-Authentication) <br>

Some APIs are required to be **idempotent** which means making multiple identical requests has the same effect as making a single request. <br>

* APIs Timeout is 10 seconds.
* You can check your API settings in LitePlay Provider Back Office
  * Whitelist IPs: IPs to access LitePlay Provider Back Office
  * API Endpoint Whitelist IP: IP that can call [Private APIs](#Private-APIs)
* Data Feeds: APIs on the LitePlay Provider side provides a set of data feeds for:
  * [Play Sessions](#Player-History)
* Player ID(username parameter) is a unique identifier of the user within the Operator system. Before sending to LitePlay Provider any gaming related request the Operator should authenticate a player using the Authenticate method. If a player is new and its account does not exist in the LitePlay Provider system it will be created automatically on the base of the data sent by the Operator server in the Authenticate response. If a player account already exists in the LitePlay Provider database it will be updated with the response data if necessary. Player ID received in the Authenticate response will be sent with all subsequent requests to the Operator.
* Play Session: a game round in which bets and wins are combined together. Each round can contain several bets, wins and refunds of the bets. 
* Transaction Reference: a unique transaction id within your Operator System.

# Private APIs

* Operator will call these APIs, (Operator -> LitePlay Provider)
  * Therefore, Operator will give IP Address to LitePlay Provider to whitelist (this is API Whitelist IP in LitePlay Provider Back Office)
* All APIs are in JSON format (`application/json`), POST method, and in HTTPS
* Authentication: please refer to [Hash Authentication](#Hash-Authentication)
* Fields will be in underscore_
* Special data type (apply to request/response data):
  * `Integer`: this will use normal number data type from JSON (no quoted)
    For example:`{"counter": 12}`
  * `String`: use string data type from JSON (quoted)
    For example:`{"name": "abc1234"}`
  * `Decimal`: for money, rate fields, this use string data type from JSON (quoted)
    For example: `{"credit": "2134.34"}`
  * `DateTime`: use string data type from JSON (quoted).
    Format will be `DD/MM/YYYY HH:mm:ssZ`.
    If no `Z` (timezone) specific, the default `+0000` (UTC) will be used.
    For example: `{"from": "15/01/2006 15:04:03+0700"}`
* `username` field:
  * Identifier of the user within the Operator’s system <br>Parameter value is case-sensitive
  * Example: bet88_anh, 781362, playerABC, playerAbc (playerABC and playerAbc – are two different player accounts within LitePlay Provider system)
* `err` field:
  * No `err` field/Empty `err` field means success
  * For list of error code, please see [Error Code List](#Error-Code-List)
  * Some error will have additional data in `data` field, please also refer to [Error Code List](#Error-Code-List) for example



## Game List

The Operator can use this API to get a list of all available games.

Endpoint: `/games`

Method: `POST`

**Response Parameters**

| Parameter | Data Type | Description              |
| --------- | --------- | ------------------------ |
| games     | Array     | Array of **Game Object** |
| err       | String    | error code               |

**Game Object**

| Parameter | Data Type | Description                        |
| --------- | --------- | ---------------------------------- |
| game_code | String    | Game identifier for each game      |
| game_name | String    | Game name (in EN)                  |
| game_type | String    | Game type e.g. Video Slot, Fishing |
| enable    | Boolean   | `true` = enable, `false` = disable |

*Example of JSON response:*

```json
{
    "games": [
        {
            "game_code": "vseldorado",
            "game_type": "Video Slot",
            "game_name": "Eldorado",
            "enable": true
        },
        {
            "game_code": "vsoceanblue",
            "game_type": "Video Slot",
            "game_name": "Ocean Blue",
            "enable": false
        }
    ],
    "err": ""
}
```

## Get Game URL

The Operator can use this API to get game link for a player to play game

Endpoint: `/game_url`

Method: `POST`

**Request Parameters**

| Parameter   | Data Type | Mandatory | Description                                                  |
| ----------- | --------- | --------- | ------------------------------------------------------------ |
| token       | String    | Required  | secure token generated by operator for the player            |
| game_code   | String    | Required  | unique identifier for the game, e.g. vseldorado, vswanted, vsdogo, etc. |
| platform    | String    | Optional  | `web`: for desktop devices<br>`mobile`: for mobile devices (mobile web browser), default<br>`app`: for APK (mobile app) |
| language    | String    | Optional  | player language in ISO 639-1 standard (e.g. en, fr, it), default to `en`  . For list of languages, please see  [Appendix A](#Appendix-A-Languages) |
| lobby_url   | String    | Optional  | an URL for opening the Operator’s website Lobby page         |
| history_url | String    | Optional  | an URL to open game history page on Operator’s side          |

**Response Parameters**

| Parameter | Data Type | Description |
| --------- | --------- | ----------- |
| url       | String    | Game URL    |
| err       | String    | error code  |

*Example of JSON request:*

```json
{
    "token": "xjneroshp458djk34",
    "game_code": "vseldorado",
    "platform": "web",
    "language": "en",
}
```

*Example of JSON response:*

```json
{
    "url": "https://<game_domain>?token=xjneroshp458djk34&game_code=vseldorado&platform=web&language=en"
}
```



## Deposit

The Operator can use this API to deposit credit to game provider

Endpoint: `/deposit`

Method: `POST`

**Request Parameters**

| Parameter   | Data Type | Mandatory | Description                          |
| ----------- | --------- | --------- |--------------------------------------|
| username       | String    | Required  | Member username                      |
| amount   | Decimal    | Required  | Credit amount, positive. ex: 1000000 |
| currency_code    | String    | Required  | Currency code. ex: IDR               |
| ip_address | String    | Required  | Current IP of player device          |
| reference | String    | Required  | a unique transaction id                               |

**Success Response Parameters**

| Parameter | Data Type | Description    |
| --------- |-----------|----------------|
| balance       | Decimal   | Player Balance |

**Fail Response Parameters**

| Parameter | Data Type | Description  |
|-----------|-----------|--------------|
| err       | String    | Error code   |
| data      | json      | Error detail |

**Error Code**

| Error Code                | Description                                          |
|---------------------------|------------------------------------------------------|
| err:invalid_ip_address    | IP Address invalid                                   |
| err:invalid_currency_code | Currency code invalid                                |
| err:maintenance           | System in maintenance                                |
| err:player_not_found      | Player not found                                     |
| err:invalid_amount        | Amount invalid. Amount must positive number decimal. |
| err:invalid_reference     | Reference invalid.                                   |
| err:duplicate_reference   | Reference deplicate.                                 |

*Example of JSON request:*

```json
{
    "username": "member001",
    "amount": 1000000,
    "currency_code": "IDR",
    "ip_address": "167.1.1.2",
}
```

*Example of JSON response:*

```json
{
    "balance": 1000000
}
```

*Example of FAIL JSON response:*

```json
{
  "err": "err:maintenance",
  "data": {
    "from_time": "06/03/2025 11:00:53+0000",
    "to_time": "07/03/2025 17:00:53+0000"
  }
}
```

## Withdraw

The Operator can use this API to withdraw credit from game provider

Endpoint: `/withdraw`

Method: `POST`

**Request Parameters**

| Parameter   | Data Type | Mandatory | Description                         |
| ----------- | --------- | --------- |-------------------------------------|
| username       | String    | Required  | Member username                     |
| amount   | Decimal    | Required  | Credit amount, positive. ex: 600000 |
| currency_code    | String    | Required  | Currency code. ex: IDR              |
| ip_address | String    | Required  | Current IP of player device         |
| reference | String    | Required  | a unique transaction id                               |

**Success Response Parameters**

| Parameter | Data Type | Description    |
| --------- |-----------|----------------|
| balance       | Decimal   | Player Balance |

**Fail Response Parameters**

| Parameter | Data Type | Description |
|-----------|-----------|-------|
| err       | String    | Error code |
| data      | json      | Error detail |

**Error Code**

| Error Code          | Description                                          |
| ------------------- |------------------------------------------------------|
| err:invalid_ip_address | IP Address invalid                                   |
| err:invalid_currency_code | Currency code invalid                                |
| err:maintenance | System in maintenance                                |
| err:player_not_found | Player not found                                     |
| err:invalid_amount | Amount invalid. Amount must positive number decimal. |
| err:invalid_reference     | Reference invalid.                                   |
| err:duplicate_reference   | Reference deplicate.                                 |

*Example of JSON request:*

```json
{
    "username": "member001",
    "amount": 600000,
    "currency_code": "IDR",
    "ip_address": "167.1.1.2",
}
```

*Example of JSON response:*

```json
{
    "balance": 400000
}
```

*Example of FAIL JSON response:*

```json
{
  "err": "err:maintenance",
  "data": {
    "from_time": "06/03/2025 11:00:53+0000",
    "to_time": "07/03/2025 17:00:53+0000"
  }
}
```

## Balance

The Operator can use this API to check balance in game provider

Endpoint: `/balance`

Method: `POST`

**Request Parameters**

| Parameter   | Data Type | Mandatory | Description                         |
| ----------- | --------- | --------- |-------------------------------------|
| username       | String    | Required  | Member username                     |
| currency_code    | String    | Required  | Currency code. ex: IDR              |
| ip_address | String    | Required  | Current IP of player device         |

**Success Response Parameters**

| Parameter | Data Type | Description    |
| --------- |-----------|----------------|
| balance       | Decimal   | Player Balance |

**Fail Response Parameters**

| Parameter | Data Type | Description |
|-----------|-----------|-------|
| err       | String    | Error code |
| data      | json      | Error detail |

**Error Code**

| Error Code          | Description                                          |
| ------------------- |------------------------------------------------------|
| err:invalid_ip_address | IP Address invalid                                   |
| err:invalid_currency_code | Currency code invalid                                |
| err:maintenance | System in maintenance                                |
| err:player_not_found | Player not found                                     |

*Example of JSON request:*

```json
{
    "username": "member001",
    "currency_code": "IDR",
    "ip_address": "167.1.1.2",
}
```

*Example of JSON response:*

```json
{
    "balance": 400000
}
```

*Example of FAIL JSON response:*

```json
{
  "err": "err:maintenance",
  "data": {
    "from_time": "06/03/2025 11:00:53+0000",
    "to_time": "07/03/2025 17:00:53+0000"
  }
}
```

## Player History

The Operator can use this API to get a list of the game rounds played by the player during the certain day and (optionally) the specific hour.

Endpoint: `/player_history`

Method: `POST`

**Request Parameters**

| Parameter | Data Type | Mandatory | Description                                          |
| --------- | --------- | --------- | ---------------------------------------------------- |
| username  | String    | Required  | Identifier of the user within the Operator’s system. |
| from      | DateTime  | Required  | From time                                            |
| to        | DateTime  | Required  | To time                                              |
| page      | Integer   | Optional  | The page index (default: 1)                          |
| page_size | Integer   | Optional  | Number of item per page (default: 100)               |

Note:

* To query this, we will use `from` <= `round_date` <= `to`
* Therefore to query for example a day, you will use:

```json
{
    "from": "02/07/2021 00:00:00+0700",
    "to": "02/07/2021 23:59:59+0700"
}
```

* To query for an hour, you will use:

```json
{
    "from": "02/07/2021 00:00:00+0700",
    "to": "02/07/2021 00:59:59+0700"
}
```

**Response Parameters**

| Parameter  | Data Type | Description             |
| ---------- | --------- | ----------------------- |
| total_size | Integer   | Total item              |
| list       | Array     | Array of **Game Round** |
| err        | String    | error code              |

**Game Round**

| Parameter     | Data Type | Description                                                  |
| ------------- | --------- | ------------------------------------------------------------ |
| round_id      | String    | Round id                                                     |
| end_date      | DateTime  | Round date                                                   |
| start_date    | DateTime  | Start round date (bet time)                                  |
| balance_after | Decimal   | Balance of player after game round                           |
| bet           | Decimal   | Bet amount                                                   |
| win           | Decimal   | Win amount                                                   |
| status        | String    | `freespin`: a freespin round<br>`refund`:a round with refunded bet<br>`end`: a complete round |
| game_code     | String    | Game code                                                    |
| game_name     | String    | Game name                                                    |

**Error Code**

| Error Code                   | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| err:player_not_found         | player with this `username` does not exist                   |
| err:invalid_date_time_format | invalid `from`/`to` datetime format (correct format: `DD/MM/YYYY HH:mm:ssZ`, for example `20/07/2021 09:20:36+0000`) |
| err:json_error               | JSON data error. There will be a `data` field with more detail |



*Example of JSON resquest:*

```json
{
    "username": "ajinomoto",
    "from": "02/07/2021 00:00:00+0700",
    "to": "02/08/2021 23:59:59+0700",
    "page": 1,
    "page_size": 3
}
```

*Example of JSON response:*

```json
{
    "total_size": 59,
    "list": [
        {
            "round_id": "BZw1EcaChwcfpZ4",
            "end_date": "20/07/2021 09:20:36+0000",
            "start_date": "20/07/2021 09:20:36+0000",
            "balance_after": "20582412.7238",
            "bet": "100",
            "win": "200",
            "status": "end",
            "game_code": "vseldorado",
            "game_name": "Eldorado"
        },
        {
            "round_id": "BegEvLBgWikUyEQ",
            "end_date": "20/07/2021 09:20:36+0000",
            "start_date": "20/07/2021 09:20:36+0000",
            "balance_after": "20582312.7238",
            "bet": "100",
            "win": "250",
            "status": "end",
            "game_code": "vseldorado",
            "game_name": "Eldorado"
        },
        {
            "round_id": "MGSN5zLrZzNeT6p",
            "end_date": "20/07/2021 09:20:35+0000",
            "start_date": "20/07/2021 09:20:35+0000",
            "balance_after": "20582162.7238",
            "bet": "100",
            "win": "250",
            "status": "end",
            "game_code": "vseldorado",
            "game_name": "Eldorado"
        }
    ],
    "err": ""
}
```

## Game Round Detail URL

The Operator can use this API to get a url that when open will show the game round detail of that `round_id`

Endpoint: `/round_detail`

Method: `POST`

**Request Parameters**

| Parameter | Data Type | Mandatory | Description    |
| --------- | --------- | --------- | -------------- |
| round_id  | String    | Required  | the `round_id` |

**Response Parameters**

| Parameter | Data Type | Description                       |
| --------- | --------- | --------------------------------- |
| url       | String    | The URL to open game round detail |
| err       | String    | error code                        |

**Error Code**

| Error Code          | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| err:round_not_found | Game round with this `round_id` does not exist               |
| err:json_error      | JSON data error. There will be a `data` field with more detail |



*Example of JSON resquest:*

```json
{
    "round_id": "MGSN5zLrZzNeT6p"
}
```

*Example of JSON response:*

```json
{
    "url": "https://ajinomoto.vip/public/round_detail/MGSN5zLrZzNeT6p",
    "err": ""
}
```

**Note: please don't just replace the round_id in this URL for different game round detail because we can change the base URL in the future**

## Logout

Logout a player. Should not be required for Transferwallet. Additionally, players can be logged out after a configured session timeout period (set in the Back Office).

**Note**:

* This will logout player from game client, but due to auto reconnect (or player refreshes the game client), an `/auth` request will be sent from LitePlay Provider to Operator, and if this request success, player will be logged in again.
* Therefore, Operator should clear the token before calling `logout`, so for the next `/auth` request, the Operator will response `err:token_not_found` and player cannot log in again.

Endpoint: `/logout`

Method: `POST`

**Request Parameters**

| Parameter | Data Type | Mandatory | Description                                          |
| --------- | --------- | --------- | ---------------------------------------------------- |
| username  | String    | Required  | Identifier of the user within the Operator’s system. |

**Error Code**

| Error Code           | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| err:player_not_found | player with this `username` does not exist                   |
| err:json_error       | JSON data error. There will be a `data` field with more detail |

*Example of JSON request:*

```json
{
 "username": "slot77_john"
}
```

*Example of JSON response:*

```json
{
 "err": ""
}
```


## Logout All Players

To logout all logged in players.

**Note**:

* Normally, this API is used during Operator's maintenance, to make sure no player is playing, to prevent receiving `/bet`, `/result`, `/refund` requests during maintenance.
* This will logout player from game client, but due to auto reconnect (or player refreshes the game client), an `/auth` request will be sent from LitePlay Provider to Operator, and if this request success, player will be logged in again.
* If Operator turns on maintenance, then calling `logout`, no more step will need to be taken because `/auth` request from LitePlay Provider to Operator will just fail.


Endpoint: `/logout/all`

Method: `POST`



# Hash Authentication

## Private APIs

To access LitePlay Provider APIs, you must create a base string firstly.
The base string is constructed by concatenating (separate by `|`)

* The HTTP method (`POST`)
* The path of the url (the endpoint, for example for request to api.google.com/api/v3/login, the path is `/api/v3/login`)
* The timestamp in unix timestamp
* The body of the request (request body always in JSON, so this is the JSON string. If you sent formatted JSON then this should be formatted too)
  Then use the base string as the text and the secret key (issued by LitePlay Provider) as the key to generate the signature using HMAC.SHA256 to encrypt.

*Example in Go:*

```go
baseString := strings.Join([]string{httpMethod, path, timestamp, body}, "|")
mac := hmac.New(sha256.New, []byte(SECRET_KEY))
mac.Write([]byte(baseString))
resultHMAC := mac.Sum(nil)
signature := hex.EncodeToString(resultHMAC)
```

*Example in JavaScript:*

```javascript
var requestData = [httpMethod, requestPath, timestamp, requestBody].join("|");
var signature = CryptoJS.HmacSHA256(requestData, SECRET_KEY).toString();
```

*Example in PHP:*

```php
$requestData = join("|", array($httpMethod, $path, $timestamp, $body));
$signature = hash_hmac('sha256', $requestData, $secretKey);
```

When you have the signature, put into request header these params:<br>

```
signature: <signature>
apikey: <API_KEY> // issued by LitePlay Provider
timestamp: <timestamp> // in unix timestamp like the one we used to create the base string
```

Now you can call Private APIs.

# Error Code List

| Error Code                     | Has `data` field | Description                                                  |
| ------------------------------ | ---------------- | ------------------------------------------------------------ |
| err:invalid_signature          | no               | Mismatch signature in `HTTP HEADER` with signature generated from request. There will be a `data` field with more detail |
| err:invalid_api_key            | no               | Invalid API key                                              |
| err:invalid_date_time_format   | no               | invalid `from`/`to` datetime format (correct format: `DD/MM/YYYY HH:mm:ssZ`, for example `20/07/2021 09:20:36+0000`) |
| err:token_not_found            | no               | No player is associated with this `token`                    |
| err:invalid_language           | no               | Language is not supported                                    |
| err:not_enough_balance         | no               | Not enough balance for bet                                   |
| err:already_refund_transaction | no               | This bet has already been refunded                           |
| err:bet_not_allow              | no               | This bet is not allowed in Operator side                     |
| err:player_not_found           | no               | player with this `username` does not exist                   |
| err:json_error                 | yes              | JSON data error. There will be a `data` field with more detail |
| err:player_not_allow           | no               | Player is not allowed to play                                |



*Example of `err:json_error` response:*

```json
{
    "data": {
        "username": "want string, got number"
    },
    "err": "err:json_error",
}
```


# Reports

# Diagrams

## Opening game flow diagram

![](assets/1.drawio.png)

**Flow**:

1. Player selects a game (is served by LitePlay Provider). 
2. Operator generate a token and game URL, then return the game URL to player browser. 
3. The game URL is loading in player browser for redirecting player to LitePlay Provider game server.
4. Provider game site shows the player can play the game.

# Appendix A: Languages

| Code | Language   |
| ---- | ---------- |
| en   | English    |
| id   | Indonesian |
| th   | Thai       |

# Appendix B: Currencies

| Code | Currency                  |
| ---- | ------------------------- |
| IDR  | Indonesia Rupiah (1:1)    |
| IDR1 | Indonesia Rupiah (1:1000) |
| THB  | Thai Baht                 |
| USD  | United States Dollar      |