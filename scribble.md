mysql -u root -p
git remote -v

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

# 14.3.5

So this time we added the ability to add comments. We added

## this to the public/javascript/comment.js file

async function commentFormHandler(event) {
event.preventDefault();

const comment_text = document
.querySelector('textarea[name="comment-body"]')
.value.trim();

const post_id = window.location.toString().split("/")[
window.location.toString().split("/").length - 1
];

if (comment_text) {
const response = await fetch("/api/comments", {
method: "POST",
body: JSON.stringify({
post_id,
comment_text,
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
}

document
.querySelector(".comment-form")
.addEventListener("submit", commentFormHandler);
--.

Which will go in and a comment using the post route in the comment-routes.js. Then we went into that file and updated the route to include the if(req.session)

## that looks like this

router.post("/", (req, res) => {
// check the session
if (req.session) {
Comment.create({
comment_text: req.body.comment_text,
post_id: req.body.post_id,
// use the id from the session
user_id: req.session.user_id,
})
.then((dbCommentData) => res.json(dbCommentData))
.catch((err) => {
console.log(err);
res.status(400).json(err);
});
}
});
--.

The only issue is that this does not function as intended. The module assumes with this if statement it will throw up an error if you are not logged in. But in fact it accepts null. In order to fix this I went into the model folder and found comment.js's user_id and added allow_null: false. Once I tested that is threw up an error that I was expecting. Then when you login it adds. I have since cleared it hoping the module will show us a way around this.

# 14.3.6

Added {{#if loggedIn}} {{/if}} statements to only display certain HTML if the user in logged in.

## Here is an example of us hiding the comment box, submit button and the upvote button

{{#if loggedIn}}

  <form class="comment-form">
    <div>
      <textarea name="comment-body"></textarea>
    </div>

    <div>
      <button type="submit">add comment</button>
      <button type="button" class="upvote-btn">upvote</button>
    </div>

  </form>
{{/if}}
--.

This was done in the single-post.handlebars. We also updated the

## single-post at the /post/:id by adding loggedIn: req.session.loggedIn

.then((dbPostData) => {
if (!dbPostData) {
res.status(404).json({ message: "No post found with this id" });
return;
}

      // serialize the data
      const post = dbPostData.get({ plain: true });

      // pass data to template
      res.render("single-post", {
        post,
        loggedIn: req.session.loggedIn,
      });
    })
    .catch((err) => {
      console.log(err);
      res.status(500).json(err);
    });

});
--.

Finally we were getting a little error where it single-post.handlebars was trying to load the front end JS even if the user wasn't logged in. To fix this we added another if statement where we

## load the front in JS

{{#if loggedIn}}

  <script src="/javascript/comment.js"></script>
  <script src="/javascript/upvote.js"></script>

{{/if}}
--.

Which cleaned up our console errors.

# 14.3.7

We finally fixed the logout button. They never realized that they had us delete it in the zip squish. But either way we went into the main.handlebars and created our first ever if else to show the logout button if logged in otherwise show the log out button

## This looked like this

<header>
        <h1>
          <a href="/">Just Tech News</a>
        </h1>
        <nav>
          {{#if loggedIn}}
            <button id="logout" class="btn-no-style">logout</button>
          {{else}}
            <a href="/login">login</a>
          {{/if}}
        </nav>
      </header>
--.

Then we finally brought the reference the the logout button back and where told to

## give it an #if

{{#if loggedIn}}

<script src="/javascript/logout.js"></script>

{{/if}}
--.

Finally went into the get (/) for the home-routes.js and amde sure that this also checked for login

## with this code

.then((dbPostData) => {
const posts = dbPostData.map((post) => post.get({ plain: true }));

      res.render("homepage", {
        posts,
        loggedIn: req.session.loggedIn,
      });
    })
    .catch((err) => {
      console.log(err);
      res.status(500).json(err);
    });

});
--.

The page now seems to be working as intended with everything in properly.

# 14.4.3

Created the first partial. We made a folder in the root of views along side the layouts folder. Inside it we created the pot-info.handlebars file that then we cut out the article section from the hompage and single-post handlebars since they are identical code. We also removed post. in front of every variable because this file does now know what the post object is yet.

## This is what we moved into post-info.handlerbars

<article class="post">
  <div class="title">
    <a href="{{post_url}}" target="_blank">{{title}}</a>
    <span>({{post_url}})</span>
  </div>
  <div class="meta">
    {{vote_count}}
    point(s) by
    {{user.username}}
    on
    {{created_at}}
    |
    <a href="/post/{{id}}">{{comments.length}} comment(s)</a>
  </div>
</article>
--.

Then in the hompage and single-post.handlerbars where we removed the articles

## we added this

{{> post-info post }}
--.
As you can see here the first bit references the page and the second renames it post. Also to console.log in handlebars simply type {{log this}} with this being whatever it is you are trying to log.

# 14.4.4

We did the same thing but got juked this time because we are dealing with an array. First we went back into the newly created views/partials folder and created a new comments.handlebars.

## In this we cut this

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

From the single-post.handlebars and

## then added

{{> comments post.comments }}
--.

To the spot where we removed it. Yet again we needed to fix an issue up top in the comments.handlebars. This time instead of being able to simply remove post. from everything, if you notice there isn't a .post anywhere here which should of been the first clue, we had to change the line up top that conatined the #each.

## The end result looked like this

<div class="comments">
  {{#each this}}
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

Basically since we are passing in an unnamed array we had to use the "this" keyword to reference the array each time we loop over it.

# 14.4.5

We set up a **test** folder and added a herlper.test.js file to it. Then we set up a utils folder and added the helper.js file to it.

## In the test we added this

const { format_date } = require("../utils/helpers");

test("format_date() returns a date string", () => {
const date = new Date("2020-03-20 16:12:03");

expect(format_date(date)).toBe("3/20/2020");
});
--.

Which failed because we didn't had the code yet to return the date back in the proper format. To do that we slid back into the newly created

## helper.js file and added this

module.exports = {
format_date: (date) => {
return `${new Date(date).getMonth() + 1}/${new Date( date ).getDate()}/${new Date(date).getFullYear()}`;
},
};
--.

This is some code that goes through and uses built in javascript Date object and formats it to give the test the date in the order it wants. This passes and we are good to go.

# 14.4.6

We set up a test to check for plurals in the

## file helpers.test.js it looks like this

const { format_date, format_plural } = require("../utils/helpers");

test("format_plural() returns a pluralized word", () => {
const word1 = format_plural("tiger", 1);
const word2 = format_plural("lion", 2);

expect(word1).toBe("tiger");
expect(word2).toBe("lions");
});

existing code
--.

Then we failed it and added

## this to herlpers.js

format_plural: (word, amount) => {
if (amount !== 1) {
return `${word}s`;
}

    return word;

},
--.

We ran that and passed it and are moving right along.

# 14.4.7

Added a test that trims down the URL. First we added the test to

## helpers.test.js that looks like this

test("format_url() returns a simplified url string", () => {
const url1 = format_url("http://test.com/page/1");
const url2 = format_url("https://www.coolstuff.com/abcdefg/");
const url3 = format_url("https://www.google.com?q=hello");

expect(url1).toBe("test.com");
expect(url2).toBe("coolstuff.com");
expect(url3).toBe("google.com");
});
--.

Then we went back into the helper.js and chained

## several functions to remove text from the URL

format_url: url => {
return url
.replace('http://', '')
.replace('https://', '')
.replace('www.', '')
.split('/')[0]
.split('?')[0];
},
--.
From top to bottom we replace http with an empty string. We replace https with an empty string. We replace www. with an empty string. We cut everything on the / and then only grab the very first thing in the array. Then we cut everything on the ? and only grab the first thing out of the array.

# 14.4.8

We added all the pluralization helpers to the code for points and comments.

## First we went into server.js and added

const helpers = require('./utils/helpers');
&
const hbs = exphbs.create({ helpers });
--.
The helpers was simply placed within the existing curly braces. Then we went into comments.handlebars partial and simply added format_date to

## call the function like this

<section class="comment">
      <div class="meta">
        {{user.username}}
        on
        {{format_date created_at}}
      </div>
      <div class="text">
        {{comment_text}}
      </div>
    </section>
  {{/each}}
--.

We also went to any spot we could find created at and put a forma_date in front of it like in post-info. Then we jumped in front of any
post_url and added format_url like this
--

<div class="title">
    <a href="{{post_url}}" target="_blank">{{title}}</a>
    <span>({{format_url post_url}})</span>
  </div>
--.

Finally we changed every point(s) and comment(s) to have it's own object that uses format_plural then passes in the string we see for exapmple "point" and then uses vote_vount or comments.length to get a number. If that number is greater than 1 it adds an "s" to the end of the passed in string.

## here are both examples

<div class="meta">
    {{vote_count}}
    {{format_plural "point" vote_count}}
    by
    {{user.username}}
    on
    {{format_date created_at}}
    |
    <a href="/post/{{id}}">{{comments.length}}
      {{format_plural "comment" comments.length}}</a>
  </div>
--.

# 14.5.3

We added the dashboard page to the site. First we went into views folder and

## added dashborad.handlebars with this code

<section>
  <h2>Create New Post</h2>

  <form class="new-post-form">
    <div>
      <label for="post-title">Title</label>
      <input type="text" id="post-title" name="post-title" />
    </div>
    <div>
      <label for="post-url">Link</label>
      <input id="post-url" name="post-url" />
    </div>
    <button type="submit" class="btn">Create</button>
  </form>
</section>
--.

Then we went into controllers and gave it it's routes folder named

## dashboard-routes.js and added this code

const router = require('express').Router();
const sequelize = require('../config/connection');
const { Post, User, Comment } = require('../models');

router.get('/', (req, res) => {
res.render('dashboard', { loggedIn: true });
});

module.exports = router;
--.

Then we slid into the controller index.js and required and used the new dashboard-routes document

## here is an updated look at the index.js

const router = require("express").Router();

const apiRoutes = require("./api/");
const homeRoutes = require("./home-routes.js");
const dashboardRoutes = require("./dashboard-routes.js");

router.use("/", homeRoutes);
router.use("/api", apiRoutes);
router.use("/dashboard", dashboardRoutes);

## module.exports = router;

Finally we made sure to add the link to the main.handlebars and put it in the spot where we hide it if you are not logged in and show it if you are

## like this

<nav>
  {{#if loggedIn}}
  <a href="/dashboard">dashboard</a>
  <button id="logout" class="btn-no-style">logout</button>
  {{else}}
  <a href="/login">login</a>
  {{/if}}
</nav>
--.

# 14.5.4

We added the ability to see our posts and the front end JS with the route to get the dashboard create button working. First we went back into the newly created dashboard-routes,js

## and added a get.route

router.get('/', (req, res) => {
Post.findAll({
where: {
// use the ID from the session
user_id: req.session.user_id
},
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
// serialize data before passing to template
const posts = dbPostData.map(post => post.get({ plain: true }));
res.render('dashboard', { posts, loggedIn: true });
})
.catch(err => {
console.log(err);
res.status(500).json(err);
});
});
--.

This basically is a a copy paste of the find all posts from post-routes with the addition of the where statement making sure we grab everything by session id. Then we made a section in

## dashboard.handlebars that looks like this

{{#if posts.length}}

<section>
  <h2>Your Posts</h2>
  <ol>
    {{#each posts as |post|}}
    <li>
      {{> post-info post}}
      <a href="/dashboard/edit/{{post.id}}" class="edit-link">Edit post</a>
    </li>
    {{/each}}
  </ol>
</section>
{{/if}}

<script src="/javascript/add-post.js"></script>

--.

The if statement makes sure that this only displays if the user has any posts. Then it take the data from the get and sends it into the post-info partial to populate the screen. We also added the javascript link to the front end js logic for this page that we are naming

## add-post.js in the public/javascript file with code like this

async function newFormHandler(event) {
event.preventDefault();

const title = document.querySelector('input[name="post-title"]').value;
const post_url = document.querySelector('input[name="post-url"]').value;

const response = await fetch(`/api/posts`, {
method: "POST",
body: JSON.stringify({
title,
post_url,
}),
headers: {
"Content-Type": "application/json",
},
});

if (response.ok) {
document.location.replace("/dashboard");
} else {
alert(response.statusText);
}
}

document
.querySelector(".new-post-form")
.addEventListener("submit", newFormHandler);
--.
This basically listens for the click of the create button and add the title and the url to the post in api/post. Then in api posts we had to make sure it also passed in an id

## by making sure user_id required the session id like this

Post.create({
title: req.body.title,
post_url: req.body.post_url,
user_id: req.session.user_id
})
--.

# 14.5.5

We added our first middleware that checks if you are logged in. If you are caught doing something you shouldn't be you will be redirected to the login page. First we created a file in the utils folder named

## auth.js and added in this code

const withAuth = (req, res, next) => {
if (!req.session.user_id) {
res.redirect('/login');
} else {
next();
}
};

module.exports = withAuth;
--.

Here we are doing what I described above and redirecting you to the login screen if you are not logged in or sending saying everything is ok and using the next() to continue onto the next action. But first we have to define where we put it. Sop essentially what we did was consider any moment where we would want to check if you need to be logged in to have acces to this feature and squeezed in a withAuth which will call this function. But first we must require up top in any of the routes files that use withAuth so it will know what it is.

## Her is an example post-routes.js using this

const router = require("express").Router();
const sequelize = require("../../config/connection");
const { Post, User, Comment, Vote } = require("../../models");
const withAuth = require("../../utils/auth");

&

router.post("/", withAuth, (req, res) => {
// expects {title: 'Taskmaster goes public!', post_url: 'https://taskmaster.com/press', user_id: 1}
if (req.session) {
Post.create({
title: req.body.title,
post_url: req.body.post_url,
user_id: req.session.user_id,
})
.then((dbPostData) => res.json(dbPostData))
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
}
});
--.

There are several other examples throughout the code in comment.routes.js and dahboard-routes.js as well. I tried adding it to user-routes.js but things got a little weird when adding it to the post which makes sense. We are asking it to see if you are logged in while you are logging in which breaks it. You can however add it to log out and it works. I ultimately decided not to and cleared the require up top.

# 14.5.6

Added the functionality to the app to edit posts. First we went into the view folder and made the

## edit-post.handlebars and put in the HTML that looked like this

<article>
  <a href="/dashboard"> &larr; Back to dashboard</a>
  <h2>
    Edit Post
  </h2>
  <form class="edit-post-form">
    <div>
      <input name="post-title" type="text" value="{{post.title}}" />
      <span>({{format_url post.post_url}})</span>
    </div>
    <div>
      {{post.vote_count}} {{format_plural "point" post.vote_count}} by you on {{format_date post.created_at}}
      |
      <a href="/post/{{post.id}}">{{post.comments.length}} {{format_plural "comment" post.comments.length}}</a>
    </div>
    <button type="submit">Save post</button>
    <button type="button" class="delete-post-btn">Delete post</button>
  </form>
</article>

<form class="comment-form">
  <div>
    <textarea name="comment-body"></textarea>
  </div>

  <div>
    <button type="submit">add comment</button>
  </div>
</form>

{{> comments post.comments}}

<script src="/javascript/edit-post.js"></script>
<script src="/javascript/delete-post.js"></script>
<script src="/javascript/comment.js"></script>

--.

Here we set up the top to display several bits of info with an input section to type the new info in and 2 button options one to save changes or one to delete the enitre post. Then below that we added a spot to add a comment if you feel like and a button to save it. Below that we call the partial that will display all comments that are tied to this post. Finally we included the front end javascript links to make these html file's buttons and what not function.

## Now on to the delete-post.js file

async function deleteFormHandler(event) {
event.preventDefault();

const id = window.location.toString().split("/")[
window.location.toString().split("/").length - 1
];
console.log(id);
const response = await fetch(`/api/posts/${id}`, {
method: "DELETE",
});
if (response.ok) {
document.location.replace("/dashboard/");
} else {
alert(response.statusText);
}
}

document
.querySelector(".delete-post-btn")
.addEventListener("click", deleteFormHandler);
--.

This first grabs the id from the URL up top. In the URL the id is the number at the very end of the string and we set it to a var of id. Then we create the fetch where we use `` back ticks instead of "" quotes because we are grabbing the id in the route path with ${id}. Then we make sure the method us delete so it knows to look for a delete route and once done we use the replce.(/"dashboard") to kick you back out to the dashboard when done.

## Now the edit-post.js we had to do this

async function editFormHandler(event) {
event.preventDefault();

const title = document.querySelector('input[name="post-title"]').value.trim();
const id = window.location.toString().split("/")[
window.location.toString().split("/").length - 1
];
const response = await fetch(`/api/posts/${id}`, {
method: "PUT",
body: JSON.stringify({
title,
}),
headers: {
"Content-Type": "application/json",
},
});

if (response.ok) {
document.location.replace("/dashboard/");
} else {
alert(response.statusText);
}
}

document
.querySelector(".edit-post-form")
.addEventListener("submit", editFormHandler);
--.

The biggest changes here is the method we are using a PUT to look for a put route in the post-routes file and passing in a body of title which is what the route expects. But in order to do that we had to make sure that title matches the HTML using a query selector and get the new data we put into the input element and apply it to the variable title. Once that was set we had to add headers again for a reason I no longer remember though they seems to be tied to the body aspect of this put and then kicked you back out to the main dashboard page again with the replace("/dashborad/").
