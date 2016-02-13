---
layout: post
title:  "UI Router Formulation Part-1 Without Child States"
date:   2016-01-10 19:59:21 -0800
categories: Angular UI-Router FrontEnd Design 
---

Angular has been a very popular framework for structuring front-end logic and views. Angular provides a lot of quality of life methods and functions to display data. Controllers with dynamic data and directives such as ng-bind to get data allows for extra modularity once we arrive at our preferred view. Two way data binding allows for views to have a lot of control over the data you display. As such, we can reuse views in a variety of places, and merely change the underlying controllers.
 
Because our views have so much power we need to be careful with what states they have access to. One popular and useful way to take control of what states our views have is with ui-router.
https://github.com/angular-ui/ui-router

Instead of having static headers and footers with content being dynamically changed through states.
We can have static ui-views whose meaning changes with the change of a state.
 
<!-- <div ui-view="header"></div>
<div ui-view="content"></div>
<div ui-view="footer"></div>
 --> 
The ui-view directive defines views that exist within a state. We can change the state using the ui-sref directive.
 
  <a ui-sref="home">Click to go to main</a>
  <a ui-sref="dashboard">Click to go to dashboard</a>

Our state holds the logic for what content and header actually means.
 
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
  .state('roadmapTemplate', {
  url: '/roadmaps',
  views: {
    'content': {
      templateUrl: 'app/roadmaps/roadmaps.content.html',
      controller: 'RoadMapsController'
    },
    'header': {
      templateUrl: 'app/main/main.header.html',
      controller: 'RoadMapsController'
    },
    'footer': {
      templateUrl: 'app/main/main.footer.html',
      controller: 'RoadMapsController'
    }
  }
  })
});

In part 2: Nested Views
 
 
 
 
Advantages:
This method of ui-routing forces us as developers to understand the metaness of both our specific website and websites as a whole. Many of our pages have headers footers and content. And it will occur in that order. Instead of having many pages with separate headers and footers lets swap the content in and out, not by changing our views by holding them in new states. But by merely changing the state, which has its own underlying content which implies our view will change. In this model our states define our views, not the views we want to display defining what state we should be in.
 
I am a user who wants to use the dashboard and need an interface for that
As opposed to
I am a user who wants a view of a dashboard, give me the state to see that.
 


