---
layout: post
title: "Easy Node Authentication With Ping"
subtitle: "Passport Module To Easily Handle Ping's OAuth and OpenID Connect Services"
description: "Introduction to ping-passport-oauth2, a Ping OAuth 2.0 authentication strategy for Passport."
header-img: "img/mon-woodlands.jpg"
authors:
    -
        name: Brenden McKamey
        githubProfile : "bsmckamey"
        avatarUrl : "https://avatars2.githubusercontent.com/u/14045210?v=3&s=460"
tags: [passport, auth, authn, authentication, authz, authorization, oauth, oauth2, ping]
---
# Easy Node Authentication With Ping

## Introduction

Adding authentication and login capability in Node can be painful. The purpose of this tutorial is to showcase the capabilities of [passport-ping-oauth2](https://github.com/MonsantoCo/passport-ping-oauth2) within a basic Node application, and it will teach you how to leverage the module within your own Node applications.

If you don't already have Node.js installed, you will need it, and you can install it [here](https://nodejs.org/en/).

### Note
If you don't have [PingFederate](https://www.pingidentity.com/en/products/pingfederate.html) in your environment, but you'd still like to learn about authentication in Node. I'd highly recommend using these [tutorials](https://scotch.io/courses/easy-node-authentication) instead.

### What we'll be building:
We will build an application that will allow you to leverage a PingFederate OAuth client to handle user authentication and the storage of the corresponding OpenID Connect data.

### End Product Sneak Peek:

A basic login screen that of course uses Ping.

![Login Screen](/img/passport-ping-oauth2-login.png)

After a user has logged in, they'll be able to see their OpenID Connect profile data:

![Profile Screen](/img/passport-ping-oauth2-profile.png)

That's it, pretty simple! So let's dive in.

## Application Structure

```sh
- app
------ models
------------ user.js
------ routes.js
- config
------ auth.js
------ database.js
------ passport.js
- views
------ pages
------------ index.ejs
------------ profile.ejs
------ partials
------------ header.ejs
app.json
index.js
package.json
```

Go ahead and create all the files and folders above and we’ll fill them in as we go along.

## Packages

###### package.json

This file handles all the packages needed for our application.

```json
{
    "name": "PingPassportTutorial",
    "description": "Easy Node Authentication With Ping.",
    "version": "0.1.5",
    "keywords": ["passport", "auth", "authn", "authentication", "authz", "authorization", "oauth", "oauth2", "ping"],
    "main": "index.js",
    "scripts": {
        "start": "node index.js"
    },
    "dependencies": {
        "ejs": "2.3.3",
        "express": "4.13.3",
        "mongoose" : "*",
        "passport": "*",
        "passport-ping-oauth2": "*",
        "connect-flash" : "*",     
        "bcrypt-nodejs" : "*",
        "morgan": "*",
        "body-parser": "*",
        "cookie-parser": "*",
        "method-override": "*",
        "express-session": "*"
    },
    "engines": {
        "node": "0.12.7"
    },
    "repository": {
        "type": "git",
        "url": "https://github.com/exampleuser/examplerepo"
    },
    "keywords": [
        "passport", "auth", "authn", "authentication", "authz", "authorization", "oauth", "oauth2", "ping"
    ],
    "license": "MIT"
}
```

Most of these are pretty self-explanatory; however, here is a list of the dependencies and the functionality they provide.

* EJS is a client-side templating language.
* Express is a web development framework for node.js.
* Mongoose is an object modeling tool designed to work in an asynchronous environment.
* Passport is authentication middleware for Node.js.
* Connect-flash is flash message middleware for Connect and Express.

## Application Setup

###### index.js

This file will be the glue for our entire application.


```js
var express = require('express');
var app = express();
var mongoose = require('mongoose');
var passport = require('passport');
var flash = require('connect-flash');
var morgan = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var session = require('express-session');
var configDB = require('./config/database.js');

mongoose.connect(configDB.url);

require('./config/passport')(passport);

app.use(morgan('dev'));
app.use(cookieParser());
app.use(bodyParser());

app.set('port', (process.env.PORT || 5000));

app.use(express.static(__dirname + '/public'));

// views is directory for all template files
app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');

app.use(session({ secret: 'tutorialsecret' }));
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());

require('./app/routes.js')(app, passport);

app.listen(app.get('port'), function() {
  console.log('Node app is running on port', app.get('port'));
});
```

## Database Config

###### config/database.js

```js
module.exports = {

    'url' : 'your-settings-here' // looks like mongodb://<user>:<pass>@mongo.onmodulus.net:27017/Mikha4ot

};
```

Fill this in with your own database. If you don’t have a MongoDB database lying around, you can install it locally. You can find instructions here: [An Introduction to MongoDB](https://scotch.io/tutorials/an-introduction-to-mongodb).

## Routes

###### app/routes.js

```js
module.exports = function(app, passport) {

    // route for home page
    app.get('/', function(req, res) {
        res.render('pages/index.ejs'); // load the index.ejs file
    });

    // route for showing the profile page
	app.get('/profile', isLoggedIn, function(req, res) {
        res.render('pages/profile.ejs', {
            user: req.user,
            flash: req.session.flash
        });
    });

    // route for logging out
    app.get('/logout', function(req, res) {
        req.logout();
        res.redirect('/');
    });

    // route for signing in
    app.get('/auth/oauth2', passport.authenticate('oauth2', {
        scope: ['openid', 'profile', 'email']
    }));

    // route for call back
    app.get('/auth/oauth2/callback',
        passport.authenticate('oauth2', {
            failureRedirect: '/login'
        }),
        function(req, res) {
            res.redirect('/profile');
        });
};

// route middleware to make sure a user is logged in
function isLoggedIn(req, res, next) {

    // if user is authenticated in the session, carry on
    if (req.isAuthenticated())
        return next();

    // if they aren't redirect them to the home page
    res.redirect('/');
}
```

## Views

###### views/partials/header.ejs

```html
<title>My Simple Passport Tutorial</title>
<link rel="stylesheet" type="text/css" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" />
<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<script type="text/javascript" src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"></script>
<link rel="stylesheet" type="text/css" href="/stylesheets/main.css" />
```

This header file is ported into each of the views, so we add all of our application wide CSS and JS here.

###### views/pages/index.ejs

```html
<!doctype html>
<html>

    <head>
        <% include ../partials/header.ejs %>
    </head>

    <body>
        
        <div class="container">
        
            <p>Login with:</p>
            <a href="/auth/oauth2" class="btn btn-primary"> Ping</a>
            
        </div>

    </body>

</html>
```

Just a simple button that points to our route for '/auth/oauth2' and triggers the passport authentication.

###### views/pages/profile.ejs

```html
<!doctype html>
<html>
    <head>
        <% include ../partials/header.ejs %>
    </head>
    <body>
        <div class="container">
            <% if (flash) { %>
                <div class="row">
                    <div class="<%= flash.type %>">   
                        <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>   
                        <p><strong><%= flash.title %></strong></p>
                        <p><%= flash.messages %></p>
                    </div>
                </div>
            <% } %>
            <div class="page-header text-center">
                <h1><span class="fa fa-anchor"></span> Profile Page</h1>
                <a href="/logout" class="btn btn-default btn-sm">Logout</a>
            </div>

            <div class="row">
                <div class="col-md-6 col-md-offset-3 well">
                    <h3 class="text-primary">Ping</h3>
                    <dl>
                        <dt>ID:</dt> <dd><%= user.ping.id %></dd>
                        <dt>Email:</dt> <dd><%= user.ping.email %></dd>
                        <dt>Name:</dt> <dd><%= user.ping.name %></dd>
                        <% if (user.ping.entitlements !== null && user.ping.entitlements.length > 0) { %>
                            <dt>Entitlements:</dt>
                            <dd> 
                                <ul>
                                    <% for (var i = 0; i < user.ping.entitlements.length; ++i) { %>
                                        <li><%= user.ping.entitlements[i] %></li>
                                    <% } %>
                                </ul>
                            </dd>
                        <% } %>
                    </dl>
                </div>
            </div>
        </div>
    </body>
</html>
```

A bit more complicated, but you can pass flash messages like this to any of your routes that are setup to handle them.

```js
req.session.flash = {
    type: 'alert alert-dismissible alert-danger',
    title: 'Access Denied',
    messages: 'Your access was denied because you do not have an entitlement for this application; however, we'll still show you your profile data.'
};
res.redirect(303, '/profile');
```

The profile view also renders our database data and loops through the array of entitlements.


## User Model

###### app/models/user.js

We will be storing **id**, **token**, **email**, **name**, and **entitlements** in our database. Normally there isn't a need to store the token; however, for demonstration purposes you'll be able to see how the passport library decrypts and passes the base 64 encoded data back.

```js
// app/models/user.js
// load the things we need
var mongoose = require('mongoose');

// define the schema for our user model
var userSchema = mongoose.Schema({

    ping         : {
        id           : String,
        token        : String,
        email        : String,
        name         : String,
        entitlements : [String]
    }

});

// create the model for users and expose it to our app
module.exports = mongoose.model('User', userSchema);
```

You'll notice that id, token, email, and name are just normal strings, but entitlements is an array of strings.

Okay, on to the fun stuff.

## Creating our Ping OAuth Client

First we're going to create a new OAuth client. After you've logged into your PingFederate administration console, click on **OAuth Settings** and then click on **Client Management**.

![Client Management](/img/passport-ping-oauth2-client-management.png)

Add a new client.

![Add Client](/img/passport-ping-oauth2-add-client.png)

You'll be using settings somewhat like this; however you'll need your own **Default Access Token Manager** and **OpenID Connect Policy** settings corresponding with your environment data. Remember to save the **Client Secret**!

![Client Settings](/img/passport-ping-oauth2-client-settings.png)

Go here if you need assistance with the [Default Access Token Manager](https://documentation.pingidentity.com/display/PF70/Configuring+Reference-Token+Management) or [OpenID Connect Policy](https://documentation.pingidentity.com/display/PF70/Configuring+OpenID+Connect+Policies#ConfiguringOpenIDConnectPolicies-2022433).

Okay, that wasn't too bad, so let's make it all work with two more files.

## Passport Strategy

###### config/auth.js

Populate your auth.js file appropriately from the Ping Configuration you just completed. It should look something like this, except with your environment specific information:

```js
module.exports = {

    'pingAuth' : {
        'authorizationURL'  : 'https://www.example.com/as/authorization.oauth2',
        'tokenURL'          : 'https://www.example.com/as/token.oauth2',
        'clientID'          : 'EXAMPLE_CLIENT_ID',
        'clientSecret'      : 'EXAMPLE_CLIENT_SECRET',
        'callbackURL'       : 'http://localhost:5000/auth/oauth2/callback',
    }

};
```

###### config/passport.js

This is the last piece of the puzzle, the file that handles all authentication and data storage.

```js
var passport = require('passport');
var OAuth2Strategy = require('passport-ping-oauth2').Strategy;
var User = require('../app/models/user');
var configAuth = require('./auth');

module.exports = function(passport) {


    // used to serialize the user for the session
    passport.serializeUser(function(user, done) {
        done(null, user.id);
    });


    // used to deserialize the user
    passport.deserializeUser(function(id, done) {
        User.findById(id, function(err, user) {
            done(err, user);
        });
    });


    passport.use(new OAuth2Strategy({
            authorizationURL: configAuth.pingAuth.authorizationURL,
            tokenURL: configAuth.pingAuth.tokenURL,
            clientID: configAuth.pingAuth.clientID,
            clientSecret: configAuth.pingAuth.clientSecret,
            callbackURL: configAuth.pingAuth.callbackURL
        },
        function(accessToken, refreshToken, params, profile, done) {
        
            User.findOne({ 'ping.id': profile.cn }, function(err, user) {
                if (err) { 
                    return done(err);
                }
                if (user) {
                    return done(null, user);
                } else {
                    var newUser = new User();
                    newUser.ping.id    = profile.id;
                    newUser.ping.token = accessToken;
                    newUser.ping.name  = profile.displayName;
                    newUser.ping.email = profile.email;
                    newUser.save(function(err) {
                        if (err) { throw err; }
                            return done(null, newUser);
                    });
                }
            });
        }));
};
```

## Run Your Application!

You should be able to run your application now whether it be locally or running on a server.

If you're running it locally, you'll need to run `npm install` from your root folder and then run `node index.js` to launch the application.

If you're running it on a server, deploy the code and run your application from the server.

## Conclusion

You've just built a simple application that handles Ping authentication for you and allows your users to seamlessly access your application without the need for them to sign up for another account. Awesome! You can build out your tokens to handle whatever profile data is available to Ping, and you can easily add this library to existing Node applications in order to abstract authentication work away from your application.

## Lastly

Hopefully you enjoyed this basic tutorial, and if you have any enhancements for [passport-ping-oauth2](https://github.com/MonsantoCo/passport-ping-oauth2) that you would like to contribute, submit a [pull request](https://help.github.com/articles/using-pull-requests/).
