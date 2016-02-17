---
layout: post
title:  "UI Router Formulation Part-1 Without Child States"
date:   2016-01-10 19:59:21 -0800
categories: Angular UI-Router FrontEnd Design 
---

Angular has been a very popular framework for structuring front-end logic and views. Angular provides a lot of quality of life methods and functions to display data. Controllers with dynamic data and directives such as ng-bind simplifies our requests to the backend for data giving us extra modularity once we arrive at our preferred view. Two way data binding allows for views to have a lot of control over the data you display. As such, we can reuse views in a variety of places, and merely change the underlying controllers.
 
Because our views have so much power we need to be careful with what states they have access to. One popular and useful way to take control of what states our views have is with ui-router.
https://github.com/angular-ui/ui-router

Instead of having static headers and footers.
We can have static ui-views whose meaning changes with the change of a state.

{% highlight HTML %} 
  <div ui-view="header"></div>
  <div ui-view="content"></div>
  <div ui-view="footer"></div> 
{% endhighlight %}

The ui-view directive defines views that exist within a state. We can change the state using the ui-sref directive. Clicking one of these links will change the state of our app.
Our new states will have their own headers or content.
{% highlight HTML %} 
  <a ui-sref="home">Click to go to main</a>
  <a ui-sref="dashboard">Click to go to dashboard</a>
{% endhighlight %}

Our state holds the logic for what content and header actually means.
 {% highlight javascript %} 
  .state('home', {
    url: '/',
    views: {
      'header': {
        templateUrl: 'app/main/main.header.html',
        controller: 'MainController'
      },
      'content': {
        templateUrl: 'app/main/main.content.html',
        controller: 'MainController'
      },
      'footer': {
        templateUrl: 'app/main/main.footer.html',
        controller: 'MainController'
      },
      'signup': {
        templateUrl: 'app/auth/signup.html',
        controller: 'AuthController'
      },
      'signin': {
        templateUrl: 'app/auth/signin.html',
        controller: 'AuthController'
      }
    }
  })
  .state('exampleTemplate', {
  url: '/examples',
  views: {
    'content': {
      templateUrl: 'app/examples/examples.content.html',
      controller: 'examplesController'
    },
    'header': {
      templateUrl: 'app/main/main.header.html',
      controller: 'examplesController'
    },
    'footer': {
      templateUrl: 'app/main/main.footer.html',
      controller: 'examplesController'
    }
  }
  })
});
{% endhighlight %}

Now we can have new content, or reuse old content in our different states.

Using Ui-Router we now have a structured way of defining dynamic views. However, this isn't the canonical way of setting up reusable templates. In part 2 we will talk about Nested Views, and how we can define parent and child states, in order to take our angular app structure to the next level.