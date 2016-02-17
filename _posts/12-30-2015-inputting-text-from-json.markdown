---
layout: post
title:  "How to Put Blocks of Text from a JSON file into Your App's HTML"
date:   2015-12-30 19:59:21 -0800
categories: Angular Cross-Origin JSON 
---

Angular allows for very useful modules that let you keep your data in organized places.
For example, if we are building a blog. We can create a structure for every blog post that we will want to have and use the ng-repeat directive to iterate and post our blog posts.
For the frontend, a simple json can hold our titles, content and an image. We can store these key value pairs and use the http module to get the data in our json file.
However, because JSON's can only hold strings we can't write our blog posts how we normally would, and expect to have any sort of structure in our blog post.
We might instead decide to inject html through our controller, allowing us to write modularized blog posts with line breaks in coherent areas. However, Angular does not directly
trust html being injected from another file, through an HTTP request.


Angular is trying ot proptect us from cross origin attacks, and injecting html from a "random" file in our computer could lead to security risks.
As such, we will need to tell angular that we trust the html that we are planning on injecting into our site.

We are going to need to inject some dependencies and modules into our code to get angular to trust us.
ngSanitize and $sce should do the trick.

In our app.js file:
{% highlight javascript %}

var myApp = angular.module('app',[
  'postController',
  'ngSanitize']);
{% endhighlight %}

We want to inject the ngSaznitize module and the postController module in our app so our app.js knows what to reference when our index.html uses it.

In our postController:

{% highlight javascript %}

var postController = angular.module('postController',['ngSanitize']);

postController.controller('postController',['$scope', '$http', '$sce',
   function($scope, $http, $sce){
    $http.get('data/posts.json').success(function(data){
      $scope.blogPosts = data;
    });
   }]);

{% endhighlight %}

We want to have our ngSanitize dependency and we pass $sce into our controller.

$sce (Strict Contextual Escaping) is a service that allows contextual escaping in your angular app. It tells angular that the get request you are about to run is safe and automatically escapes the values and allows tags to be injected into your HTML.

Now we can run our http get request to get data from our json file and set our $scope object to have the relevant data in the blogPosts key.

By having the $sce in our function, we implicitly tell angular that we can trust the data located within. In order to access the injected html
We have to ng-bind our data.
In our index.html: 
{% highlight html %}

            <ng-model="blogPosts">
              <section ng-repeat="blogPost in blogPosts">
                
                <p class="lead" ng-bind-html="blogPost.content"> 
                </p>
{% endhighlight %}


Now our ng-bind-html directive directly injects html with no questions asked depending on what is in our "content" key of our blogPost JSON object. 

Angular helps with modularized code, and being able to hold a structure in place and loop through our blog posts is pretty helpful.
