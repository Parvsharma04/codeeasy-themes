# CodeEasy Themes - Visual Studio Code Theme Extension

<img src="https://github.com/Parvsharma04/codeeasy/blob/master/logo.png?raw=true" width="100px"/>

## Overview

Welcome to CodeEasy Themes, a collection of visually pleasing and easy-to-read themes for Visual Studio Code. This extension provides a variety of carefully crafted color schemes to enhance your coding experience and make your code more readable.


## Multer

const express = require('express')
const multer = require('multer')
const app = express()
const storage = multer.diskStorage({
    destination: function(req, file, cb){
        cb(null, 'upload/')
    },
    filename: function(req, file, cb){
        cb(null, file.originalname)
    },
})
const upload = multer({storage})

app.listen(3000, ()=>{
    console.log('server online')
})

app.get('/', (req, res)=>{
    res.render('form.ejs')
})

app.post('/submit', upload.single('photo'), (req, res)=>{
    res.json(req.file)
})

## Cheat

TO-DO APP STORAGE
let container = document.getElementById('text');
let input = document.getElementById('input');
let addButton = document.getElementById('add');

function to add key and values in localstorage

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

    Set the checkbox's checked property based on the value from localStorage
    let isChecked = localStorage.getItem(key) === "true";
    newCheck.checked = isChecked;
    left.innerHTML = key;
    newContainer.appendChild(left);

    update the task is done or not
    right.appendChild(newPencil);
    right.appendChild(newCheck);
    right.appendChild(newButton);


    newContainer.appendChild(right);
    container.appendChild(newContainer);

    newButton.addEventListener('click', () => {
        localStorage.removeItem(key);        Use the key, not innerText
        container.removeChild(newContainer);
    })
}

function to check whether localStorage is empty or not
if (localStorage.length != 0) {
    for (let i = 0; i < localStorage.length; i++) {
        addLocalValues(Object.keys(localStorage)[i], Object.values(localStorage)[i]);   
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

        adding to local Storage
        localStorage.setItem(input.value, false);
        input.value = "";

        newButton.addEventListener('click', () => {
            localStorage.removeItem(newButton.parentElement.parentElement.innerText);
            container.removeChild(newContainer);
        })
    }
})

function to update localStorage on the based on task complete
container.addEventListener('change', (e) => {
    if (e.target.type === 'checkbox') {
        const isChecked = e.target.checked;
        const textContent = e.target.parentElement.parentElement.querySelector('#left').textContent;
        localStorage.setItem(textContent, isChecked);
    }
});

function to edit div text by using pencil symbol
container.addEventListener('click', (e) => {
    if (e.target.id === 'newPencil') {
        const taskContainer = e.target.parentElement.parentElement.parentElement;
        const taskTextElement = taskContainer.querySelector('#left');
        Store the old value before editing
        const oldValue = taskTextElement.textContent;
        taskTextElement.contentEditable = true;
        taskTextElement.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                taskTextElement.contentEditable = false;
                const newValue = taskTextElement.textContent;
                Remove the old value from localStorage
                localStorage.removeItem(oldValue);
                Set the updated value in localStorage
                localStorage.setItem(newValue, false);
                Update the task text content
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

        adding to local Storage
        localStorage.setItem(input.value, false);
        input.value = "";

        newButton.addEventListener('click', () => {
            localStorage.removeItem(newButton.parentElement.parentElement.innerText);
            container.removeChild(newContainer);
        })
    }
    function to update localStorage on the based on task complete
    const list = document.querySelectorAll("input[type=checkbox]");

    for (const checkbox of list) {
        checkbox.addEventListener('click', () => {
            const isChecked = checkbox.checked;
            const textContent = checkbox.parentElement.parentElement.textContent;

            localStorage.setItem(textContent, isChecked);
        });

        Initialize the checkbox state based on localStorage
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

        setting up the questions and answers        
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


## Installation

1. Launch Visual Studio Code.
2. Go to the Extensions view by clicking on the Extensions icon in the Activity Bar on the side of the window or use the shortcut `Ctrl+Shift+X`.
3. Search for "CodeEasy Themes".
4. Click on the "Install" button for the CodeEasy Themes extension.

## Available Themes

- **Codeeasy:** A dark theme that reduces eye strain with carefully selected color combinations.
  
   More to come in the future! 

## Activation

1. After installing the extension, open the Command Palette (`Ctrl+Shift+P`).
2. Type `Preferences: Color Theme` and press Enter.
3. Choose your preferred CodeEasy theme from the list.

## Features

- **Readability:** CodeEasy Themes are designed with a focus on readability to make your code more accessible.
- **Syntax Highlighting:** Enjoy carefully chosen colors for syntax elements to improve code comprehension.
- **Consistency:** All themes maintain a consistent color palette for a cohesive coding experience.

## Customization

Feel free to customize the themes according to your preferences. You can easily modify colors using the built-in Visual Studio Code color customization features.

1. Open the Command Palette (`Ctrl+Shift+P`).
2. Type `Preferences: Open Settings (JSON)` and press Enter.
3. Add or modify color settings under the `"editor.tokenColorCustomizations"` section.

```json
"editor.tokenColorCustomizations": {
    "textMateRules": [
        {
            "scope": [
                "comment",
                "keyword"
            ],
            "settings": {
                "foreground": "#008000"
            }
        }
    ]
}
