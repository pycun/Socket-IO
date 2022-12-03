# Socket IO

## Como funciona SocketIo

Es una biblioteca basada en eventos para aplicaciones web en tiempo real. Permite la comunicación bidireccional en tiempo real entre clientes web y servidores. 

Tiene dos partes: una biblioteca del lado del cliente que se ejecuta en el navegador y una biblioteca del lado del servidor

[image]("https://socket.io/images/bidirectional-communication2.png")

Se basa en el protocolo WebSocket y proporciona garantías adicionales, como el respaldo a long-polling  HTTP o la reconexión automática.

## ¿Que no es SocketIo

Socket.IO **NO** es una implementación de WebSocket.

Aunque Socket.IO de hecho usa WebSocket para el transporte cuando es posible, agrega metadatos adicionales a cada paquete. Es por eso que un cliente WebSocket no podrá conectarse con éxito a un servidor Socket.IO, y un cliente Socket.IO tampoco podrá conectarse a un servidor WebSocket simple.

    // WARNING: the client will NOT be able to connect!
    const socket = io("ws://echo.websocket.org");


## Características

### HTTP long-polling fallback

La conexión recurrirá al long-polling en caso de que no se pueda establecer la conexión WebSocket.

[image]

### Reconexion Automatica

Socket.IO incluye un mecanismo de latido, que comprueba periódicamente el estado de la conexión.

[image]

### Acknowledgements

Facilidad de terminar y recibir un evento con tiempo de espera.

### Broadcasting (Radiodifusión)

Del lado del servidor, puede enviar un evento a todos los clientes conectados o a un subconjunto de clientes:

	// to all connected clients
	io.emit("hello");

	// to all connected clients in the "news" room
	io.to("news").emit("hello");


### Multiplexación

Los espacios de nombres le permiten dividir la lógica de su aplicación en una única conexión compartida. Esto puede ser útil, por ejemplo, si desea crear un canal "administrador" al que solo puedan unirse los usuarios autorizados

	io.on("connection", (socket) => {
	  // classic users
	});

	io.of("/admin").on("connection", (socket) => {
	  // admin users
	});
---

## Instalar Node en el servidor

> Fuente: https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04

1.- **Ejecutamos los siguientes comandos**

    sudo apt update
    sudo apt install nodejs

**Una vez instalado podemos confirmar la version de Node (socketio funciona con node > 10):**

    node -v

**Ahora debemos instalar el administrador de paquetes:**

    sudo apt install npm

**Clonamos el proyecto:**

    git clone https://github.com/socketio/chat-example.git

**Entramos al repositorio e instalamos las dependencias del archivo packet.json**

    cd chat-example/
    npm install

**Esto instalara los siguientes dependencias:**

    "dependencies": {
        "express": "^4.17.1",
        "socket.io": "^4.1.2"
    }

**Express nos servira para levantar un servidor de prueba. Socket.io es lo que probaremos.**

## Poniendo a prueba SocketIo

> Info: https://socket.io/docs/v4/emitting-events/

> example1.js

```js

// Levantar un servidor http
const app = require('express')();
const http = require('http').Server(app);
const port = process.env.PORT || 3000;

// Implementacion de Socketio en el lado del Servidor con NodeJS
const io = require('socket.io')(http);

// Servir archivos
app.get('/', (req, res) => {
    res.sendFile(__dirname + '/example1.html');
});

// Levantamos una conexion que crea el objeto socket
io.on('connection', (socket) => {
    // Le decimos al socket que siempre este a la escucha del evento "chat message"
    socket.on('chat message', msg => {
        // Al recibir un mensaje a traves del evento, reenvialo
        io.emit('chat message', msg);
    });
});

http.listen(port, () => {
    console.log(`Socket.IO server running at http://localhost:${port}/`);
});
```

> example1.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      body { margin: 0; padding-bottom: 3rem; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }

      #form { background: rgba(0, 0, 0, 0.15); padding: 0.25rem; position: fixed; bottom: 0; left: 0; right: 0; display: flex; height: 3rem; box-sizing: border-box; backdrop-filter: blur(10px); }
      #input { border: none; padding: 0 1rem; flex-grow: 1; border-radius: 2rem; margin: 0.25rem; }
      #input:focus { outline: none; }
      #form > button { background: #333; border: none; padding: 0 1rem; margin: 0.25rem; border-radius: 3px; outline: none; color: #fff; }

      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages > li { padding: 0.5rem 1rem; }
      #messages > li:nth-child(odd) { background: #efefef; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form id="form" action="">
      <input id="input" autocomplete="off" /><button>Send</button>
    </form>
    <!-- Implementacion de SocketIo del lado del Cliente -->
    <script src="/socket.io/socket.io.js"></script>
    <!-- Tambien puede instanciarse de la siguiente forma -->
    <!-- <script src="https://cdn.socket.io/4.5.3/socket.io.min.js" integrity="sha384-WPFUvHkB1aHA5TDSZi6xtDgkF0wXJcIIxXhC6h8OT8EH3fC5PWro5pWJ1THjcfEi" crossorigin="anonymous"></script> -->
    
    <script>
      // Se establece un socket en la conexion global
      var socket = io();

      var messages = document.getElementById('messages');
      var form = document.getElementById('form');
      var input = document.getElementById('input');

      form.addEventListener('submit', function(e) {
        e.preventDefault();
        if (input.value) {
          // Cuando se ejecuta un submit del formulario, emite un mensaje a traves del evento "chat message"
          socket.emit('chat message', input.value);
          input.value = '';
        }
      });

      // Hacemos que el socket escuche el evento "chat message"
      socket.on('chat message', function(msg) {
        var item = document.createElement('li');
        item.textContent = msg;
        messages.appendChild(item);
        window.scrollTo(0, document.body.scrollHeight);
      });
    </script>
  </body>
</html>
```
---

## Usando diferentes canales

> example2.js

```js
// Usando un socket
// Un evento para enviar
// Un evento para recibir

io.on('connection', (socket) => {
  socket.on('chat A', msg => {
    console.log('message: ' + msg);
    io.emit('chat B', msg); 
  });
}
```
> example2.html

```js
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    if (input.value) {
      socket.emit('chat A', input.value);
      input.value = '';
    }
  });

  socket.on('chat B', function(msg) {
    var item = document.createElement('li');
    item.textContent = msg;
    messages.appendChild(item);
    window.scrollTo(0, document.body.scrollHeight);
  });
```

---

## Enviar un mensaje unicamente cuando se conectan al socket

> example3.js

```js
// Unicamente manda un mensaje
io.on('connection', (socket) => {
  socket.emit("hello", "world");
});
```

> example3.html

```js
  // Aunque envie datos al evento "hello" el servidor no hace nada.
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    if (input.value) {
      socket.emit('hello', input.value);
      input.value = '';
    }
  });

  // Escucha el evento "Hello"
  socket.on("hello", (msg) => {
    console.log(msg); // world
    var item = document.createElement('li');
    item.textContent = msg;
    messages.appendChild(item);
    window.scrollTo(0, document.body.scrollHeight);
  });
```

## Enviar mas de un parametro en el socket

> example4.js

```js
// Unicamente manda un mensaje
io.on('connection', (socket) => {
  socket.emit("hello", 1, "2", { 3: '4', 5: Buffer.from([6]) });
});
```

> example4.html

```js
socket.on("hello", (arg1, arg2, arg3) => {
  console.log(arg1);
  console.log(arg2);
  console.log(arg3);
  
  var item = document.createElement('li');
  item.textContent = `Arg1: ${arg1} - Arg2: ${arg2} - Arg3: ${arg3}`;
  messages.appendChild(item);


  window.scrollTo(0, document.body.scrollHeight);
});
```

## Para mas ejemplos:

> https://socket.io/docs/v4/emitting-events/

## Broadcasting

El broadcasting solo puede utilizarse del lado del servidor.

> https://socket.io/docs/v4/emit-cheatsheet/

**Enviando mensaje a todos los que estan escuchando el socket**
[image](https://socket.io/images/broadcasting-dark.png)

**Enviando mensaje a todos menos al que envio el mensaje**
[image](https://socket.io/images/broadcasting2-dark.png)

> example5.js

```js
socket.broadcast.emit("chat message", msg);
```

## Room

Los rooms solo pueden utilizarse del lado del servidor.

> example6.js

```js
var roomId = null;

io.on('connection', function(socket) {
    socket.on('room', function(room) {
        roomId = room;
        console.log(`Socket ${socket.id} joining ${room}`);
        socket.join(room);

        if (roomId){
            console.log("Exists Room")
            io.sockets.in(roomId).emit('message', `what is going on, party people from room ${roomId}?`);
        }

        socket.on('message', (r, msg) => {
            console.log(msg);
            io.sockets.in(r).emit('message', msg);
        });
    });
});
```
> example6.html

```js
let queryString = window.location.search;
const urlParams = new URLSearchParams(queryString);
var roomId = urlParams.get("id")

  socket.on('connect', function() {
      // Connected
      socket.emit('room', roomId);
  });

  form.addEventListener('submit', function(e) {
      e.preventDefault();
      if (input.value) {
          socket.emit('message', roomId, input.value);
          input.value = '';
      }
  });

  socket.on('message', function(msg) {
      var item = document.createElement('li');
      item.textContent = msg;
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
  });
```
