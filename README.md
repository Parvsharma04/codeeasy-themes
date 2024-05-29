# CodeEasy Themes - Visual Studio Code Theme Extension

<img src="https://github.com/Parvsharma04/codeeasy/blob/master/logo.png?raw=true" width="100px"/>

## Overview

Welcome to CodeEasy Themes, a collection of visually pleasing and easy-to-read themes for Visual Studio Code. This extension provides a variety of carefully crafted color schemes to enhance your coding experience and make your code more readable.

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
```
