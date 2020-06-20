So you're at the gate, wanting to know how to get in. Luckily, you have all this documentation to tell you how to properly connect to Bongo Chat to send and receive messages.  

# Data Structure
| Field   | Type           | Description           |
|---------|----------------|-----------------------|
| code *  | integer        | code for the payload  |
| event   | string         | the name of the event |
| payload | any json value | event data            |  

\* `code` is only sent to the client if the `event` is `invalid` to know which data sent to the server caused it.

## Sending Data
Data sent *should* be valid JSON if parsed, or the server will send an `invalid` event with code `0`.  
They should also have all fields defined in the [data structure](#data-structure) or the server will again respond with `invalid` event and code `1`

## Receiving Data
As a client, receiving data is fairly simple. The server always sends stringified JSON that you should be able to parse.  
As said above, you will not always receive a `code` so don't rely on that for handling events. Instead, just look at `event` for the name.  
`code` is only sent for the `invalid` event.

# Connecting
By default, Bongo Chat servers listen for websocket connections on the port `8080` so if you want to connect you can use the server's IP/domain with port `8080` and the `ws` protocol. Here is an example of a URL to connect: `ws://localhost:8080`  
Once connected, you should send a `welcome` event with an empty `payload`.  

### Example Welcome
```json
{
	"event": "welcome",
	"payload": {}
}
```  

Then the client should automatically receive the `hello` event with a `pretoken`. Servers here can setup some kind of a checkpoint here if they wish.  

### Example Hello
```json
{
	"event": "hello",
	"payload": {
		"pretoken": "20b8e1b54a4f4fb66c6a3bba59b42cbd"
	}
}
```  

## Identifying
Next, the client should `identify`. Here you can choose whether to register or login. If logging in, the client should send the `identify` data with the login data. An example is shown here:  
### Example Identify for Login
```json
{
	"event": "identify",
	"payload": {
		"pretoken": "20b8e1b54a4f4fb66c6a3bba59b42cbd",
		"accountName": "bongocat",
		"password": "bongobongobongo"
	}
}
```  

And for registration:  
```json
{
	"event": "identify",
	"payload": {
		"pretoken": "20b8e1b54a4f4fb66c6a3bba59b42cbd",
		"accountName": "bongocat",
		"username": "Bongo Cat",
		"tag": "cat",
		"password": "bongobongobongo"
	}
}
``` 
Yes, `tag` can be 1-4 alphanumeric characters.

If registered or logged in was successful, you will get a [`ready`](#events-ready) event with the user's data.  

# Events
All events, sent and received, should be camelCase.  

### Welcome
Used to get `hello` from the server, saying you want to initialize identification.  

### Hello
Confirms the server heard your request to be `welcome`d and sends the pretoken.  

#### Hello Structure
| Field    | Type           | Description                    |
|----------|----------------|--------------------------------|
| pretoken | string         | pretoken to use with identify  |

### Identify
Used to login (or register) a user.

#### Identify Structure
| Field       | Type   | Description                                |
|-------------|--------|--------------------------------------------|
| pretoken    | string | pretoken to use with identify              |
| accountName | string | the user's account name                    |
| username?   | string | user's desired username (for registration) |
| tag?        | string | user's desired tag (for registration)      |
| password    | string | the user's account password                |  

### Ready
Sends user information.  

#### Ready Structure
| Field       | Type   | Description                                |
|-------------|--------|--------------------------------------------|
| user        | object | user's information                         |  

#### Ready User Information
Sends user information.
| Field       | Type    | Description                                |
|-------------|---------|--------------------------------------------|
| sessionID   | string  | session id                                 |  
| id          | integer | snowflake id of the user                   |  
| username    | string  | user's  username                           |
| tag         | string  | user's tag                                 |