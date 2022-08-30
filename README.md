# Notes (SERVER)

- Install dependencies - `composer install`
- Create `app/socket.php`

```php
<?php

namespace App;

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class Socket implements MessageComponentInterface {

    public function __construct()
    {
        $this->clients = new \SplObjectStorage;
    }

    public function onOpen(ConnectionInterface $conn) {

        // Store the new connection in $this->clients
        $this->clients->attach($conn);

        echo "New connection! ({$conn->resourceId})\n";
    }

    public function onMessage(ConnectionInterface $from, $msg) {

        foreach ( $this->clients as $client ) {

            if ( $from->resourceId == $client->resourceId ) {
                continue;
            }

            $client->send( "Client $from->resourceId said $msg" );
        }
    }

    public function onClose(ConnectionInterface $conn) {
    }

    public function onError(ConnectionInterface $conn, \Exception $e) {
    }
}
```

- Create `run` php script

```php
#!/usr/bin/env php

<?php

use Ratchet\Server\IoServer;
use Ratchet\Http\HttpServer;
use Ratchet\WebSocket\WsServer;
use App\Socket;

require dirname( __FILE__ ) . '/vendor/autoload.php';

$server = IoServer::factory(
    new HttpServer(
        new WsServer(
            new Socket()
        )
    ),
    8088 // <--------------------------
);

$server->run();
```

- **NB** Run script using **CLI - Command Prompt** (Security measures)
- `php run`
---


# Notes (CLIENT)

- https://github.com/fadilxcoder/websocket-php-html/tree/client (Branch)
- Serve index.html - `http-server -p 8516 --log-ip`
- URL : http://192.168.100.XX:8516/

```js

// Create a new WebSocket.

//var socket  = new WebSocket('ws://127.0.0.1:8088');
var socket  = new WebSocket("ws://192.168.100.XX:8088");

var message = document.getElementById('message');

function transmitMessage() {
    socket.send( message.value );
}

socket.onmessage = function(e) {
    alert( e.data );
}
```

- HTML layout

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Websocket UI</title>
        <style>
            #chatFormContainer {
                text-align: center;
                position: fixed;
                bottom: 5px;
                left: 5px;
                right: 5px;
            }

            #chatFormContainer input {
                display: inline-block;
                width: 90%;
                padding: 20px;
            }
        </style>
    </head>

    <body>
        <div id="chatLog">

        </div>
        <div id="chatFormContainer">
            <form id="chatForm">
                <input id="chatMessage" placeholder="Type  message and press enter..." type="text">
            </form>
        </div>
        <script>
            var username = `user_${getRandomNumber()}`
            
            //var socket  = new WebSocket('ws://127.0.0.1:8088');
            var socket  = new WebSocket("ws://192.168.100.XX:8088");

            var chatLog = document.getElementById('chatLog');
            var chatForm = document.getElementById('chatForm');
            chatForm.addEventListener("submit", sendMessage);

            socket.onopen = function() {
                console.log(`Websocket connected`);
                socket.send(JSON.stringify({
                    event: 'new_joining',
                    sender: username,
                }));
            }
            socket.onmessage = function(message) {
                console.log(message);
                var payload = JSON.parse(message.data);
                console.log(payload);

                if (payload.sender == username) {
                    payload.sender = "You";
                }

                if (payload.event == "new_message") {

                    //Handle new message
                    chatLog.insertAdjacentHTML('afterend', `<div> ${payload.sender}: ${payload.text} <div>`);

                } else if (payload.event == 'new_joining') {

                    //Handle new joining
                    chatLog.insertAdjacentHTML('afterend', `<div> ${payload.sender} joined the chat<div>`);

                }
            }

            function getRandomNumber() {
                return Math.floor(Math.random() * 1000);
            }

            function sendMessage(e) {
                e.preventDefault();

                var message = document.getElementById('chatMessage').value;

                socket.send(JSON.stringify({
                    event: 'new_message',
                    sender: username,
                    text: message
                }));
            }
        </script>
    </body>
</html>
```
