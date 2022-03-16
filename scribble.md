# 14.1.3

We are using the same module from last week and building on it. The first thing we did was add a public folder with a nested stylesheet folder. Inside of that we added a copy and pasted style sheet then

## went into the server.js and added this

const path = require("path");

and

app.use(express.static(path.join(\_\_dirname, "public")));
--.

These two things together tell the server.js file to use everything in the public folder on load.

## Then we tested it out with

http://localhost:3001/stylesheets/style.css
--.

# 14.1.4

Added the HTML.

## First we installed handlebars with this

npm install express-handlebars
--.

Then we required it in the server.js and put a few more lines in to get it set up.

## server.js handlebars set up code

const exphbs = require('express-handlebars');
const hbs = exphbs.create({});

app.engine('handlebars', hbs.engine);
app.set('view engine', 'handlebars');
--.

Next we created a view folder and a layouts folder a put a main.handlebars inside it and gave it the HTML starter code. Finally we went up one folder back into the views and added a homepage.handlebars file that just contained a single div letting you know you are on the homepage. The file structure here is strict because everything here will go into layouts and apply itself to the {{body}} in the main.handlebars html.

Then we changed the name of our routes folder to controllers and updated the server.js to reflect this name change. Finally we created a

## home-routes.js in the controllers folder and put in this

const router = require('express').Router();

router.get('/', (req, res) => {
res.render('homepage');
});

module.exports = router;
--.

Which tells it to render the homepage.handlebars on load.

Also we had to update the controllers/index.js to include this file.

## With this

const homeRoutes = require('./home-routes.js');

router.use('/', homeRoutes);
--.

# 14.1.5

we went into home-routes.js

## and added this

router.get('/', (req, res) => {
res.render('homepage', {
id: 1,
post_url: 'https://handlebarsjs.com/guide/',
title: 'Handlebars Docs',
created_at: new Date(),
vote_count: 10,
comments: [{}, {}],
user: {
username: 'test_user'
}
});
});
--.

This is essentially a fake get that will send this data into the html to be useable.

## in homepage.handlebars we added this

<ol class="post-list">
  <li>
    <article class="post">
      <div class="title">
        <a href="{{post_url}}" target="_blank">{{title}}</a>
        <span>({{post_url}})</span>
      </div>
      <div class="meta">
        {{vote_count}} point(s) by {{user.username}} on
        {{created_at}}
        |
        <a href="/post/{{id}}">{{comments.length}} comment(s)</a>
      </div>
    </article>
  </li>
</ol>
--.

Which takes the above data and plugs it into the html using {{}} to hold the object names. Simply launching localhot:3001 will look at all of this in it's HTML format.

# 14.1.6

We went into home-routes.js and set it up to pull from the database instead of being hardcoded.

## It looked like this at first

router.get('/', (req, res) => {
Post.findAll({
attributes: [
'id',
'post_url',
'title',
'created_at',
[sequelize.literal('(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)'), 'vote_count']
],
include: [
{
model: Comment,
attributes: ['id', 'comment_text', 'post_id', 'user_id', 'created_at'],
include: {
model: User,
attributes: ['username']
}
},
{
model: User,
attributes: ['username']
}
]
})
.then(dbPostData => {
// pass a single post object into the homepage template
res.render('homepage', dbPostData[0]);
})
.catch(err => {
console.log(err);
res.status(500).json(err);
});
});
--.

We hardcoded the 0 to show the first spot. The only real issue here is the data came to us in a huge sequelize object that didn't really work with what we had.

## To make it work we added this

res.render('homepage', dbPostData[0].get({ plain: true }));
--.

This does something called serialize which just parses over the data and gives us only the info we want.

## Finally in this same file of home-routes.js we changed it to this

const posts = dbPostData.map(post => post.get({ plain: true }));

res.render('homepage', { posts });
--.

So we will map over the entire array. Unfortunately this busted homepage.handlebars and we had to add {{#each posts}} and {{/each}}
to make the array work.

## This looked something like this

<ol class="post-list">
  {{#each posts}}
  <li>
    <article class="post">
      <div class="title">
        <a href="{{post_url}}" target="_blank">{{title}}</a>
        <span>({{post_url}})</span>
      </div>
      <div class="meta">
        {{vote_count}} point(s) by {{user.username}} on
        {{created_at}}
        |
        <a href="/post/{{id}}">{{comments.length}} comment(s)</a>
      </div>
    </article>
  </li>
  {{/each}}
</ol>
--.

Finally for what appears to be for nothing more than coding legibility we changed the each up top to {{#each posts as |post|}} so the code will show the word post everywhere so people will be less confused.

## The end result looked like this

<ol class="post-list">
  {{#each posts as |post|}}
  <li>
    <article class="post">
      <div class="title">
        <a href="{{post.post_url}}" target="_blank">{{post.title}}</a>
        <span>({{post.post_url}})</span>
      </div>
      <div class="meta">
        {{post.vote_count}} point(s) by {{post.user.username}} on
        {{post.created_at}}
        |
        <a href="/post/{{post.id}}">{{post.comments.length}} comment(s)</a>
      </div>
    </article>
  </li>
  {{/each}}
</ol>
--.

# 14.2.3

We set up the login page. First we went into the main.handelbars and

added a login button to the header like this
--.

<header>
        <h1>
          <a href="/">Just Tech News</a>
        </h1>
        <nav>
          <a href="/login">login</a>
        </nav>
      </header>
--.

Then we went into the views folder and created a new file called login.handlebars and added the html code that will display the

## login forms like this

<form class="login-form">
  <div>
    <label for="email-login">email:</label>
    <input type="text" id="email-login" />
  </div>
  <div>
    <label for="password-login">password:</label>
    <input type="password" id="password-login" />
  </div>
  <div>
    <button type="submit">login</button>
  </div>
</form>

<form class="signup-form">
  <div>
    <label for="username-signup">username:</label>
    <input type="text" id="username-signup" />
  </div>
  <div>
    <label for="email-signup">email:</label>
    <input type="text" id="email-signup" />
  </div>
  <div>
    <label for="password-signup">password:</label>
    <input type="password" id="password-signup" />
  </div>
  <div>
    <button type="submit">signup</button>
  </div>
</form>
--.

Finally we linked it so when clicked we will be taken to this page by going into the cotrollers folder in the home-routes.js file and

## added this code

router.get("/login", (req, res) => {
res.render("login");
});
--.

which will link us out to the login.handlebars page. The code is much cleaner than our loading homepage because we are not pulling anything from the database to display this page.

# 14.2.4

We set up the front end javascript to handle the login and sign up forms. Once it is all said in done when you sign up you will get added to the database and once you log in you will get redirected to the main page. The very first thing we did was we went into the public folder and made a nested javacript folder. Then we added a login.js file. Afterwards we slid into the login.handlebars html and

## included the path to the javascript file the html will be using at the very bottom of the file

<script src="/javascript/login.js"></script>

--.

Then we went back into the newly created login.js file and added

## this code to listen to the sign up form

function signupFormHandler(event) {
event.preventDefault();

const username = document.querySelector('#username-signup').value.trim();
const email = document.querySelector('#email-signup').value.trim();
const password = document.querySelector('#password-signup').value.trim();

if (username && email && password) {
fetch('/api/users', {
method: 'post',
body: JSON.stringify({
username,
email,
password
}),
headers: { 'Content-Type': 'application/json' }
}).then((response) => {console.log(response)})
}
}

querySelector('.signup-form').addEventListener('submit', signupFormHandler);
--.

This worked well and added the user to the database without issue which was easy to verify in insomnia. But the they got all ES6 happy and wanted

## us to code it like this

async function signupFormHandler(event) {
event.preventDefault();

const username = document.querySelector("#username-signup").value.trim();
const email = document.querySelector("#email-signup").value.trim();
const password = document.querySelector("#password-signup").value.trim();

if (username && email && password) {
const response = await fetch("/api/users", {
method: "post",
body: JSON.stringify({
username,
email,
password,
}),
headers: { "Content-Type": "application/json" },
});

    // check the response status
    if (response.ok) {
      console.log("success");
    } else {
      alert(response.statusText);
    }

}
}

document
.querySelector(".login-form")
.addEventListener("submit", loginFormHandler);
--.

To avoid having to use .then and .catch and what not. Finally we added all the info for the

## log in form that looked like this

async function loginFormHandler(event) {
event.preventDefault();

const email = document.querySelector('#email-login').value.trim();
const password = document.querySelector('#password-login').value.trim();

if (email && password) {
const response = await fetch('/api/users/login', {
method: 'post',
body: JSON.stringify({
email,
password
}),
headers: { 'Content-Type': 'application/json' }
});

    if (response.ok) {
      document.location.replace('/');
    } else {
      alert(response.statusText);
    }

}
}

document.querySelector('.login-form').addEventListener('submit', loginFormHandler);
--.

The main difference here is that after you login it kicks you back out to the homepage, which of course the module didn't realize so that's fun. To make the work match the module I had to bring down the if(response.ok) from the submit form to test it out there way.

# 14.2.5

We installed express session and connect session sequalize for the first time. Then we went into the root server.js

## and added this code to allow the new packages

const session = require('express-session');

const SequelizeStore = require('connect-session-sequelize')(session.Store);

const sess = {
secret: 'Super secret secret',
cookie: {},
resave: false,
saveUninitialized: true,
store: new SequelizeStore({
db: sequelize
})
};

app.use(session(sess));
--.

Then we went into controllers/api/user-routes.js changed the .then

## in the router.post("/") to look like this

router.post("/", (req, res) => {
// expects {username: 'Lernantino', email: 'lernantino@gmail.com', password: 'password1234'}
User.create({
username: req.body.username,
email: req.body.email,
password: req.body.password,
})
.then((dbUserData) => {
req.session.save(() => {
req.session.user_id = dbUserData.id;
req.session.username = dbUserData.username;
req.session.loggedIn = true;

        res.json(dbUserData);
      });
    })
    .catch((err) => {
      console.log(err);
      res.status(500).json(err);
    });

});
--.

The req.session stuff is what actually logs you in be creating a personal session tied to your user id. Then we did the same thing for user login in the user-routes .js file.

## The code looked like this

router.post("/login", (req, res) => {
User.findOne({
where: {
email: req.body.email,
},
}).then((dbUserData) => {
if (!dbUserData) {
res.status(400).json({ message: "No user with that email address!" });
return;
}

    const validPassword = dbUserData.checkPassword(req.body.password);

    if (!validPassword) {
      res.status(400).json({ message: "Incorrect password!" });
      return;
    }

    req.session.save(() => {
      // declare session variables
      req.session.user_id = dbUserData.id;
      req.session.username = dbUserData.username;
      req.session.loggedIn = true;

      res.json({ user: dbUserData, message: "You are now logged in!" });
    });

});
});
--.

Again making it so when you login you create a session to give the user experience of being logged into a site. You can aslo go into the dev tools to get the session name by clicking application and navigating to the cookies folder. Finally we added a bit of code to the the controller/hone-routes.js that will atomiatically return you to the home page everytime you click on log in if you are already logged in.

## home page log in reroute

router.get("/login", (req, res) => {
if (req.session.loggedIn) {
res.redirect("/");
return;
}

res.render("login");
});
--.

# 14.2.6

created a log out file and button to terminate the session. First we went into controller/api/user-route.js and added a new route to

## log us out from the session.

router.post("/logout", (req, res) => {
if (req.session.loggedIn) {
req.session.destroy(() => {
res.status(204).end();
});
} else {
res.status(404).end();
}
});
--.

Then we added a logout button to the html in the main.handlebars just above the log in button

## like this

<nav>
          <button id="logout" class="btn-no-style">logout</button>
          <a href="/login">login</a>
        </nav>
--.

Finally we added t link just above the closing body in out main.handlebars to link to.....

Then we made a file in public/javascript/logout.js

## where we added

async function logout() {
const response = await fetch("/api/users/logout", {
method: "post",
headers: { "Content-Type": "application/json" },
});

if (response.ok) {
document.location.replace("/");
} else {
alert(response.statusText);
}
}

document.querySelector("#logout").addEventListener("click", logout);
--.
This hot hot ES6 syntax to log us out by fetching the route we just made.

# 14.3.3

We added a new page called single-post.handlebars to show the page when a user clicks on one post.

## Then in this file we added this

<article class="post">
  <div class="title">
    <a href="{{post.post_url}}" target="_blank">{{post.title}}</a>
    <span>({{post.post_url}})</span>
  </div>
  <div class="meta">
    {{post.vote_count}}
    point(s) by
    {{post.user.username}}
    on
    {{post.created_at}}
    |
    <a href="/post/{{post.id}}">{{post.comments.length}} comment(s)</a>
  </div>
</article>

<form class="comment-form">
  <div>
    <textarea name="comment-body"></textarea>
  </div>

  <div>
    <button type="submit">add comment</button>
    <button type="button" class="upvote-btn">upvote</button>
  </div>
</form>

<div class="comments">
  {{#each post.comments}}
    <section class="comment">
      <div class="meta">
        {{user.username}}
        on
        {{created_at}}
      </div>
      <div class="text">
        {{comment_text}}
      </div>
    </section>
  {{/each}}
</div>
--.

The top bit makes a link that is named after the title and displays the URL text next to it. Then we go down and display more stuff that matched in ways that makes sense. Once we get to the comment section we make a text area and attach 2 buttons to it that do not have functionality just yet.

Finally we revisit the #each to pull out all of the comments and display them as shown.

In home-routes.js we had to set up the backend to actually get all the data for what was just mentioned.

## It looks like this

router.get("/post/:id", (req, res) => {
Post.findOne({
where: {
id: req.params.id,
},
attributes: [
"id",
"post_url",
"title",
"created_at",
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
include: [
{
model: Comment,
attributes: ["id", "comment_text", "post_id", "user_id", "created_at"],
include: {
model: User,
attributes: ["username"],
},
},
{
model: User,
attributes: ["username"],
},
],
})
.then((dbPostData) => {
if (!dbPostData) {
res.status(404).json({ message: "No post found with this id" });
return;
}
--.

This is fairly straightforward referencing multiple tables from the post table to get all the needed data. The only part where they juked me was when they revealed that at some point in the hompage.handlebars we added a href link to kick us out to this new page.

## That link looked like this

<a href="/post/{{post.id}}">{{post.comments.length}} comment(s)</a>
--.

Basically saying when you click on the word comments it will shot us here by grabbing it's id.

# 14.3.4

We made the files for the front end js called comment.js and upvote.js. Then we went into the single-post.handlebars and added

## the javascript links like this

<script src="/javascript/comment.js"></script>
<script src="/javascript/upvote.js"></script>

--.

Then we went into the newly created upvote.js and added code to make sure that it uses our created PUT route called api/posts/upvote in the post-routes.js folder.

## That looks like this

async function upvoteClickHandler(event) {
event.preventDefault();

const id = window.location.toString().split("/")[
window.location.toString().split("/").length - 1
];

console.log(id);
console.log("button clicked");

const response = await fetch("/api/posts/upvote", {
method: "PUT",
body: JSON.stringify({
post_id: id,
}),
headers: {
"Content-Type": "application/json",
},
});

if (response.ok) {
document.location.reload();
} else {
alert(response.statusText);
}
}

document
.querySelector(".upvote-btn")
.addEventListener("click", upvoteClickHandler);
--.

Finally we went into the post-routes.js to make sure it knows to check if you are logged in before you have permission to upvote

## with if (req.session)

router.put("/upvote", (req, res) => {
// make sure the session exists first
if (req.session) {
// pass session id along with all destructured properties on req.body
Post.upvote(
{ ...req.body, user_id: req.session.user_id },
{ Vote, Comment, User }
)
.then((updatedVoteData) => res.json(updatedVoteData))
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
}
});
--.
