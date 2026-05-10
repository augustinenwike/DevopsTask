# MY MERN STACK IMPLEMENTATION ON AWS
### MongoDB | Express | React | Node.js

---

## What I Gained From This Project

After completing this project, I:

- Strengthened my confidence working in the Linux terminal on AWS EC2
- Deepened my understanding of the MERN stack and how its layers communicate end-to-end
- Gained practical experience setting up a Node.js/Express REST API connected to MongoDB Atlas
- Learned how to scaffold a React frontend and proxy API requests to the backend
- Built and deployed a fully working **Todo web application** — with Create, Read, and Delete functionality — live on AWS EC2

---

## Project Overview

This document details the deployment of a **MERN stack** (MongoDB, Express, React, Node.js) application on an **AWS EC2** instance running Ubuntu 26.04 LTS. The application is a Todo manager where users can add and delete tasks, with data persisted in MongoDB Atlas.

Screenshots are presented in the exact order they were captured during deployment.

---

## Step 0 — Preparing Prerequisites

Launched a new EC2 instance with the following configuration:
- **AMI:** Ubuntu Server 26.04 LTS (HVM), SSD Volume Type — Free Tier eligible
- **Instance type:** t3.micro
- **Tag/Name:** `Project-MERN`

![EC2 Launch Instance](images/step0-launch.png)

The instance started successfully with:
- **Public IPv4:** `54.162.21.202`
- **Private IPv4:** `172.31.37.47`
- **Instance ID:** `i-07f274cd1e68521fd`
- **Status:** Running 

![EC2 Instance Running](images/step0-running.png)

Connected to the instance via SSH using the `.pem` key file:

```bash
ssh -i Downloads/udo-task.pem ubuntu@54.162.21.202
```

The host fingerprint was confirmed and added to known hosts. Successfully logged into Ubuntu 26.04 LTS:

![SSH Terminal Login](images/step0-ssh-login.png)

Switched to root user to avoid prefixing every command with `sudo`:

```bash
sudo -i
```

![Switch to root](images/step0-sudo.png)

---

## Step 1 — Updating the Server

Updated the package list to ensure the latest package information is available:

```bash
apt update
```

![apt update](images/step1-apt-update.png)

Upgraded all installed packages to apply the latest security patches (26 packages upgraded, including `openssh-server`, `vim`, `curl`, and others):

```bash
apt upgrade -y
```

![apt upgrade](images/step1-apt-upgrade.png)

---

## Step 2 — Installing Node.js

Node.js 18.x was installed via the NodeSource repository. Note: **Node.js 18.x is end-of-life** — for production use, consider Node 20.x or later.

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

A deprecation warning was displayed, but installation continued. Pre-requisite packages (`apt-transport-https`, `ca-certificates`, `curl`, `gnupg`) were confirmed up to date:

![NodeSource setup script](images/step2-node-setup.png)

Installed Node.js:

```bash
apt-get install -y nodejs
```

Node.js `18.20.8` and npm `10.8.2` were installed successfully (29.7 MB):

![Node.js installation](images/step2-node-install.png)

Verified the installations:

```bash
node -v   # v18.20.8
npm -v    # 10.8.2
```

![Node and npm version check](images/step2-node-verify.png)

---

## Step 3 — Setting Up the Backend Project

Created the project directory and initialised the Node project:

```bash
mkdir Todo
cd Todo
npm init
```

Filled in the prompts:
- **package name:** todo
- **version:** 1.0.0
- **description:** A todo app
- **entry point:** index.js
- **keywords:** todo, application
- **author:** udo
- **license:** ISC

The generated `package.json` was confirmed and written to disk:

![npm init](images/step3-npm-init.png)

Installed Express as the web framework:

```bash
npm install express
```

65 packages added, 0 vulnerabilities found:

![Install Express](images/step3-express-install.png)

Created the main entry file and installed `dotenv` for environment variable management:

```bash
touch index.js
npm install dotenv
```

![touch index.js and install dotenv](images/step3-dotenv-install.png)

---

## Step 4 — Creating the Initial Express Server

Opened `index.js` in vim and wrote the initial server:

```bash
vim index.js
```

```javascript
const express = require('express');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "\*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use((req, res, next) => {
  res.send('Welcome to Express');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`)
});
```

![index.js in vim](images/step4-index-vim.png)

Started the server:

```bash
node index.js
```

Output confirmed: `Server running on port 5000`

![Server running on port 5000](images/step4-server-running.png)

Tested in the browser at `http://54.162.21.202:5000`:

**Result:** "Welcome to Express" displayed successfully 

![Welcome to Express browser](images/step4-browser.png)

---

## Step 5 — Setting Up Routes

Created a `routes` directory and an `api.js` file:

```bash
mkdir routes
cd routes
touch api.js
```

![Create routes directory](images/step5-routes-mkdir.png)

Opened `api.js` in vim and defined the three API routes:

```javascript
const express = require('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

![api.js routes file](images/step5-api-routes.png)

---

## Step 6 — Installing Mongoose and Creating the Todo Model

From the `routes` directory, installed Mongoose for MongoDB object modelling:

```bash
npm install mongoose
```

17 packages added, 0 vulnerabilities:

![Install Mongoose](images/step6-mongoose-install.png)

Created a `models` directory inside `routes` and created `todo.js`:

```bash
mkdir models
cd models
touch todo.js
vim todo.js
```

![Create models directory](images/step6-models-mkdir.png)

Defined the Mongoose schema and model:

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

![todo.js model code](images/step6-todo-model.png)

---

## Step 7 — Connecting to MongoDB Atlas

Created a `.env` file in the root `Todo` directory and added the MongoDB Atlas connection string:

```bash
vim .env
```

```
DB=mongodb+srv://udo:038August@nodetuts.ycajm63.mongodb.net/MERN?appName=MERN
```

![.env MongoDB connection string](images/step7-env.png)

Updated `index.js` with the full production-ready configuration — including MongoDB connection, body-parser, route mounting, and error handling:

```bash
vim index.js
```

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('Database connected successfully'))
  .catch(err => console.log(err));

//since mongoose promise is deprecated, we override it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "\*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
  console.log(err);
  next();
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`)
});
```

![Updated index.js in vim](images/step7-index-updated.png)

Restarted the server:

```bash
node index.js
```

Output confirmed both server and database are up:
- `Server running on port 5000`
- `Database connected successfully` 

![Server and DB running](images/step7-server-db.png)

---

## Step 8 — Completing the API Routes

Updated `routes/api.js` to include the actual Mongoose logic for each route:

```javascript
const express = require('express');
const router = express.Router();
const Todo = require('./models/todo');

router.get('/todos', (req, res, next) => {
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});

router.post('/todos', (req, res, next) => {
  if(req.body.action){
    Todo.create(req.body)
      .then(data => res.json(data))
      .catch(next);
  } else {
    res.json({ error: "The input field is empty" });
  }
});

router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({ "_id": req.params.id })
    .then(data => res.json(data))
    .catch(next);
});

module.exports = router;
```

---

## Step 9 — Testing the API with Insomnia

With the server running, the REST API was tested using **Insomnia**.

**POST** — Created a new todo item:

- **URL:** `http://54.162.21.202:5000/api/todos`
- **Body (JSON):** `{ "action": "Finished project 1 and 2" }`
- **Status:** 200 OK 
- **Response:** `[{"action":"Finished project 1 and 2","_id":"6a00ae2165a727da36639e2d","__v":0}]`

![POST request in Insomnia](images/step9-post.png)

**GET** — Retrieved all todos:

- **URL:** `http://54.162.21.202:5000/api/todos`
- **Status:** 200 OK 
- **Response:** `[{"_id":"6a00ae2165a727da36639e2d","action":"Finished project 1 and 2"}]`

![GET request in Insomnia](images/step9-get.png)

**DELETE** — Deleted a todo by ID:

- **URL:** `http://54.162.21.202:5000/api/todos/6a00ae2165a727da36639e2d`
- **Status:** 200 OK 
- **Response:** The deleted document was returned

![DELETE request in Insomnia](images/step9-delete.png)

**GET after DELETE** — Confirmed the item was removed:

- **Response:** `[]` — empty array 

![GET after DELETE in Insomnia](images/step9-get-empty.png)

---

## Step 10 — Creating the React Frontend

From the `Todo` root directory, scaffolded the React client app:

```bash
npx create-react-app client
```

`create-react-app@5.1.0` was installed. Several deprecation warnings appeared (these are cosmetic — the app is deprecated in favour of frameworks like Next.js or Vite, but functional for this exercise). React app was created at `/root/Todo/client`:

![Create React App](images/step10-cra.png)

Installed `concurrently` to run both the backend and frontend with a single command:

```bash
npm install concurrently --save-dev
```

25 packages added:

![Install concurrently](images/step10-concurrently.png)

Installed `nodemon` to auto-restart the backend server when files change:

```bash
npm install nodemon --save-dev
```

26 packages added:

![Install nodemon](images/step10-nodemon.png)

---

## Step 11 — Configuring package.json Scripts

Updated the root `package.json` (`~/Todo/package.json`) to add the development scripts:

```json
{
  "name": "todo",
  "version": "1.0.0",
  "description": "A todo app",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "start-watch": "nodemon index.js",
    "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
  },
  "keywords": ["todo", "application"],
  "author": "udo",
  "license": "ISC",
  "dependencies": {
    "dotenv": "^17.4.2",
    "express": "^5.2.1",
    "mongoose": "^8.23.1"
  },
  "devDependencies": {
    "concurrently": "^9.2.1",
    "nodemon": "^3.1.14"
  }
}
```

![Root package.json in vim](images/step11-root-packagejson.png)

Configured the React client's `package.json` (`~/Todo/client/package.json`) to proxy API requests to the backend — this allows the frontend to call `/api/todos` without specifying the full server address:

```json
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:5000",
  ...
}
```

![Client package.json with proxy](images/step11-client-packagejson.png)

---

## Step 12 — Running the Full Stack in Development

Started the React client standalone first to confirm it compiles:

```bash
cd client
npm run start
```

Compiled successfully. React app served at:
- Local: `http://localhost:3000`
- Network: `http://172.31.37.47:3000`

![React client compiled successfully](images/step12-react-compile.png)

Verified in the browser at `http://54.162.21.202:3000`:

**Result:** Default React welcome page loaded 

![Default React browser page](images/step12-react-browser.png)

Stopped the client and ran both together from the root `Todo` directory:

```bash
cd ~/Todo
npm run dev
```

Both processes started via `concurrently`:
- **[0]** nodemon started, watching `*.js`, `*.mjs`, `*.cjs`, `*.json`
- **[0]** `Server running on port 5000`
- **[0]** `Database connected successfully`
- **[1]** React dev server started
- **[1]** `Compiled successfully!`

![npm run dev output](images/step12-npm-run-dev.png)

---

## Step 13 — Creating the React Components

Navigated to the components directory and created the three component files:

```bash
cd client/src
mkdir components
cd components
touch Input.js ListTodo.js Todo.js
```

![Create component files](images/step13-components-mkdir.png)

Installed `axios` in the client for HTTP requests to the backend API:

```bash
cd ~/Todo/client
npm install axios
```

3 packages added (26 vulnerabilities flagged — these are cosmetic audit warnings from the older `create-react-app` toolchain):

![Install axios](images/step13-axios-install.png)

---

## Step 14 — Writing the React Components

### Input.js

Opened `Input.js` in vim:

```bash
vi Input.js
```

This is a **class component** responsible for the input field and the "add todo" button. It POSTs a new todo to the backend, calls `getTodos()` on success, then clears the input:

```javascript
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ""
  }

  addTodo = () => {
    const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    } else {
      console.log('input field required')
    }
  }

  handleChange = (e) => {
    this.setState({
      action: e.target.value
    })
  }

  render() {
    let { action } = this.state;
    return (
      <div>
        <input type="text" onChange={this.handleChange} value={action} />
        <button onClick={this.addTodo}>add todo</button>
      </div>
    )
  }
}

export default Input
```

![Input.js code in vim](images/step14-input-js.png)

### ListTodo.js

Opened `ListTodo.js` in vim:

```bash
vi ListTodo.js
```

This is a **functional component** that receives the `todos` array and a `deleteTodo` function as props. It maps each todo to an `<li>` — clicking an item triggers deletion:

```javascript
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {
        todos &&
        todos.length > 0 ?
        (
          todos.map(todo => {
            return (
              <li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
            )
          })
        )
        :
        (
          <li>No todo(s) left</li>
        )
      }
    </ul>
  )
}

export default ListTodo
```

![ListTodo.js code in vim](images/step14-listtodo-js.png)

### Todo.js

Opened `Todo.js` in vim:

```bash
vi Todo.js
```

This is the **parent container class component** that manages all state and API calls. It renders both `<Input>` and `<ListTodo>`:

```javascript
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  }

  componentDidMount(){
    this.getTodos();
  }

  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if(res.data){
          this.setState({
            todos: res.data
          })
        }
      })
      .catch(err => console.log(err))
  }

  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))
  }

  render() {
    let { todos } = this.state;
    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )
  }
}

export default Todo;
```

![Todo.js code in vim](images/step14-todo-js.png)

---

## Step 15 — Updating App.js

Updated the root `App.js` to remove the default React template and render the `Todo` component:

```bash
vi App.js
```

```javascript
import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
}

export default App;
```

![App.js updated in vim](images/step15-app-js.png)

---

## Step 16 — Styling the Application

### App.css

Updated `App.css` with a custom dark theme:

```bash
vi App.css
```

```css
.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 60%;
  margin-left: auto;
  margin-right: auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5rem;
  color: #787a80;
}

input:focus {
  outline: none;
}

button {
  width: 25%;
  height: 45px;
  border: none;
  margin-left: 10px;
  font-size: 25px;
  background: #101113;
  border-radius: 5px;
  color: #787a80;
  cursor: pointer;
}

button:focus {
  outline: none;
}

ul {
  list-style: none;
  text-align: left;
  padding: 15px;
  background: #171a1f;
  border-radius: 5px;
}

li {
  padding: 15px;
  font-size: 1.5rem;
  margin-bottom: 15px;
  background: #282c34;
  border-radius: 5px;
  overflow-wrap: break-word;
  cursor: pointer;
}

@media only screen and (min-width: 300px) {
  .App {
    width: 80%;
  }
  input {
    width: 100%;
  }
  button {
    width: 100%;
    margin-top: 15px;
    margin-left: 0;
  }
}

@media only screen and (min-width: 640px) {
  .App {
    width: 60%;
  }
  input {
    width: 50%;
  }
  button {
    width: 30%;
    margin-left: 10px;
    margin-top: 0;
  }
}
```

![App.css in vim](images/step16-app-css.png)

### index.css

Updated `index.css` with global body styles and dark background:

```bash
vi index.css
```

```css
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282c34;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
    monospace;
}
```

![index.css in vim](images/step16-index-css.png)

---

## Step 17 — Final Launch and Verification

Returned to the root `Todo` directory and ran the full stack:

```bash
cd ~/Todo
npm run dev
```

Both the backend and frontend started successfully via `concurrently`:
- **[0]** nodemon restarted `node index.js`
- **[0]** `Server running on port 5000`
- **[0]** `Database connected successfully`
- **[1]** React compiled successfully
- **[1]** App available at `http://localhost:3000` / `http://172.31.37.47:3000`

![Final npm run dev output](images/step17-final-dev.png)

Opened the browser at `http://54.162.21.202:3000`:

**Result:** The fully functional Todo application loaded with:
- A heading: **My Todo(s)**
- An input field and **add todo** button
- A live list of todos fetched from MongoDB:
  - Done with project 3
  - Done with project 2
  - Done with Project 1
  - Starting Project 4
- Clicking any item **deletes it** from the database and refreshes the list 

![Final Todo app in browser](images/step17-final-browser.png)

---

##  Final Result — MERN Stack Fully Operational

| Component | Technology | Version / Detail |
|-----------|-----------|-----------------|
| **M** — MongoDB | MongoDB Atlas (Cloud) | 8.x (NoSQL database) |
| **E** — Express | Express.js | 5.2.1 |
| **R** — React | React + Create React App | 19.2.0 |
| **N** — Node.js | Node.js via NodeSource | 18.20.8 |

**My MERN stack is completely installed, fully operational, and serving a live React + Express + MongoDB application on AWS EC2.** 