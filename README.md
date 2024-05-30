<!-- # CodeEasy Themes - Visual Studio Code Theme Extension

<img src="https://github.com/Parvsharma04/codeeasy/blob/master/logo.png?raw=true" width="100px"/>

## Overview

Welcome to CodeEasy Themes, a collection of visually pleasing and easy-to-read themes for Visual Studio Code. This extension provides a variety of carefully crafted color schemes to enhance your coding experience and make your code more readable. -->

## CODE

```
const express = require('express');
const app = express();
const session = require('express-session');
const loginRouter = require('./src/routes/login');
const authorRouter = require('./src/routes/author');
const adminRouter = require('./src/routes/admin');
const articleRouter = require('./src/routes/articles');
const homeRouter = require('./src/routes/home');
const connectDB = require('./DB/connect')

connectDB()

app.listen(8080, ()=>{console.log(`http://localhost:8080`)});

app.set('view engine','ejs');
app.set('views','./src/views')

app.use(session({
    resave:false,
    saveUninitialized:false,
    secret:'keyboard'
}));

app.use(express.urlencoded({extended: false}));
app.use(express.json());
app.use(express.static('./public'))

app.use('/',loginRouter);

const userAuth = (req,res,next)=>{

    if(req.session && req.session.user){
        next()
    }
    else{
        res.redirect('/');
    }
}

const adminAuth = (req,res,next)=>{
    if(req.session.user.role=='admin'){
        next();
    }
    else{
        res.send('Invalid access')
    }
}

const authorAuth = (req,res,next)=>{
    if(req.session.user.role=='author'){
        next();
    }
    else{
        res.send('Invalid access')
    }
}
app.use('/',homeRouter);
app.use('/admin',userAuth,adminAuth,adminRouter);
app.use('/author',userAuth,authorAuth,authorRouter)
app.use('/article',userAuth,articleRouter);


```

## SCHEMAS

```
const mongoose = require('mongoose')
const author = require('../src/routes/author')

const POST = new mongoose.Schema({
    title:{
        type: String,
        required: [true, 'title is required'],
        trim: true
    },
    article:{
        type: String,
        required: [true, 'post is required'],
    },
    image:{
        type: String
    },
    articleId: {
        type: Number
    },
    date:{
        type: Date
    },
    author:{
        type: String
    },
    authorId:{
        type: Number
    }
}, {collection: 'posts'})

module.exports = mongoose.model('entry', POST)
```

```
const mongoose = require('mongoose')

const user = new mongoose.Schema({
    firstName:{
        type: String
    },
    lastName:{
        type: String
    },
    maidenName:{
        type: String
    },
    age:{
        type: Number
    },
    gender:{
        type: String
    },
    email:{
        type: String
    },
    phone:{
        type: String
    },
    username:{
        type: String
    },
    password:{
        type: String
    },
    role:{
        type: String
    }
}, {collection: 'users'})

module.exports = mongoose.model('user', user)
```

```
const mongoose = require('mongoose')
const url = require('../url/url')
const connectDB = async () =>{
    try{
        await mongoose.connect(url)
        console.log('DB Connected.')
    }catch(err){
        console.log(err)
    }
}

module.exports = connectDB
```

```


module.exports=(
    function(){
        const router = require('express').Router();
        const User = require('../../models/schemaUser');
        const posts = require('../../models/schemaPOST')

        router.get('/dashboard', async (req,res)=>{
             const article= await posts.find({});
             const users= await User.find({});
            res.render('./admin/dashboard.ejs',{user:req.session.user,article,users})
        })

        router.get('/addUser',(req,res)=>{
            res.render('./admin/adduser.ejs');
        })

        router.post('/addUser', async (req,res)=>{
            const existingUser = await User.find({username: req.body.username});
            if(existingUser.length > 0){
                res.send('User already exists');
            } else {
                const userObj = {
                    firstName:req.body.firstname,
                    lastName:req.body.lastname,
                    username:req.body.username,
                    email:req.body.email,
                    password:req.body.password,
                    role:req.body.role
                }
                await User.create(userObj); // Make sure to await the creation process
                res.send('User Added Successfully');
            }
        });


        router.get('/deleteUser/:username', async (req, res) => {
            try {
                const user = await User.findOne({ username:req.params.username });
                if (!user) {
                    res.json(JSON.stringify({message:'error'}));
                }
                else{
                    await User.findOneAndDelete({username: req.params.username})
                    res.json(JSON.stringify({message:'ok'}));
                }
            } catch (error) {
                console.error(error);
            }
        });

        return router;
    }
)();
```

```
module.exports = (
    function () {
        const router = require('express').Router();
        const multer = require('multer');
        const path = require('path');
        const entry = require('../../models/schemaPOST')
        const storage = multer.diskStorage({
            destination: './public/images',
            filename: function (req, file, cb) {
                cb(null, Date.now() + '-' + file.originalname);
            }
        });

        const upload = multer({
            storage: storage,
            fileFilter: function (req, file, cb) {
                const extension = path.extname(file.originalname);
                if (extension.toLowerCase() == '.jpg' || extension.toLowerCase() == '.jpeg' || extension.toLowerCase() == '.png') {
                    cb(null, true);
                }
                else {
                    cb(new Error('Invalid file type'))
                }
            }
        });
        router.get('/add', (req, res) => {
            res.render('./articles/add.ejs');
        })

        router.post('/add', upload.single('image'), async(req, res) => {
                const article = {
                    articleId: Date.now(),
                    title: req.body.title,
                    article: req.body.article,
                    image: req.file.filename,
                    date: new Date().toLocaleDateString(),
                    author: req.session.user.firstName+" "+req.session.user.lastName,
                    authorId: req.session.user.id
                }
                entry.create(article);
                res.render('./articles/add', { message: 'Record added sucessfully' })

        });

        router.get('/delete/:id',async (req,res)=>{
            try {
                const article = await entry.findOne({articleId:req.params.id});
            if(!article){
                res.json(JSON.stringify({message:'error'}));
            }
            else{
                await entry.findOneAndDelete({articleId: req.params.id})
                res.json(JSON.stringify({message:'ok'}));
            }
            } catch (error) {
                console.log(error)
            }
        })
        return router;
    }
)();
```

```
module.exports=(
    function(){
        const router = require('express').Router();
        const entry = require('../../models/schemaPOST')

        router.get('/dashboard', async (req,res)=>{
            const articles = await entry.find({authorId: req.session.user.id});
            res.render('./author/dashboard.ejs',{user:req.session.user,articles})
        })

        return router;
    }
)();
```

```
const entry = require('../../models/schemaPOST')
module.exports = (
    function(){
        const router = require('express').Router();
        router.get('/', async (req,res)=>{
            const articles = await entry.find({})
            res.render('./home/home.ejs',{articles})
        })

        router.get('/post/:id', async (req, res) => {
            try {
                const article = await entry.findOne({ articleId: req.params.id });
                if (!article) {
                    return res.status(404).send('Article not found');
                }
                res.render('./home/post.ejs', { article });
            } catch (error) {
                console.error('Error fetching article:', error);
                res.status(500).send('Internal Server Error');
            }
        });


        return router;
    }
)();
```

```
const router = require('express').Router();
const User = require('../../models/schemaUser');

router.get('/login', (req, res) => {
    res.render('./login/login.ejs');
});

router.post('/login', async (req, res) => {
    try {
        const user = await User.findOne({ username: req.body.username, password: req.body.pass });

        if (user) {
            req.session.user = user;
            if (user.role === 'admin') {
                return res.redirect('/admin/dashboard');
            } else if (user.role === 'author') {
                return res.redirect('/author/dashboard');
            }
        } else {
            return res.render('./login/login.ejs', { message: 'Invalid username or password' });
        }
    } catch (error) {
        console.error('Error occurred during login:', error);
        return res.status(500).send('Internal Server Error');
    }
});

module.exports = router;
```

## EJS TEMPLATES

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Admin Dashboard</title>
</head>
<body>
  <h1>Admin Dashboard</h1>
  <ul>
    <li>Hi! <%=user.firstName%></li>
    <li><a href="/">Home</a></li>
    <li><a href="/article/add">Add Post</a></li>
    <li><a href="/admin/addUser">Add User</a></li>
  </ul>

  <div class="section">
    <h2>Articles</h2>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Title</th>
          <th>Author</th>
          <th>Date</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <% article.forEach(article => { %>
        <tr>
          <td><%= article.articleId %></td>
          <td><%= article.title %></td>
          <td><%= article.author %></td>
          <td><%= article.date %></td>
          <td>
            <button onclick="deleteArticle(this, '<%= article.articleId %>')">
              Delete
            </button>
          </td>
        </tr>
        <% }) %>
      </tbody>
    </table>
  </div>

  <div class="section">
    <h2>User Details</h2>
    <table>
      <thead>
        <tr>
          <th>Username</th>
          <th>First Name</th>
          <th>Last Name</th>
          <th>Email</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <% users.forEach(user => { %>
        <tr>
          <td><%= user.username %></td>
          <td><%= user.firstName %></td>
          <td><%= user.lastName %></td>
          <td><%= user.email %></td>
          <td>
            <button onclick="deleteUser(this, '<%= user.username %>')">
              Delete
            </button>
          </td>
        </tr>
        <% }) %>
      </tbody>
    </table>
  </div>

  <script>
    async function deleteArticle(e, id) {
      const response = await fetch(`/article/delete/${id}`);
      const data = JSON.parse(await response.json());
      if (data.message == "ok") {
        alert("Record deleted successfully");
        const parentNode = e.parentNode.parentNode;
        parentNode.remove();
      } else if (data.response == "error") {
        alert("Something went wrong");
      }
    }

    async function deleteUser(e, username) {
      const response = await fetch(`/admin/deleteUser/${username}`);
      const data = JSON.parse(await response.json());
      if (data.message == "ok") {
        alert("Record deleted successfully");
        const parentNode = e.parentNode.parentNode;
        parentNode.remove();
      } else if (data.response == "error") {
        alert("Something went wrong");
      }
    }
  </script>
</body>
</html>
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div class="post">
        <h4><%=article.title%></h4>
        <div class="post-img">
            <img src="/images/<%= article.image %>" height="150px" width="120px"/>

        </div>
        <p><%=article.article%></p>
        <p><span><%=article.date%></span><span><%=article.author%></span></p>
    </div>
</body>
</html>
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div style="text-align: right; margin: 20px;">
        <a href="/login" style="text-decoration: none; background-color: #007bff; color: #fff; padding: 10px 20px; border-radius: 5px;">Login</a>
    </div>
    <div class="container">
        <% articles.forEach(article=>{%>
            <div class="blog-card">
                <div class="card-img">
                    <img src="/images/<%= article.image %>" height="150px" width="120px"/>
                </div>
                <h4><a href="/post/<%=article.articleId%>"><%=article.title%></a></h4>
                <p><%=article.article.slice(0,32)%>...</p>
                <p><span><%=article.date%></span><span><%=article.author%></span></p>
                <a href="/post/<%=article.articleId%>">Read More</a>
            </div>
        <%})%>
        
    </div>

    <!-- Add login button here -->
</body>
</html>

```

## package.json

```
{
  "dependencies": {
    "ejs": "^3.1.9",
    "express": "^4.19.2",
    "express-session": "^1.18.0",
    "mongoose": "^8.3.2",
    "multer": "^1.4.5-lts.1"
  },
  "nodemonConfig": {
    "ignore": [
      "*.json",
      "src/model/*"
    ]
  }
}
```


```
TO-DO APP (STORAGE)

let container = document.getElementById('text');
let input = document.getElementById('input');
let addButton = document.getElementById('add');

//function to add key and values in localstorage
const addLocalValues = (key, value) => {
    let newContainer = document.createElement('div');
    newContainer.setAttribute('id', 'newContainer');

    let left = document.createElement('div');
    left.setAttribute('id', 'left');

    let right = document.createElement('div');
    right.setAttribute('id', 'right');

    let newPencil = document.createElement('button');
    newPencil.setAttribute('id', 'newPencil');
    newPencil.innerHTML = '<i class="fa-solid fa-pencil fa-2xl"></i>';

    let newCheck = document.createElement('input');
    newCheck.setAttribute('id', 'newCheck');
    newCheck.setAttribute('type', 'checkbox');

    let newButton = document.createElement('button');
    newButton.setAttribute('id', 'newButton');
    newButton.innerHTML = '<i class="fa-solid fa-trash fa-bounce fa-2xl" style="color: black;"></i>';

    // Set the checkbox's checked property based on the value from localStorage
    let isChecked = localStorage.getItem(key) === "true";
    newCheck.checked = isChecked;
    left.innerHTML = key;
    newContainer.appendChild(left);
    // update the task is done or not
    right.appendChild(newPencil);
    right.appendChild(newCheck);
    right.appendChild(newButton);


    newContainer.appendChild(right);
    container.appendChild(newContainer);

    newButton.addEventListener('click', () => {
        localStorage.removeItem(key); // Use the key, not innerText
        container.removeChild(newContainer);
    })
}

//function to check whether localStorage is empty or not
if (localStorage.length != 0) {
    for (let i = 0; i < localStorage.length; i++) {
        addLocalValues(Object.keys(localStorage)[i], Object.values(localStorage)[i]);
        // console.log(Object.keys(localStorage)[i], Object.values(localStorage)[i]);
        // console.log(Object.keys(localStorage)[i]);
        // console.log(Object.values(localStorage)[i]);        
    }
}

input.addEventListener('keypress', (e) => {
    if (e.key === "Enter" && input.value != "") {
        let newContainer = document.createElement('div');
        newContainer.setAttribute('id', 'newContainer');

        let left = document.createElement('div');
        left.setAttribute('id', 'left');

        let right = document.createElement('div');
        right.setAttribute('id', 'right');
        let newCheck = document.createElement('input');
        newCheck.setAttribute('id', 'newCheck');
        newCheck.setAttribute('type', 'checkbox')
        let newButton = document.createElement('button');
        newButton.setAttribute('id', 'newButton');
        newButton.innerHTML = '<i class="fa-solid fa-trash fa-bounce fa-2xl" style="color: black;"></i>';
        let newPencil = document.createElement('button');
        newPencil.setAttribute('id', 'newPencil');
        newPencil.innerHTML = '<i id="newPencil" class="fa-solid fa-pencil fa-2xl"></i>';


        left.innerHTML = input.value;
        newContainer.appendChild(left);
        right.appendChild(newPencil);
        right.appendChild(newCheck);
        right.appendChild(newButton);
        newContainer.appendChild(right);
        container.appendChild(newContainer);

        //adding to local Storage
        localStorage.setItem(input.value, false);
        input.value = "";

        newButton.addEventListener('click', () => {
            localStorage.removeItem(newButton.parentElement.parentElement.innerText);
            container.removeChild(newContainer);
        })
    }
})

// function to update localStorage on the based on task complete
container.addEventListener('change', (e) => {
    if (e.target.type === 'checkbox') {
        const isChecked = e.target.checked;
        const textContent = e.target.parentElement.parentElement.querySelector('#left').textContent;
        localStorage.setItem(textContent, isChecked);
    }
});

//function to edit div text by using pencil symbol
container.addEventListener('click', (e) => {
    if (e.target.id === 'newPencil') {
        const taskContainer = e.target.parentElement.parentElement.parentElement;
        const taskTextElement = taskContainer.querySelector('#left');
        // Store the old value before editing
        const oldValue = taskTextElement.textContent;
        taskTextElement.contentEditable = true;
        taskTextElement.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                taskTextElement.contentEditable = false;
                const newValue = taskTextElement.textContent;
                // Remove the old value from localStorage
                localStorage.removeItem(oldValue);
                // Set the updated value in localStorage
                localStorage.setItem(newValue, false);
                // Update the task text content
                taskTextElement.textContent = newValue;
            }
        });
    }
});

addButton.addEventListener('click', (e) => {
    if (input.value == '') {
        alert("Plz add enter something to add");
    } else {
        let newContainer = document.createElement('div');
        newContainer.setAttribute('id', 'newContainer');

        let left = document.createElement('div');
        left.setAttribute('id', 'left');

        let right = document.createElement('div');
        right.setAttribute('id', 'right');
        let newCheck = document.createElement('input');
        newCheck.setAttribute('id', 'newCheck');
        newCheck.setAttribute('type', 'checkbox')
        let newButton = document.createElement('button');
        newButton.setAttribute('id', 'newButton');
        newButton.innerHTML = '<i class="fa-solid fa-trash fa-bounce fa-2xl" style="color: black;"></i>';
        let newPencil = document.createElement('button');
        newPencil.setAttribute('id', 'newPencil');
        newPencil.innerHTML = '<i class="fa-solid fa-pencil fa-2xl"></i>';


        left.innerHTML = input.value;
        newContainer.appendChild(left);
        right.appendChild(newPencil);
        right.appendChild(newCheck);
        right.appendChild(newButton);
        newContainer.appendChild(right);
        container.appendChild(newContainer);

        //adding to local Storage
        localStorage.setItem(input.value, false);
        input.value = "";

        newButton.addEventListener('click', () => {
            localStorage.removeItem(newButton.parentElement.parentElement.innerText);
            container.removeChild(newContainer);
        })
    }
    // function to update localStorage on the based on task complete
    const list = document.querySelectorAll("input[type=checkbox]");

    for (const checkbox of list) {
        checkbox.addEventListener('click', () => {
            const isChecked = checkbox.checked;
            const textContent = checkbox.parentElement.parentElement.textContent;

            localStorage.setItem(textContent, isChecked);
        });

        // Initialize the checkbox state based on localStorage
        const textContent = checkbox.parentElement.parentElement.textContent;
        const isChecked = localStorage.getItem(textContent) === 'true';

        checkbox.checked = isChecked;
    }
});







BUILD A QUIZ MODULE 

const form = document.querySelector('form')
const result = document.querySelector('.result')
const correctOption = document.querySelector('#correct')
const incorrectOption = document.querySelector('#incorrect')
let score = 0;
const title = document.querySelector(".question span")
const answers = document.querySelectorAll(".answers ul li");
const prevBtn = document.getElementById('previous')
const nextBtn = document.getElementById('next') 
const totalQuestions = document.getElementById('totalQuestions');
const heading = document.getElementById('heading');

correctOption.style.display = "none"
incorrectOption.style.display = "none"
result.style.display = "none"

const reqQuestion = prompt('How many question do you want ?');

const url = https://quiz26.p.rapidapi.com/questions?cat=c&sort=questions&limit=${reqQuestion}&page=3;
const options = {
	method: 'GET',
	headers: {
		'X-RapidAPI-Key': '6f692112c4msh67a2f9cedce6f8bp1f05a6jsnb801684049df',
		'X-RapidAPI-Host': 'quiz26.p.rapidapi.com'
	}
};
let currentQuestionIndex = 0;

async function fetchData() {
    try {
        let response = await fetch(url, options);
        let quizData = await response.json();

        totalQuestions.innerText = No of Questions : ${reqQuestion}

        // setting up the questions and answers        
        setQuestion(quizData.data[currentQuestionIndex]);

        nextBtn.addEventListener('click', () => {
            if (currentQuestionIndex < quizData.data.length - 1) {
                currentQuestionIndex++;
                setQuestion(quizData.data[currentQuestionIndex]);
            } else {
                heading.innerText = Socre : ${score}
                alert('You have reached the last question.');
            }
        });

        prevBtn.addEventListener('click', () => {
            if (currentQuestionIndex > 0) {
                currentQuestionIndex--;
                setQuestion(quizData.data[currentQuestionIndex]);
            } else {
                alert('You are on the first question.');
            }
        });

        form.addEventListener("submit", (e) => {
            e.preventDefault();
        
            let selectedOption = document.querySelector('input[name="option"]:checked');
            console.log(selectedOption.value)

            if (selectedOption) {
                console.log(quizData.data[0].answer)
                if (selectedOption.value === ${quizData.data[currentQuestionIndex].answer}) {
                    correctOption.style.display = 'inline flex';
                    incorrectOption.style.display = 'none';
                    score++;
                } else {
                    correctOption.style.display = 'none';
                    incorrectOption.style.display = 'inline flex';
                }
                result.style.display = "flex"
            } 
            else {
                alert('Please select an option before submitting.');
            }
        })
    } 
    catch (error) {
        console.error(error);
    }
}
function setQuestion(questionData) {
    title.innerHTML = questionData.question;
    let i = 0;
    answers.forEach(li => {
        li.childNodes[3].innerHTML = questionData.options[i];
        li.childNodes[1].value = questionData.options[i];
        i++;
    });
}

fetchData();









Creating sum, multiply, division and subtraction functions in one module and export them and use them in another module.


function add(...a){
  let sum = 0;
  for(elem of a){
    sum += elem;
  }
  return sum;
}
function sub(...a){
  let minus = a[0];
  let n = a.length;
  for(let i=1; i<n; i++){
    minus -= a[i];
  }
  return minus;
}
function mul(...a){
  let multiply = 0;
  for(elem of a){
    multiply *= elem;
  }
  return multiply;
}
function div(...a){
  let division = a[0];
  let n = a.length;
  for(let i=1; i<n; i++){
    division /= a[i];
  }
  return division;
}
module.exports = {add, sub, mul, div};









Write a function which takes directory path as input and displays all the directories and files inside the it.

const prompt = require('prompt-sync')();

const fs = require('fs')



let dpath = prompt("Enter directory path: ");

const data = fs.readdirSync(dpath, 'utf-8');

console.log(data);










Using EventEmitter class raise an event in one module and write the listener for that event in another module to handle it .


const eventEmitter = require('events');

const event = new eventEmitter();

function eventSender() {

  event.emit('run');

  console.log("Handling the event")

}

event.addListener('run', () => {

  console.log("event emitter");

});

module.exports = eventSender;










Ecommerce
Task 1: Create a Nodejs server to handle request at port 3001, give proper message at console "Server started..." if actually it started or give "unable to start server" if there is an issue in starting the server.

Task 2: Create a end point “/” return all the products from products.json file to users. The products.json is json file with Title, Category and price as three properties.

Task 3: Add some products in file with category="food" and category="others"

Task 4: Edit the above endpoint to receive the category and return the products matching the category e.g /?category=food returns all the products with category=food


const http = require('http');
const fs = require('fs')
const QueryString = require('url')
try{
  http.createServer( (req, res) => {
    const commingUrl = QueryString.parse(req.url, true);
    if(req.url === '/'){
      const data = fs.readFileSync('./products.json', 'utf-8');
      res.setHeader('Content-type', 'application/json');
      res.write(data);
    }
    else if(commingUrl.pathname === '/add'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      arr.push(commingUrl.query);
      fs.writeFileSync('./products.json', JSON.stringify(arr));
    }
    else if(commingUrl.pathname === '/category'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      let ans = [];
      for(elem of arr){
        if(elem.Category === commingUrl.query.search){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    res.end();
  }).listen(3001, ()=> {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server");
}










Ecommerce Search
Task 1: Create a end point to handle the “/products?category=cloths” that return all the products from products.json file matching the category passed to users. The products.json is json file with Title, Category and price as three properties.

Task 2: Create a end point to handle the “/filterproducts?category=cloths&price=300” that return all the products from products.json file matching the category and price>=(specified price) to users. The products.json is json file with Title, Category and price as three properties.



const http = require('http');
const fs = require('fs')
const QueryString = require('url')
try{
  http.createServer( (req, res) => {
    const commingUrl = QueryString.parse(req.url, true);
    if(req.url === '/'){
      const data = fs.readFileSync('./products.json', 'utf-8');
      res.setHeader('Content-type', 'application/json');
      res.write(data);
    }
    else if(commingUrl.pathname === '/add'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      arr.push(commingUrl.query);
      fs.writeFileSync('./products.json', JSON.stringify(arr));
    }
    else if(commingUrl.pathname === '/category'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      let ans = [];
      for(elem of arr){
        if(elem.Category === commingUrl.query.search){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    else if(commingUrl.pathname === '/products'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      let ans = [];
      for(elem of arr){
        if(elem.Category === commingUrl.query.category){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    else if(commingUrl.pathname === '/filterproducts'){
      let data = fs.readFileSync('./products.json', 'utf-8');
      let arr = JSON.parse(data);
      let ans = [];
      for(elem of arr){
        if(elem.category === commingUrl.query.category && parseFloat(elem.price) >= parseFloat(commingUrl.query.price)){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    res.end();
  }).listen(3001, ()=> {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server");
}










Invalid Request method
Complete the below Stories

Task 1: Create Signup page with "/signup" post endpoint to receive user details like name, gender and age from the html page, store this data in users.txt file, which already contains the users as object with properties like name, gender and age.

Task 2: if request to the endpoint “/signup” was made with get request, return an html content to the user with “Invalid request method - 1” to the user. Where 1 is the total no of invalid requests method, keep on increasing this, and the count on the server for each invalid request method.


const http = require('http');
const QueryString = require('url');
const fs = require('fs')
let cnt = 0;
http.createServer( (req, res) => {
  const comingUrl = QueryString.parse(req.url, true);
  if(comingUrl.pathname === '/'){
    const data = fs.readFileSync('./index.html', 'utf-8');
    res.setHeader('Content-type', 'text/html');
    res.write(data)
  }
  else if(comingUrl.pathname === '/signup'){
    if(req.method === 'POST'){
      let body = '';
      req.on('data', (chunk) => {
        body += chunk;
      })
      req.on('end', () => {
        const formData = JSON.parse(body);
        let arr = fs.readFileSync('users.txt', 'utf-8');
        arr = JSON.parse(arr);
        arr.push(formData);
        fs.writeFileSync('./users.txt', JSON.stringify(arr));
      });
    }
    else if (req.method === 'GET') {
      res.setHeader('Content-type', 'text/plain')
      res.write(Invalid request method - ${cnt++})
    }
  }
  res.end();
}).listen(3000, () => {
  console.log("Server started");
})











Web Server
Complete the below Stories without express (using plain httpserver)
Task 1: Write the endpoint to handle request other /about, /home, All other request to server must be handled by this endpoint
Task 2: Enpoint must store all the other request in a file with Request URL, Date and Time in a file on server with errors.log each entry in one line.
Task 3: if the total lines in the files exceeds 5 remove the last error, the error log must contains max of 5 lastest entries only.


const http = require('http')
const fs = require('fs');
const QueryString = require('url');
http.createServer( (req, res) => {
  const comingUrl = QueryString.parse(req.url);
  if(comingUrl.pathname === '/' || comingUrl.pathname === '/favicon.ico'){
    res.write("No endpoint added means you're in '/' path, add any path except /read")
  }
  else if(comingUrl.pathname === '/read'){
    let arr = fs.readFileSync('./store.json', 'utf-8');
    res.setHeader('Content-type', 'application/json');
    res.write(arr);
  }
  else{
    let d = new Date();
    const obj = {
      url: req.url,
      date: d.toDateString(),
      time: d.toLocaleTimeString()
    }
    let arr = fs.readFileSync('./store.json', 'utf-8');
    arr = JSON.parse(arr);
    while(arr.length >= 5){
      arr.pop();
    }
    arr.push(obj);
    fs.writeFileSync('./store.json', JSON.stringify(arr));
    res.write(You're in ${req.url} and if you want to read data go to /read);
  }
  res.end();
}).listen(3000, () => {
  console.log("Server started...")
})











Basic Static Site -1
Complete the Story without Express Server
Task 1: Create a web server to handle request at port 3001, give proper message at console "Server started..." if actually it started or give "unable to start server" if there is an issue in starting the server.
Task 2: Create a home endpoint “/” to send the home.html page to user.
Task 3: Home.html is using one style.css and one image task.jpg, handle endpoints to send style.css and task.jpg, so that home.html can use them. All other request to any other file should be rejected.



const http = require('http');
const fs = require('fs')
const QueryString = require('url');
try{
  http.createServer( (req, res) => {
    const comingUrl = QueryString.parse(req.url, true);
    console.log(comingUrl)
    if(comingUrl.pathname === '/'){
      const data = fs.readFileSync('./home.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    else if(comingUrl.pathname === '/style.css'){
      const data = fs.readFileSync('./style.css', 'utf-8');
      res.setHeader('Content-type', 'text/css');
      res.write(data);
    }
    else if(comingUrl.pathname === '/task.jpeg'){
      const data = fs.readFileSync('./task.jpeg');
      res.setHeader('Content-type', 'image/jpeg');
      res.write(data);
    }
    else{
      const data = fs.readFileSync('./error.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    res.end();
  }).listen(3001, () => {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server...")
}












Basic Static Site-2
Complete the Story using Express Server
Task 1: Create a logic to handle request to all the pages which actually does not exists, send a response 404.html page to them.
Task 2: Write a code to manage a simple endpoint "/details" which should be called using get method only, and it must contain a variable called id, if id doesn't exists send a message for invalid request, if the it contains a variable check it must have a value, if value is blank send a message "Specify the value" and if all ok send a message "Request received with value of id".
e.g.
1. http://localhost:3000/details
"Invalid Request"
2. http://localhost:3000/details?id=
"Specify the value"
3. http://localhost:3000/details?id=1
"Request received with value 1"



const http = require('http');
const fs = require('fs')
const QueryString = require('url');
try{
  http.createServer( (req, res) => {
    const comingUrl = QueryString.parse(req.url, true);
    if(comingUrl.pathname === '/'){
      const data = fs.readFileSync('./home.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    else if(comingUrl.pathname === '/style.css'){
      const data = fs.readFileSync('./style.css', 'utf-8');
      res.setHeader('Content-type', 'text/css');
      res.write(data);
    } 
    else if(comingUrl.pathname === '/task.jpeg'){
      const data = fs.readFileSync('./task.jpeg');
      res.setHeader('Content-type', 'image/jpeg');
      res.write(data);
    }
    else if(comingUrl.pathname === '/details'){
      if(req.method === 'GET'){
        if(comingUrl.query.id === undefined){
          res.write("Invalid Request");
        }
        else if(comingUrl.query.id.length == 0){
          res.write("Specify the value");
        }
        else{
          res.write(Request received with value ${comingUrl.query.id})
        }
      }
      else{
        res.write("Make the get request only")
      }
    }
    else{
      const data = fs.readFileSync('./error.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    res.end();
  }).listen(3001, () => {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server...")
}













Todo App
Task 1: Create a end point “/addtask” to  receive data from html page like title and id and status, store in into a json file todo.json , the json file contains an array so update the array with new task.
Task 2: Create an endpoint to handle the request “/tasks?status=pending” and return all the tasks from json with status=pending



const http = require('http');
const fs = require('fs')
const QueryString = require('url');
try{
  http.createServer( (req, res) => {
    const comingUrl = QueryString.parse(req.url, true);
    if(comingUrl.pathname === '/'){
      const data = fs.readFileSync('./index.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    else if(comingUrl.pathname === '/style.css'){
      const data = fs.readFileSync('./style.css', 'utf-8');
      res.setHeader('Content-type', 'text/css');
      res.write(data);
    }
    else if(comingUrl.pathname === '/addtask'){
      if(req.method === 'POST'){
        let body = '';
        req.on('data', (chunk) => {
          body += chunk;
        })
        req.on('end', () => {
          const formData = JSON.parse(body);
          let arr = fs.readFileSync('./todo.json', 'utf-8');
          arr = JSON.parse(arr);
          arr.push(formData);
          fs.writeFileSync('./todo.json', JSON.stringify(arr));
        })
      }
      else{
        res.write("Make the post request only")
      }
    }
    else if(comingUrl.pathname === '/tasks'){
      let arr = fs.readFileSync('./todo.json', 'utf-8');
      arr = JSON.parse(arr);
      let ans = [];
      for(elem of arr){
        if(elem.status === comingUrl.query.status){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    else{
      const data = fs.readFileSync('./error.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    res.end();
  }).listen(3000, () => {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server...")
}












TodoApp -1
Task 1: Create a Nodejs server to handle request at port 3001, give proper message at console "Server started..." if actually it started or give "unable to start server" if there is an issue in starting the server.
Task 2: Create an end point “/” that return all the tasks from todo.json file to client. The todo.json is json file contains array of task objects with each object having properties as id, title and status,store some dummy data
Task 3: Create an end point "/delete?id=1" to delete the task with the id=1 and return the pending tasks.



const http = require('http');
const fs = require('fs')
const QueryString = require('url');
try{
  http.createServer( (req, res) => {
    const comingUrl = QueryString.parse(req.url, true);
    if(comingUrl.pathname === '/'){
      const data = fs.readFileSync('./index.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    else if(comingUrl.pathname === '/style.css'){
      const data = fs.readFileSync('./style.css', 'utf-8');
      res.setHeader('Content-type', 'text/css');
      res.write(data);
    }
    else if(comingUrl.pathname === '/addtask'){
      if(req.method === 'POST'){
        let body = '';
        req.on('data', (chunk) => {
          body += chunk;
        })
        req.on('end', () => {
          const formData = JSON.parse(body);
          let arr = fs.readFileSync('./todo.json', 'utf-8');
          arr = JSON.parse(arr);
          arr.push(formData);
          fs.writeFileSync('./todo.json', JSON.stringify(arr));
        })
      }
      else{
        res.write("Make the post request only")
      }
    }
    else if(comingUrl.pathname === '/tasks'){
      let arr = fs.readFileSync('./todo.json', 'utf-8');
      arr = JSON.parse(arr);
      let ans = [];
      for(elem of arr){
        if(elem.status === comingUrl.query.status){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      res.write(JSON.stringify(ans));
    }
    else if(comingUrl.pathname === '/read'){
      let arr = fs.readFileSync('./todo.json', 'utf-8');
      res.setHeader('Content-type', 'application/json');
      res.write(arr);
    }
    else if(comingUrl.pathname === '/delete'){
      let arr = fs.readFileSync('./todo.json', 'utf-8');
      arr = JSON.parse(arr);
      let ans = [];
      for(elem of arr){
        if(elem.id !== comingUrl.query.id){
          ans.push(elem);
        }
      }
      res.setHeader('Content-type', 'application/json');
      fs.writeFileSync('./todo.json', JSON.stringify(ans));
      res.write(JSON.stringify(ans));
    }
    else{
      const data = fs.readFileSync('./error.html', 'utf-8');
      res.setHeader('Content-type', 'text/html');
      res.write(data);
    }
    res.end();
  }).listen(3000, () => {
    console.log("Server started...")
  })
}
catch(err){
  console.log("Unable to start server...")
}









BLOGAPP
const express = require('express');
const multer = require('multer');
const fs = require('fs');       
const path = require('path');

const app = express();
const port = 4000;

app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));

const storage = multer.diskStorage({
    destination: './public/images/',
    filename: function (req, file, cb) {
        cb(null, file.fieldname + '-' + Date.now() + path.extname(file.originalname));
    }
});

const upload = multer({ storage: storage }).single('image');

app.get('/', (req, res) => {
    res.sendFile(__dirname + '/dashboard.html');
});

app.get('/add-article', (req, res) => {
    res.sendFile(__dirname + '/add-article.html');
});

app.post('/upload', (req, res) => {
    upload(req, res, (err) => {
        if (err) {
            console.log(err);
            res.send('Error uploading file.');
        } else {
            const { title, content } = req.body;
            let image = '';
            if (req.file && req.file.filename) {
                image = req.file.filename;
            } else {
                res.status(400).send('No file uploaded.');
                return;
            }

            const article = { title, content, image };
            let articles = [];
            if (fs.existsSync('article.json')) {
                articles = JSON.parse(fs.readFileSync('article.json', 'utf-8'));
            }

            articles.push(article);

            fs.writeFileSync('article.json', JSON.stringify(articles, null, 2));

            res.redirect('/');
        }
    });
});

app.listen(port, () => {
    console.log(Server is running on http://localhost:${port});
});








ATTENDANCE QUESTION
const express = require('express')
const app = express()
const fs = require('fs')
const data = require('./attendance.json')
app.use(express.json())
app.use(express.urlencoded({extended:true}))
app.get('/attendance/:id/:attend',(req,res)=>{
    const id = req.params.id
    const attend = req.params.attend
    const obj = {
        roolno : id,
        attendance : attend
    }
  const x =   data.find(e=>{
        return e.roolno ==  id;
    })
    console.log(x);
    console.log(data)
    console.log(obj);
    if(!x){
        data.push(obj)
    }
    fs.writeFileSync('./attendance.jsonlJSON.stringify(data));
})
app.listen(8080)






const express = require('express');
const multer = require('multer');
const path = require('path');

const app = express();

app.get('/upload', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'upload.html'));
})
// Define storage for uploaded files
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        // Set the destination directory where uploaded files will be stored
        cb(null, 'uploads/');
    },
    filename: function (req, file, cb) {
        // Set the filename for uploaded files
        cb(null, Date.now() + path.extname(file.originalname));
    }
});

// Create multer instance with defined storage
const upload = multer({ storage: storage });
// Route to handle file upload
app.post('/upload', upload.array('file'), (req, res) => {
    // Access uploaded file information
    const uploadedFile = req.file;
    if (!uploadedFile) {
        res.status(400).send('No file uploaded');
    } else {
        res.send('File uploaded successfully');
    }
});

// Serve uploaded files statically
app.use('/uploads', express.static('uploads'));

// Start the server
const PORT = 3000;
app.listen(PORT, () => {
    console.log(Server is running on port ${PORT});
});













const express = require('express');
const multer = require('multer');
const path = require('path');

const app = express();

app.get('/upload', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'upload.html'));
})
// Define storage for uploaded files
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        // Set the destination directory where uploaded files will be stored
        cb(null, 'uploads/');
    },
    filename: function (req, file, cb) {
        // Set the filename for uploaded files
        cb(null, Date.now() + path.extname(file.originalname));
    }
});

// Create multer instance with defined storage
const upload = multer({ storage: storage });
// Route to handle file upload
app.post('/upload', upload.array('file'), (req, res) => {
    // Access uploaded file information
    const uploadedFile = req.file;
    if (!uploadedFile) {
        res.status(400).send('No file uploaded');
    } else {
        res.send('File uploaded successfully');
    }
});

// Serve uploaded files statically
app.use('/uploads', express.static('uploads'));

// Start the server
const PORT = 3000;
app.listen(PORT, () => {
    console.log(Server is running on port ${PORT});
});




LOGINNNN WITHHH MONGOOO


APP.JS


const express = require('express');
const app = express();
app.listen(8080);
const session = require('express-session');

const {MongoClient} = require('mongodb');

app.use(express.static('./public'))

const client = new MongoClient('Enter your uri')

app.use(session({
    secret:'secretkey',
    resave:false,
    saveUninitialized:true
}));

app.get('/login',(req,res)=>{
    res.sendFile('login.html',{root:'./views'});
})

app.get('/register',(req,res)=>{
    res.sendFile('register.html',{root:'./views'});
})

app.get('/home',(req,res)=>{
    res.sendFile('/home.html',{root:'./views'});
})

const userAuth = (req,res,next)=>{
    if(req.session && req.session.user){
        next();
    }
    else{
        res.redirect('/login')
    }
}

app.post('/login',async (req,res)=>{
    await client.connect();
    const user = await client.db('databaseName').collection('collection').findOne({username:req.body.username, password:req.body.password})
    await client.close();
    
    if(!user){
        res.status(401).send('Invalid Username or password');
    }else{
        res.session.user = user;
        res.redirect('/home');
    }
})

app.post('/register',async(req,res)=>{
    await client.connect();
    const result = await client.db('databaseName').collection('collection').insertOne({
        fullname:req.body.fullname,
        username:req.body.username,
        password:req.body.password,
        email:req.body.email
    });

    await client.close();

    if(result.acknowledged){
        res.redirect('/login')
    }else{
        res.status(400).send('Something went wrong');
    }
})
















MONGOOSE BLOGGGG SITEEEE







APP.JS



const express = require('express');
const app = express();
const session = require('express-session');
const loginRouter = require('./src/routes/login');
const authorRouter = require('./src/routes/author');
const adminRouter = require('./src/routes/admin');
const articleRouter = require('./src/routes/articles');
const homeRouter = require('./src/routes/home');
const mongoose = require('mongoose');
app.use(express.urlencoded());
app.use(express.json());
require('dotenv').config();

mongoose.connect(process.env.URI)

app.listen(process.env.PORT);

app.set('view engine','ejs');
app.set('views','./src/views')

app.use(session({
    resave:false,
    saveUninitialized:true,
    secret:'keyboard'
}));


app.use(express.static('./public'))

app.use('/',loginRouter);

const userAuth = (req,res,next)=>{
    
    if(req.session && req.session.user){
        next()
    }
    else{
        res.redirect('/login');
    }
}

const adminAuth = (req,res,next)=>{
    if(req.session.user.role=='admin'){
        next();
    }
    else{
        res.send('Invalid access')
    }
}

const authorAuth = (req,res,next)=>{
    if(req.session.user.role=='author'){
        next();
    }
    else{
        res.send('Invalid access')
    }
}
app.use('/',homeRouter);
app.use('/admin',userAuth,adminAuth,adminRouter);
app.use('/author',userAuth,authorAuth,authorRouter)
app.use('/article',userAuth,articleRouter);


















EXPRESS(FORMS, FETCH)




SERVER.JS



const express=require("express");
const app=express();

app.use(express.urlencoded({extended:false}));
//these middleware is used when application/json data come
app.use(express.json())

app.get("/",(req,res)=>{
    res.sendFile(__dirname+"/login.html");
})

app.get("/add",(req,res)=>{
    console.log(req.query);
    res.send("ok");
})

app.post("/add",(req,res)=>{
    console.log(req.body);
    res.send("done");
})

app.listen(3000);


















EXPRESS(LOGIN, SIGNUP, DASHBOARD)



SERVER.JS

const express=require("express");
const app=express();
const fs=require("fs");
const multer=require("multer");
app.use(express.static(__dirname+"/public"));
const upload=multer({dest:__dirname+"/public"});

app.use(express.urlencoded({extended:false}));
app.use(express.json());

app.get("/",(req,res)=>{
    res.sendFile(__dirname+"/admin.html");
})
app.post("/upload",upload.single("pic"),(req,res)=>{
const {name,price,desc,id}=req.body;
const image=req.file.filename;
const obj={name,price,id,desc,image};
fs.readFile(__dirname+"/product.json","utf-8",(err,data)=>{
    data=JSON.parse(data);
    data.push(obj);
    fs.writeFileSync(__dirname+"/product.json",JSON.stringify(data));
    res.redirect("/");
})
})

app.get("/login",(req,res)=>{
    res.sendFile(__dirname+"/login.html");
})
app.get("/signup",(req,res)=>{
    res.sendFile(__dirname+"/signup.html");
})
app.post("/login",(req,res)=>{
   let {pass,email}=req.body;
   fs.readFile(__dirname+"/user.json","utf-8",(err,data)=>{
    if(err){
        res.status(500).send("Internal server error");
    }else{
        data=JSON.parse(data);
        let flag=false;
        data.forEach(element => {
            if(element.pass==pass&&element.email==email){
                flag=true;
            }
        });
        if(flag){
            res.sendFile(__dirname+"/dashboard.html");
        }else{
            res.redirect("/signup");
        }
    }
   })
})
app.post("/signup",(req,res)=>{
    let {email}=req.body;
   fs.readFile(__dirname+"/user.json","utf-8",(err,data)=>{
    if(err){
        res.status(500).send("Internal server error");
    }else{
        data=JSON.parse(data);
        let flag=false;
        data.forEach(element => {
            if(element.email==email){
                flag=true;
            }
        });
        if(flag){
          res.send("email already exist");
        }else{
            data.push(req.body);
            fs.writeFileSync(__dirname+"/user.json",JSON.stringify(data));
            res.redirect("/login");
        }
    }
   })
})

app.get("/getAllProduct",(req,res)=>{
    res.sendFile(__dirname+"/product.json");
})

app.get("/showdesc/:id",(req,res)=>{
    fs.readFile(__dirname+"/product.json","utf-8",(err,data)=>{
        if(err){
            res.status(500).send("Internal server error");
        }else{
            data=JSON.parse(data);
           let data1=data.filter(element => {
                if(element.id==req.params.id){
                   return true;
                }
            });
           res.send(data1[0].desc);
        }
       })
})
app.listen(3000);























EXPRESS(MULTER)



SERVER.JS





const express=require("express");
const app=express();
const multer=require("multer");

const maxSize=1*1024*1024 //1 MB

const storage=multer.diskStorage({
    destination:(req,file,cb)=>{
        console.log(file);
    //     if(user=="admin")
    //     cb(null,__dirname+"/public/admin");
    // else{
        cb(null,__dirname+"/done/files");
    // }
    },
    filename:(req,file,cb)=>{
        var name=Date.now()+".jpg";
        cb(null,name);
    }

})
const filter=(req,file,cb)=>{
  var ext=file.mimetype.split("/")[1];
  if(ext=="jpeg"||ext=="jpg"){
    cb(null,true);
  }else{
    cb(new Error("not supported"),false);
  }
}



const upload=multer({storage:storage,fileFilter:filter,limits:{fileSize:maxSize}});
// const upload=multer({dest:__dirname+"/public/files"});

app.get("/",(req,res)=>{
    res.sendFile(__dirname+"/upload.html");
})

//for handle multiple fields
app.post("/filelds",upload.fields([{name:"pic",maxCount:1},{name:"pic1",maxCount:1}]),(req,res)=>{
  console.log(req.body);
  console.log(req.files);
    res.end("multiple fields recived");
})

app.post("/upload",upload.single("pic"),(req,res)=>{
  console.log(req.body);
  console.log(req.file);
    res.end("file recived");
})
//for handle multiple file
app.post("/multiple",upload.array("pic",5),(req,res)=>{
  console.log(req.body);
  console.log(req.files);
    res.end("multiple files recived");
})


app.post("/withoutimage",upload.single("image"),(req,res)=>{
  console.log(req.body);
  const image = req.file ? req.file.filname : '';
  console.log(image); 
})

app.post("/none",upload.none(),(req,res)=>{
  console.log(req.body);
    res.end("data recived");
})



app.listen(3000);
















EXPRESS(ROUTING USERDEFINEMIDDLEWARE)




SERVER.JS





const express = require("express");
const app = express();
const route = require(__dirname + "/route.js");

app.use("/", route);
//const cors=require("cors");

//to allow cross origin we use middleware
// app.use(cors());

//user define middleware
app.use((req, res, next) => {
    var date = Date.now();
    console.log(date);
    next();
})

app.use((req, res, next) => {
    console.log("hello world", req.url);
    next();
})

app.get("/", (req, res) => {
    res.send("hello world");
})

//route based middleware

app.get("/abc", (req, res, next) => {
    console.log("route based middleware");
    next();
}, (req, res, next) => {
    res.send("abc");
})

function fun(req, res, next) {
    console.log("function based middleware");
    next();
}
//route based middleware through function
app.get("/def", fun, (req, res, next) => {
    res.send("def");
})


//to get data using params 
//dynamic routing
app.get("/name/:name/id/:id", (req, res) => {
    console.log(req.params);
    res.end();
})
app.listen(3000, (err) => {
    if (err)
        console.log(err);
    else
        console.log("server started at port 3000");
})



//routing
//params













MULTER





SERVER.JS





const express=require('express')
const app=express()
const multer=require('multer')
const storage = multer.diskStorage({
    destination:(req,file,cb)=>{
        console.log(file);
        cb(null,__dirname+'/public');
    },
    filename:(req,file,cb)=>{
        cb(null,Date.now()+'.jpg')
    }
})

const filter = (req,file,cb)=>{
    if(file.mimetype.split('/')[1]=='jpg' || file.mimetype.split('/')[1]=='jpeg' || file.mimetype.split('/')[1]=='html' )
        cb(null,true)
    else 
        cb(new Error('Invalid File Extension !'),false)
}

const upload = multer({storage:storage, fileFilter: filter, limits:{fileSize:1024*1024*4} });

app.get('/reel',(req,res)=>{
    res.sendFile(__dirname+'/upload.html')
})

app.post('/reel',upload.single("img"),(req,res)=>{
    console.log(req.body);
    console.log(req.file);
    res.send('UPLOADED !!')
})

// app.get('/view',(req,res)=>{
//     res.render()
// })

app.listen(1000,()=> console.log('started'));














MONGODB


SERVER.JS

const express=require("express");
const app=express();
//npm i mongodb
const mongodb=require("mongodb"); //multiple classes
const client=mongodb.MongoClient;
const object=mongodb.ObjectId;
app.set("view engine","ejs");
app.use(express.urlencoded({extended:false}));
let dbinstance;
app.use((req,res,next)=>{
    console.log(req.url);
    next();
})
client.connect("mongodb://127.0.0.1:27017").then((database)=>{
    console.log("connected")
   dbinstance=database.db("sectionE");
}).catch(e=>{
    console.log(e);
})



app.get("/show/:id",(req,res)=>{
    console.log(req.params.id);
    dbinstance.collection("user").findOne({_id:new object(req.params.id)}).then(data=>{
        // console.log(data);
        res.send(data);
    })
})

app.get("/",(req,res)=>{
    dbinstance.collection("product").find({}).toArray().then(data=>{
         console.log(data);
        res.render("dashboard",{data});
    })
})

app.get("/view/:id",(req,res)=>{
    dbinstance.collection("product").findOne({_id:new object(req.params.id)}).then(data=>{
        // console.log(data);
        res.render("view",{data});
    })
})

app.get("/delete/:id",(req,res)=>{
    dbinstance.collection("product").findOne({_id:new object(req.params.id)}).then(data=>{
        // console.log(data);
        res.render("delete",{data});
    })
})

app.get("/update/:id",(req,res)=>{
    dbinstance.collection("product").findOne({_id:new object(req.params.id)}).then(data=>{
         console.log(data);
        res.render("update",{data});
    })
})

app.post("/confirmupdate",(req,res)=>{
    console.log(req.url);
    dbinstance.collection("product").updateOne({_id:new object(req.body.id)},{$set:{"price":req.body.price}}).then(data=>{
        // console.log(data);
        res.redirect("/");
    }) 
})


app.get("/confirmdelete/:id",(req,res)=>{
    dbinstance.collection("product").deleteOne({_id:new object(req.params.id)}).then(data=>{
        // console.log(data);
        res.redirect("/");
    })
})


app.listen(3000,(err)=>{
    if(err)
    console.log(err);
else
console.log("server running");
});












SESSION










SERVER.JS





const express=require("express");
const app=express();
const session=require("express-session");

app.use(session({
    saveUninitialized:true,
    resave:false,
    secret:"abc",
}))
app.use(express.urlencoded({extended:false}));

app.get("/login",(req,res)=>{
    if(req.session.logedin){
        res.redirect("/");
    }else{
      res.sendFile(__dirname+"/login.html");
    }
})
app.post("/login",(req,res)=>{
    console.log(req.body)
})

app.get("/",(req,res)=>{
    if(req.session.logedin){
        res.send("welcome home page");
    }else{
       
    res.redirect("/login");
    }
   
})

app.listen(3000);















READ AND WRITE



INDEX.JS
const http = require('http');
const fs = require('fs');
const path = require('path');
const url = require('url');
http.createServer((req,res)=>{
    const urllink = req.url;
    const data = url.parse(urllink,true);    
    let fname = data.query.fname;
    
    if(req.url=='/product'){
        res.setHeader('Content-Type','application/json');
        res.setHeader('Access-Control-Allow-Origin', '*'); 
        res.setHeader('Access-Control-Allow-Methods', 'OPTIONS, GET');
        res.statusCode=200;
        res.statusMessage = 'Ok';
        let data = fs.readFileSync('./products.json','utf-8');
        
        res.write(data);
        res.end();
    }
    else{
        res.setHeader('Content-Type','text/plain');
        res.statusCode=404;
        res.statusMessage = 'Not found';
        res.end();
    }
}).listen(3030);










SERVER.JS








const http = require('http');
const fs = require('fs');
const path = require('path');
const url = require('url');

http.createServer((req,res)=>{
    res.setHeader('Content-Type','application/json');
    res.setHeader('Access-Control-Allow-Origin', '*'); 
    res.setHeader('Access-Control-Allow-Methods', 'OPTIONS, GET');

    let path = url.parse(req.url,true);

    if(path.pathname=='/product'){     
        
        res.statusCode=202;
        res.statusMessage = 'Ok';
        let data = fs.readFileSync('./products.json','utf-8'); 
        let products = JSON.parse(data);
        let result = products.filter(product=>{
            return product.name.toUpperCase() == path.query.name.toUpperCase()
        })            
        res.write(JSON.stringify(result));
        res.end();
    }
    else{
        res.setHeader('Content-Type','text/plain');
        res.statusCode=404;
        res.statusMessage = 'Not found';
        res.end();
    }
}).listen(3030)












AUTHENTICATION / AUTHORIZATION













SERVER.JS
const express=require("express");
const app=express();
const fs=require("fs");
const cookie=require("cookie-parser");
const session=require("express-session");

app.use(cookie());
app.use(session({
    saveUninitialized:true,
    resave:false,
    secret:"abc"
}))

app.use(express.urlencoded({extended:false}));
app.use(express.json());

function check(req,res,next){
    if(req.session.username)
    next();
   else
   res.redirect("/login");
}

app.get("/",check,(req,res)=>{
    res.sendFile(__dirname+"/dashboard.html");
})

app.get("/home",check,(req,res)=>{
    res.sendFile(__dirname+"/home.html");
})

function auth(req,res,next){
    if(req.session.username&&req.session.role=="admin")
    next();
   else
   res.redirect("/");
}

app.get("/admin",auth,(req,res)=>{
    res.sendFile(__dirname+"/admin.html");
})

app.get("/login",(req,res)=>{
    if(req.session.username)
    res.redirect("/");
else
    res.sendFile(__dirname+"/login.html");
})
app.get("/signup",(req,res)=>{
    if(req.session.username)
    res.redirect("/");
else
    res.sendFile(__dirname+"/signup.html");
})
app.post("/login",(req,res)=>{
   let {pass,email}=req.body;
   fs.readFile(__dirname+"/user.json","utf-8",(err,data)=>{
    if(err){
        res.status(500).send("Internal server error");
    }else{
        data=JSON.parse(data);
        // let flag=false;
        // data.forEach(element => {
        //     if(element.pass==pass&&element.email==email){
        //         flag=true;
        //     }
        // });
        let result=data.filter((element)=>{
            if(element.pass==pass&&element.email==email){
                       return true;
                    }
        })
        if(result.length>0){
            req.session.username=email;
            req.session.role=result[0].role;
           res.redirect("/");
        }else{
            res.redirect("/signup");
        }
    }
   })
})
app.post("/signup",(req,res)=>{
    let {email}=req.body;
   fs.readFile(__dirname+"/user.json","utf-8",(err,data)=>{
    if(err){
        res.status(500).send("Internal server error");
    }else{
        data=JSON.parse(data);
        let flag=false;
        data.forEach(element => {
            if(element.email==email){
                flag=true;
            }
        });
        if(flag){
          res.send("email already exist");
        }else{
            let obj={
               name:req.body.name,
               email:req.body.email,
               pass:req.body.pass,
               role:"user"
            }
            data.push(obj);
            fs.writeFileSync(__dirname+"/user.json",JSON.stringify(data));
            res.redirect("/login");
        }
    }
   })
})

app.get("/logout",(req,res)=>{
    req.session.destroy();
    res.redirect("/login");
})

app.listen(3000);


























HTTP (FORM)

SERVER.JS


const http = require("http");
const fs = require("fs");
const url = require("url");
const server = http.createServer((req, res) => {
    console.log(req.url);
    console.log(req.method);
    let urlparsed = url.parse(req.url, true);
    if (req.url == "/login" || req.url == "/") {
        fs.readFile(__dirname + "/login.html", "utf-8", (err, data) => {
            if (err) {
                res.writeHead(302, { 'Content-Type': 'text/plain' })
                res.write("not");
            }
            else {
                res.writeHead(200, { 'Content-Type': 'text/html' })
                res.write(data);
            }
            res.end();
        })
    }
    else if (req.url == "/signup" || req.url == "/") {
        fs.readFile(__dirname + "/signup.html", "utf-8", (err, data) => {
            if (err) {
                res.writeHead(302, { 'Content-Type': 'text/plain' })
                res.write("not");
            }
            else {
                res.writeHead(200, { 'Content-Type': 'text/html' })
                res.write(data);
            }
            res.end();
        })
    }
    else if (urlparsed.pathname == "/check") {
        console.log(urlparsed.query);
        fs.readFile(__dirname + "/user.json", "utf-8", (err, data) => {
            if (err) {
                res.writeHead(302, { 'Content-Type': 'text/plain' })
                res.write("not");
                res.end();
            }
            else {
                data = JSON.parse(data);
                if (data.length == 0) {
                    data = [];
                }
                let flag = false;
                data.forEach(element => {
                    if (element.username == urlparsed.query.username && element.password == urlparsed.query.password) {
                        flag = true;
                    }
                });
                if (flag)
                    res.write("user exist");
                else {
                    res.write("not exist");
                }
                res.end();
            }
        })
    }
    else if (urlparsed.pathname == "/add") {
        console.log(urlparsed.query);
        fs.readFile(__dirname + "/user.json", "utf-8", (err, data) => {
            if (err) {
                res.writeHead(302, { 'Content-Type': 'text/plain' })
                res.write("not");
            }
            else {
                data = JSON.parse(data);
                if (data.length == 0) {
                    data = [];
                }
                let flag = false;
                data.forEach(element => {
                    if (element.username == urlparsed.query.username && element.password == urlparsed.query.password) {
                        flag = true;
                    }
                });
                if (flag) {
                    res.writeHead(200, { 'Content-Type': 'text/plain' })
                    res.write("already exist");
                    res.end();
                }
                else {
                    data.push(urlparsed.query);
                    fs.writeFile(__dirname + "/user.json", JSON.stringify(data), (err) => {
                        if (err) {
                            res.writeHead(302, { 'Content-Type': 'text/plain' })
                            res.write("error");
                        }
                        else {
                            res.writeHead(200, { 'Content-Type': 'text/plain' })
                            res.write("user created succ");
                        }
                        res.end();
                    })
                }
            }
        })
    }
    else {
        res.writeHead(404, { 'Content-Type': 'text/plain' });
        res.write("Page not found");
        res.end();
    }
});
server.listen(3000, (err) => {
    if (err)
        console.log(err)
    else
        console.log("server is runnning at 3000");
})

//now complete the logic delete by your own




























HTTP(FETCH)




SERVER,JS



const http = require("http");
const fs = require("fs");
const path = require("path");
const url = require("url");
const server = http.createServer((req, res) => {
    const urlparsed = url.parse(req.url, true);
    if (req.url == "/") {
        fs.readFile(path.join(__dirname, "home.html"), "utf-8", (err, data) => {
            if (err) {
                res.writeHead(302, { 'Content-Type': 'text/plain' })
                res.write("not");
            }
            else {
                res.writeHead(200, { 'Content-Type': 'text/html' })
                res.write(data);
            }
            res.end();
        })
    }
    else if (req.url == "/submit") {
        res.write(JSON.stringify({ name: "abc" }));
        res.end();
    }
    else if (urlparsed.pathname == "/add" && req.method == "GET") {
        console.log(urlparsed.query);
        res.write("done");
        res.end();
    }
    else if (req.url == "/add" && req.method == "POST") {
        let responseData = "";
        req.on('data', (chunk) => {
            responseData += chunk;
        });

        req.on('end', () => {
            console.log(JSON.parse(responseData));
            res.end("post done");
        })
    }
});
server.listen(3000, (err) => {
    if (err)
        console.log(err)
    else
        console.log("server is runnning at 3000");
})

















PATH


SERVER.JS





const express=require("express");
const app=express();
const session=require("express-session");

app.use(session({
    saveUninitialized:true,
    resave:false,
    secret:"abc",
}))
app.use(express.urlencoded({extended:false}));

app.get("/login",(req,res)=>{
    if(req.session.logedin){
        res.redirect("/");
    }else{
      res.sendFile(__dirname+"/login.html");
    }
})
app.post("/login",(req,res)=>{
    console.log(req.body)
})

app.get("/",(req,res)=>{
    if(req.session.logedin){
        res.send("welcome home page");
    }else{
       
    res.redirect("/login");
    }
   
})

app.listen(3000);




























MONGOOSEE

INDEX.JS

const {MongoClient} = require('mongodb');

const client = new MongoClient('mongodb+srv://user:user123@chat.5hlcps0.mongodb.net/?retryWrites=true&w=majority&appName=chat')

async function run(){
    await client.connect();
    const db = client.db('MovieDataBase');
    const collection = await db.collection('MoviesDetail').insertOne(
        {
            "id":14,
            "movie":"Forrest Gump",
            "rating":{"$numberDouble":"8.8"},
            "image":"images/forrest_gump.jpg",
            "imdb_url":"https://www.imdb.com/title/tt0109830/"
        }
    );
    // const data = await collection.find({}).toArray();
    console.log(collection);   
    
}
run();












APP.JS


const mongoose = require('mongoose');
const MoviesDetail = require('./src/model/MoviesDetail')
mongoose.connect('mongodb+srv://user:user123@chat.5hlcps0.mongodb.net/MovieDataBase?retryWrites=true&w=majority&appName=chat');

async function run(){
    // const data = await MoviesDetail.where('id').eq(1);
    const data = await MoviesDetail.create({
        id:13,
        name:'Robot',
        rating:9.9,
        imdb_url:'https://www.imdb.com',
        image:'image url'
    })
    console.log(data);
}

run()









CRUD MONGODB


SERVER.JS







const express=require("express");
const app=express();
const multer=require("multer");
const session=require("express-session");
// const upload=multer({dest:__dirname+"/public"});
const storage=multer.diskStorage({
    destination:(req,file,cb)=>{
cb(null,__dirname+"/public");
    },
    filename:(req,file,cb)=>{
cb(null,Date.now()+".jpg");
    }
})

app.use(session({
    saveUninitialized:true,
    resave:false,
    secret:"abc",
}))

const filter=(req,file,cb)=>{
cb(null,true);
}
const upload=multer({storage:storage,fileFilter:filter});
const mongodb=require("mongodb");
const client=mongodb.MongoClient;
let instance;
client.connect("mongodb+srv://Rakhi:abc123abc@cluster0.qzwljbc.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0").then(databases=>{
instance=databases.db("SectionE");
console.log("connected");
})

app.use(express.urlencoded({extended:false}));
app.get("/",(req,res)=>{
    if(req.session.name)
    res.send("hello");
else
res.redirect("/login");
})
app.get("/login",(req,res)=>{
    if(req.session.name)
    res.redirect("/");
else
    res.sendFile(__dirname+"/login.html");
})

app.post("/login",(req,res)=>{
instance.collection("user").find(req.body).toArray().then(data=>{
  if(data.length>0){
    req.session.name=data[0].name;
    res.redirect("/");
  } else{
    res.redirect("/signup");
  } 
})
})


app.get("/signup",(req,res)=>{
    if(req.session.name)
    res.redirect("/");
else
    res.sendFile(__dirname+"/signup.html");
})

app.post("/signup",upload.single("pic"),(req,res)=>{
instance.collection("user").find({name:req.body.name}).toArray().then(async (data)=>{
  if(data.length>0){
    res.send("already exist");
  } else{
    const image = req.file ? req.file.filename : '';
   let result=await instance.collection("user").insertOne({name:req.body.name,email:req.body.email,password:req.body.password,image:image})
   res.redirect("/login");
  } 
})
})

app.get("/logout",(req,res)=>{
    delete req.session.name;
  //  req.session.destroy();
  res.redirect("/login");
})

app.listen(3000);


















EJS
APP.JS


const express = require('express');
const app = express();
const session = require('express-session');
const bodyparser = require('body-parser');
const userRouter = require('./src/routes/user');
const adminRouter = require('./src/routes/admin');
app.listen(8080);

app.use(bodyparser.urlencoded({extended:true}));
app.use(bodyparser.json())

app.use(session({
    secret:'secret',
    resave: false,    
    saveUninitialized:true
}))

const users = [
    {username:'abc',pass:'abc',role:'admin'},
    {username:'xyz',pass:'xyz',role:'user'}
]

const userAuth = (req,res,next)=>{
    if(req.session && req.session.user){
        next();
    }
    else{
        res.redirect('/login');
    }    
}

const authorisation = (req,res,next)=>{
    if(req.session.user.role=='admin'){
        next();
    }
    else{
        res.send('Invalid access')
    }
}

app.use('/admin',userAuth,authorisation,adminRouter);
app.use('/user',userAuth,userRouter);

app.post('/login',(req,res)=>{
   const user = users.find(user=>req.body.name==user.username && req.body.pass == user.pass);
   if(user){
    req.session.user=user;
    res.redirect('/');
   }
   else{
    res.send('invalid username or password');
   }
})

app.get('/',(req,res)=>{
    res.sendFile('home.html',{root:'./public'});
})


app.get('/login',(req,res)=>{
    res.sendFile('login.html',{root:'./public'});
});

app.get('/logout',(req,res)=>{
    req.session.destroy();
    res.redirect('/');
})

SERVER.JS
const express = require('express');
const app = express();
const fs = require('fs');
app.listen(8080);
app.set('view engine','ejs');
app.use(express.static('./public'))

app.get('/',(req,res)=>{
    const users = JSON.parse(fs.readFileSync('./users.json')).users;
    res.render('home.ejs',{users});
})











ROUTES



APP.JS
const express = require('express');
const app = express();
app.listen('8080');

const htmlRoute = require('./src/routes/htmlroute');

app.use('/home',htmlRoute); 












ROUTING




INDEX.JS

const http = require('http');
const fs = require('fs')
const server = http.createServer((req,res)=>{
    if(req.url=='/about'){
        res.setHeader('Content-Type','text/html')
        let data = fs.readFileSync('./view/about.html','utf-8')
        res.write(data)
        res.end(); 
    }
    else if(req.url=='/index.html'){
        res.setHeader('Content-Type','text/html')
        let data = fs.readFileSync('./view/home.html','utf-8')
        res.write(data)
        res.end(); 
    }
    else if(req.url=='/contact'){
        res.setHeader('Content-Type','text/html')
        let data = fs.readFileSync('./view/home.html','utf-8')
        res.write(data)
        res.end(); 
    }
    else{
        res.setHeader('Content-Type','text/html')
        res.write('<b>Page Not Found</b>');
        res.end(); 
    }
    
})

server.listen(3030,()=>{
    console.log('server started')
})










SERVER.JS




const http = require('http');
const fs = require('fs')
const path = require('path');

http.createServer((req,res)=>{       
   sendResponse(req.url,res);
}).listen(3030,()=>{
    console.log('server started')
})

function sendResponse(url,response){
    try{
    const extName = path.extname(url);
    let header = 'text/html'
    let filePath =''
    if(extName == '.css'){
        header = 'text/css' 
        filePath = path.join(__dirname,'styles',url) //./styles/style.css      
    } 
    else if(extName == '.jpg'){
        header = 'image/jpg' 
        filePath = path.join(__dirname,'images',url) //   
    }   
    else{
        filePath = path.join(__dirname,'view',url);
    }       
    response.setHeader('Content-Type',header);
    let data = fs.readFileSync(filePath,'utf-8')
    response.write(data);
    response.end();
    }
    catch{
        response.end();
        return '';
        
    }
}



















MULTER



INDEX.JS

const express = require('express');
const app = express();
const multer = require('multer');
const path = require('path')
app.listen(8080);
const storage = multer.diskStorage({
    destination:'./public/upload',
    filename:function(req,file,cb){
        const name = Date.now()+"-"+path.extname(file.originalname);
        cb(null,name);
    }
})
const size = 1024*1024*5;
const upload = multer(
    {
        storage:storage,
        limits:{fileSize:size},
        fileFilter:function(req,file,cb){
            if(file.mimetype=='image/jpg'){
                cb(null,true)
            }
            else{
                cb(new Error('Invalid mime type'))
            }
        }
    }
    )

// app.post('/upload',upload.single('fileUpload'),(req,res)=>{
//     console.log(req.file)
//     res.send('file uploaded successfully')
// })

app.post('/upload',upload.array('fileUpload',10),(req,res)=>{
    console.log(req.files);
    res.send('file uploaded successfully')
})

// const fieldDetails = [{name:'imageUpload',maxCount:1},{name:'fileUpload',maxCount:20}];
// app.post('/upload',upload.fields(fieldDetails),(req,res)=>{
//     console.log(req.files);
//     res.send('file uploaded successfully')
// })
















EXPRESS

APP.JS

const express = require('express');
const app = express();
app.use(express.static('public'));
app.use('cors')

app.get('/home.html',(req,res)=>{
    res.sendFile('./view/home.html',{root:__dirname});
});

app.get('/about.html',(req,res)=>{
    res.sendFile('./view/about.html',{root:__dirname});
})

app.get('/contact.html',(req,res)=>{
    res.sendFile('./view/contact.html',{root:__dirname});
})

app.listen(8080,(err)=>{
    if(err) throw err;
    console.log('Server started at 8080');
})



SERVER.JS



const http = require('http');
const fs = require('fs');
http.createServer(async (req,res)=>{
    if(req.url == '/demo.json'){
        let data = '';
        await req.on('data',(chunk)=>{
                data+=chunk;
        });        
        const demoData = JSON.parse(fs.readFileSync('./demo.json','utf-8')); 
        data = JSON.parse(data);
        const filteredData = demoData.filter((demo)=>
        {                       
            return data.roll == demo.roll
        }
        )
        res.setHeader('Content-Type','application/json');
        res.setHeader('Access-Control-Allow-Origin','*')
        res.end(JSON.stringify(filteredData));
    }
}).listen(8080)








SESSION

APP.JS


const express = require('express');
const app = express();
const session = require('express-session');
const bodyparser = require('body-parser');
app.listen(8080);

app.use(bodyparser.urlencoded({extended:true}));
app.use(bodyparser.json())

app.use(session({
    secret:'secret',
    resave: false,    
    saveUninitialized:true
}))

app.get('/login',(req,res)=>{
    res.sendFile('login.html',{root:'./public'});
})

const userAuth = (req,res,next)=>{
    console.log(req.body)
    if(req.body.name=='abc' && req.body.pass=='abc'){
        next()
    }
    else{
        res.send('Invalid username or password')
    }
}

app.post('/login',userAuth,(req,res)=>{
    res.send('valid user');
})

// var count = 0;
// app.get('/view',(req,res)=>{
//    if(req.session.count){
//         req.session.count++;
//    } 
//    else{
//     req.session.count = 1;
//    }
//    console.log(req.session)
//    res.send(Count is ${req.session.count} <br> ${req.session.id});
// })















EVENTS



INDEX.JS
const fs = require('fs');
const path = require('path');
fs.readdir('./',(err,files)=>{
    if(err) throw err;
    console.log(files);
    const filePath = path.join(__dirname,files[0])
})



EVENT.JS


const event = require('events');
const emiter = new event();
const http = require('http');
const server = http.createServer((req,res)=>{

});


server.listen(3030);

emiter.on('logeMessage',()=>{
    console.log('event Listened');
})

emiter.on('connect',()=>{
    console.log('Connected')
})
emiter.emit('logeMessage');











DYNAMIC ROUTING








const express = require('express');
const app = express();
const bodyparser = require('body-parser');
app.listen(8080);

// app.use(bodyparser.json());
// app.use(bodyparser.urlencoded({extended:true}));

app.use((req,res,next)=>{
    console.log(req.hostname);
    const time = new Date()
    console.log(time)
    next();
})

app.get('/user/:name/:email',(req,res)=>{
    console.log(req.params);
    res.send(req.params)
})

app.use((req,res,next)=>{
    console.log(req.url);
    const time = new Date()
    console.log(time)
    next();
})













MONGOOSE CRUD



APP.JS





const express = require('express');
const app = express();
app.listen(8080);
const mongoose = require('mongoose');
app.use(express.static('./public'))

app.set('view engine','ejs');
app.set('views','./src/views')

const uri = 'mongodb+srv://demoUser:user123@cluster0.kjhkths.mongodb.net/Employee?retryWrites=true&w=majority&appName=Cluster0';
mongoose.connect(uri);

const employeesRoute = require('./src/routes/employees');

app.use('/emp',employeesRoute);

















MONGOOSE





SERVER.JS


const express=require("express");
const app=express();
const session=require("express-session");
const multer=require("multer");
const abc=multer.diskStorage({
    destination:(req,file,cb)=>{
        cb(null,__dirname+"/public");
    },filename:(req,file,cb)=>{
        cb(null,Date.now()+".jpg");
    }
})

const filter=(req,file,cb)=>{
    if(file.mimetype.split("/")[1]=="jpg")
        cb(null,true)
    else
    cb(new Error("not supported"),false)
}


const upload=multer({storage:abc,fileFilter:filter,limits:{fieldSize:1024*1024}})
//const upload=multer({dest:__dirname+"/public"});
//npm i mongoose
 const model=require("./models.js");
//  const product=require("./models1.js");
const mongoose=require("mongoose");
mongoose.connect("mongodb+srv://Rakhi:abcd123abcd@cluster0.qzwljbc.mongodb.net/user?retryWrites=true&w=majority&appName=Cluster0").then(res=>{
    console.log("connected");
}).catch(e=>{
    console.log(e);
})



function authentication(req,res,next){
    if(req.session.logedin){
res.redirect("/home");
    }else{
        next();
    }
}


app.use(session({
    saveUninitialized:true,
    resave:false,
    secret:"abc"
}))

app.use((req,res,next)=>{
    console.log(req.session);
    next();
})
app.use(express.urlencoded({
    extended:false,
}))



function authentication1(req,res,next){
    if(req.session.logedin){
       next();
            }else{
              res.redirect("/login");
            }
}

app.get("/home",authentication1,(req,res)=>{
    res.send("its home page");
})

app.get("/login",authentication,(req,res)=>{
    res.sendFile(__dirname+"/login.html");
})

app.get("/signup",authentication,(req,res)=>{
    res.sendFile(__dirname+"/signup.html");
})

app.post("/login",(req,res)=>{
    model.findOne({name:req.body.username,password:req.body.password}).then(data=>{
        if(data){
            req.session.logedin=true;
            req.session.detail=data;
            res.redirect("/home");
        }else{
            res.redirect("/signup");
        }})
})

app.post("/signup",(req,res)=>{
    model.findOne({name:req.body.username,password:req.body.password}).then(data=>{
        if(data){
            res.redirect("/login");
        }else{
            let obj={
                name:req.body.username,password :req.body.password,
            }
            let newuser=new model(obj); 
            newuser.save().then(data=>{
                res.send("user craeted succ");
            }).catch(e=>{
                res.send("something went wrong");
            })
        }
    })
})


app.listen(3000);






HTTP(CROSSORIGIN)





SERVER.JS





const http = require('http');
const fs = require('fs');
//const path = require('path');
const url = require('url');

const server=http.createServer((req,res)=>{
    console.log("done");
    res.setHeader('Content-Type','application/json');
    res.setHeader('Access-Control-Allow-Origin', '*'); 
    res.setHeader('Access-Control-Allow-Methods', 'OPTIONS, GET');

    let path = url.parse(req.url,true);

    if(path.pathname=='/product'){     
        
        res.statusCode=200;
        res.statusMessage = 'Ok';
        let data = fs.readFileSync(__dirname+'/product.json','utf-8'); 
        let products = JSON.parse(data);
        let result = products.filter(product=>{
            return product.name == path.query.name
        })            
        res.write(JSON.stringify(result));
        res.end();
    }
    else{
        res.setHeader('Content-Type','text/plain');
        res.statusCode=404;
        res.statusMessage = 'Not found';
        res.end();
    }
})
server.listen(8080,(err)=>{
    if(err)
    console.log(err)
else
console.log("server started");
})
```