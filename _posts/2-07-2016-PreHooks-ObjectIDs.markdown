---
layout: post
title:  "Handling Pre Hooks when Working With Mongoose"
date:   2016-02-07 19:59:21 -0800
categories: Node.js Express Mongoose MongoDB ObjectID BackEnd RESTful-API
---

Node and Express make routing and designing RESTful API's much easier. Express gives us access to the next() middle ware. This allows us to have certain routes that handle different models and pieces of backend logic. If we are working with the MEAN stack, we can use [Mongoose][Mongoose]Mongoose as an abstraction layer for our MongoDB database. 

Using Mongoose allows us to add pre and post hooks that will manipulate our request data before we pass it to the function (in our model controller) that we want. We can use this to  update other related models that exist in our database. This becomes particularly useful when we are using ObjectIDs to connect our data models (i.e. users have an array of comments, through object ids). We can listen for the "save" command on a comment and subsequently update our user model to hold that comment.

Let's show how hooks work by setting up a simple Node/Express Backend that handles routing for users and the comments they might have. This means that we will have two mongoose models: Users and Comments.

Our server.js looks like this:



{% highlight javascript %}

var express  = require('express'),
    mongoose = require('mongoose'),
    CONF = require('../_config.json'); 
    // Defines a JSON object holding our PORT and DB URI it looks like this:
   /*
    {
      "PORT": {
        "MAIN": 3030,
        "TEST": 5080
      },
      "DB_URI": {
        "MAIN": "mongodb://localhost/roadmapToAnything",
        "TEST": "mongodb://localhost/roadmapTestDB"
      }
    }                             *
    */
    

mongoose.Promise = require('bluebird'); 
// Useful library for promises, gives Mongoose access to the then() function
// ensuring functionality happens in our desired order.


var app      = express();
var port     = process.env.PORT || CONF.PORT.MAIN;
var database = process.env.MONGOLAB_URI || CONF.DB_URI.MAIN;

if (process.env.NODE_ENV === 'test') {
  port = CONF.PORT.TEST;
  database = CONF.DB_URI.TEST;
}

mongoose.connect(database);

require('./middleware.js')(app, express); 
// Is the middleware common to all routes (e.g. CORS and bodyParser). 

require('./router.js'    )(app, express);

app.listen(port, function(){ 
  console.log('APP IS RUNNING ON', port);
});
  

module.exports = {
  app: app
};

{% endhighlight %}

Our server has a bit of syntax that just presents best practices as far as using a deployment server and if that doesn't exist to use the local host instead. Furthermore by using the json we can keep our code DRY (Dont' repeat yourself). What is most important is what happens after we pass in the middleware: the router.js file.


Our router.js looks like this: 

{% highlight javascript %}

module.exports = function (app, express) {

  var apiRouter = express.Router();
  

  app.use('/api', apiRouter);

  require('./api/apiRouter.js')(apiRouter); 

};

{% endhighlight %}

Our router.js file defines apiRouter as an express router, giving our routes access to the next() middle ware which wil help us integrate mongoose pre hooks. It also gives us a pre url route that our front end has access to. Before we access any route on our backend we first use /api/(desiredRoute). We are telling our server to use our api router. Now that we have defined it we should give our api router functionality. We do this by requiring our apiRouter.js file and passing in this apiRouter that we just created. When we do this all the functions that we create in our apiRouter.js will exist on our server as routes we are listening for on the port we decided to listen on.


Our apiRouter.js looks like this:

{% highlight javascript %}

var userController    = require('./users/userController.js'),
    

module.exports = function (apiRouter) {

  /*
   *      All routes begin with /api. When we access on front end we do something like
          http.get(/api/users).
   */

   /* * * * * * * * * * * * * * * * * * * * * 
    *              User Routes              *
    * * * * * * * * * * * * * * * * * * * * */

  apiRouter.get( '/login',  userController.login);
  apiRouter.post('/signup', userController.createUser);

  apiRouter.get(   '/users',                 userController.getUsers);
  apiRouter.get(   '/users/:username',       userController.getUserByName);
  apiRouter.put(   '/users/:username',       userController.updateUserByName);
  apiRouter.delete('/users/:username',       userController.deleteUserByName);

  /* * * * * * * * * * * * * * * * * * * * * 
   *              Comment Routes           *
   * * * * * * * * * * * * * * * * * * * * */

  apiRouter.post(  '/comments',              commentController.createComment );
  apiRouter.delete('/comments/:commentId',   commentController.deleteComment );
};


{% endhighlight %}


This is the setup for our user route and comment routes. As you can see we have a userController handle our routing. Each route, accesses a different property of the userController, these properties will define the functionality of accessing those routes. However, in our Mongoose Model for our user we have the following properties:


{% highlight javascript %}

var mongoose = require('mongoose'),
    ObjectId = mongoose.Schema.ObjectId,
    deepPopulate = require('mongoose-deep-populate')(mongoose),

    hooks    = require('../modelTriggers.js');

var UserSchema = new mongoose.Schema({
  username          : { type: String, required: true, unique: true },
  password          : { type: String, required: true },
  firstName         : { type: String },
  lastName          : { type: String },
  created           : { type: Date },
  updated           : { type: Date },
  comments          : [ {type: ObjectId, ref: 'Comment'} ]
});

UserSchema.plugin(deepPopulate);

hooks.setUserHooks(UserSchema);

module.exports = mongoose.model('User', UserSchema);

{% endhighlight %}


In our userModel.js We add one line of javascript to define our pre hook. And after we set up the model we expose it to the controller. The model itself is never accessed through our routes. Instead we pass the model into our userController and have the userController handle the appropriate functionality. ObjectId tells us that the comments field in our model is another model that exists in our database. It also gives us reference to a specific comment through its id, to later query on, as opposed to the content that exists within that comment within our user Model.

Let's see how a UserHook would work to automatically update our user's created date on creation:

{% highlight javascript %}

module.exports.setUserHooks = function(UserSchema) {
  UserSchema.pre('save', function(next) {
    setCreatedTimestamp.call(this, next);
  });

{% endhighlight %}

In our modelTriggers.js we define the pre hooks. These functions are attached as pre hooks, and work as listeners. Whenever we call a function on our modelController and it runs a particular function if we have a pre hook attached, before it runs the pre hook will do the requisite work. In this example we set a pre hook for the save command. Now everytime the save command is sent through our routes, we will set the created timestamp onto the model automatically. 

Things become even more interesting when we decide that users have other models attached to them as objectIDs. In this example let's try comments: Here we are running a author update command to push into our array of comments the new comment that our user just authored.

{% highlight javascript %}


module.exports.setCommentHooks = function(CommentSchema) {
  CommentSchema.pre('save', function(next) {
    if (this.isNew) {
      var User = require('./users/userModel.js');
      var authorId = this.author; // Same as user
      var commentId = this._id;

      var authorUpdate = {$push: {comments: commentId}};

      User.findByIdAndUpdate(authorId, authorUpdate)
        .exec(function(err){ if (err) throw err; });  
    }

    setCreatedTimestamp.call(this, next);
  });

  CommentSchema.pre('remove', function(next) {
    var User = require('./users/userModel.js');
    var authorId = this.author;

    var authorUpdate = {$pull: {comments: this._id}};

    User.findByIdAndUpdate(authorId, authorUpdate)
    .exec(function(err){ if (err) throw err; });  

    next();
  });

};

{% endhighlight %}

Here we add a pre hook to our comment schema to push our comment into the comments array that exists in our user model. The findByIdAndUpdate is a mongoose function that exists on our User model allowing us to take the user id and run our authorUpdate command on it. Semantically the function is doing the following: prior to saving our comment into our comments data store, we update our user to hold that comment in their comment array.

Using this design pattern lets us seperate concerns when designing our RESTful API's, and makes for more readable code. The seperation of files might seem a bit excessive at first, but when we have multiple data models who each might have to reference other controllers on their pre hooks having all of our hooks in one file and accessing them through functions lets us not repeat ourselves.

Mongoose is incredibly powerful and the next() middleware Express gives to us is incredibly powerful when moving from asynchronous function to other functions. As our data beomces more complex the power of having a javascript object as data allows us to make the necessary modifications easily before data into our database. The ability to set pre hooks leverages the more flexible power of MongoDB and the lightweight nature of express.


[Mongoose]: http://mongoosejs.com/
