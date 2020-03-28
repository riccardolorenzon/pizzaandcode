---
title: "How to create a Chat application using Flask and React"
date: 2020-03-28T17:24:47+01:00
disqus: false
---
It's been a while since my last post on this blog. I travelled a lot in the begin of this year, and i consider myself like to have done it before the global spread of the Covid-19. 

I'm now working from home everyday, this saves me some time on the commute, that i can spend on reading and learning some new stuff. 
In this context i decided to create a chat application using React and Flask, mostly to try out new features of React 16.8, first of all [React hooks](https://reactjs.org/docs/hooks-intro.html). 

I also wanted to try to use Websockets with alternative web servers than `aiohttp`, since i have extensive experience with `Flask` but i never actually used it for this type of applications, i wanted to see how i could use it in this specific context. 
After a quick search on the web i found a library for `Flask` that allows low latency bi-directional communications between the clients and the server, [Flask-SocketIO](https://flask-socketio.readthedocs.io/en/latest/). 

### Requirements
You need to install the followings in your machine: 
* Python: A recent version of `Python 3`. 
* Yarn: A `node` manager for packages.
* Node.js: The Javascript runtime for our `React` application. 

### Creating the backend application
For this use case we might want to include the frontend and backend in a single directory. 
```
mkdir chat_app
cd chat_app
mkdir chat_app_be
```
I want to keep all my applications separated, for that i use the `Python` module `venv`. 
```
python3 -m venv venv 
source venv/bin/activate
```
For this example we will need the libraries:
* flask 
* flask-socketio
* python-dotenv
* gevent
* gevent-websocket

I included gevent and gevent-websocket since the Werkzeug development server doesn't support Websockets. It could still be used though, but only with long-polling trasnport. 

We will install this libraries using `pip install ...`, we will create a file `requirements.txt` containing the name of the libraries and the versions installed in order to make the creation of this environment reproducible. 
```
# install packages
pip install flask, flask-socket-io, python-dotenv
# create requirements.txt file
pip freeze > requirements.txt
```
We use python-dotenv in order to avoid to set any environment variables, using a dot file instead. 
```
echo "FLASK_APP=app.py \nFLASK_ENV=development" > .flaskenv
```
I really like the simplicity of Flask, compared to other Python web frameworks. All we need to do in order to get started is to create a single Python module, `app.py`. 
```
from flask import Flask
from flask_socketio import SocketIO, send

app = Flask(__name__)
app.config['SECRET_KEY'] = 'blah' 
socketio = SocketIO(app, cors_allowed_origins='*')

@socketio.on('message')
def handle_message(message):
    print('received message: ' + message)
    send(message, broadcast=True)

if __name__ == '__main__':
    socketio.run(app)
```
This module creates the socketio objects and starts the application by replacing the standard Flask development server start up `app.run`. 
For the ones that are wondering why not using `flask run` here to start the server, the flask run command introduced in Flask 0.11 can be used to start a Flask-SocketIO development server based on Werkzeug, but this method of starting the Flask-SocketIO server is not recommended due to lack of WebSocket support.

### Creating the frontend application
Next step is the frontend application, there are several ways to create a React application, fot this example i'm gonna use `npx`. 
```
npx create-react-app chat_app_fe
```
Create React App doesn’t handle backend logic or databases; it just creates a frontend build pipeline, so you can use it with any backend you want. Under the hood, it uses Babel and webpack, but you don’t need to know anything about them. 
For the client application we are gonna need the socket.io package. 
```
npm install socket.io
```
This will install the package and update the file package.json with the reference of this library. 
On the App.js we are gonna put all the logic for our application. 
```
import React, { useState } from 'react';
import io from "socket.io-client";
let endpoint = "http://localhost:5000";
let socket = io.connect(endpoint);

const App = () => {
  const [messages, setMessages] = useState([]);
  const [message, setMessage] = useState("");

  socket.on("message", msg => {
    setMessages([...messages, msg])});

  const onChange = (event) => {
    setMessage(event.target.value);
  };

  const onClick = () => {
    socket.emit("message", message);
    setMessage("");
  };

  return (
    <div className="App">
      <h2>Messages</h2>
      <div>
        {messages.map(msg => (<p>{msg}</p>))}
      </div>
      <p>
        <input type="text" onChange={onChange} value={message} />
      </p>
      <p>
        <input type="button" onClick={onClick} value="Send"/>
      </p>
    </div>
  );
};

export default App;
```
If you have used React before, you will recognize that we are using a functional component here. 
In order to get the messages from the websocket a new connection is started and a `state` variable is updated at each message. TO notice how the state is managed in this functional component by using React `useState` hook. 

If you now run the server, with `yarn start`, and open the application in multiple tabs, you will see how the message is broadcast to all clients. 

### Conclusion
This is a simple but working example of a chat. It can be easily extended with separate channels for groups of 2+ users. 
