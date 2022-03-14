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
