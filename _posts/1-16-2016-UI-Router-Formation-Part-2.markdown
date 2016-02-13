---
layout: post
title:  "UI Router Formulation Part-2 With Child States"
date:   2016-01-16 19:59:21 -0800
categories: Angular UI-Router FrontEnd Design 
---



Parent and Child States in ui-router are surprisingly difficult to set up. Small Differences in syntax can lead to poorly designed applications. 


One of the best features that ui-router has is the ability to pass views and their associated controllers into child states. Hypothetically, you could have one single  main state, and have every state change merely change the content portion of your index.html. We can generalize our websites as having a header, a footer, and a content.
Clearly, content may be very complex, and we can imagine a variety of different types of content that may exist.
But if we imagine content as containing a variety of subviews then we can disect any webpage into three parts. THe header the footer and the content. Just like everything, The two extremes and somewhere in the middle. And just like real life, we can keep the complexity nested in the middle, and have our ends be static no matter what states we pass onto all of our children.

If we structure our pages in this manner, we can define a main state that keeps our header and footer and only have substates that change the middle portion.

To do this in angular:

.state('home', {
      url: '/',
      views: {
        'content': {
          templateUrl: 'app/main/main.html',
          controller: 'MainController'
        },
        'header': {
          templateUrl: 'app/main/header.html',
          controller: 'MainController'
        },
         'welcome@home': {
          templateUrl: 'app/main/welcome.html',
          controller: 'MainController'
        },
         'featured': {
          templateUrl: 'app/main/welcome.html',
          controller: 'MainController'
        },
        'browse@home': {
          templateUrl: 'app/browse/browse.html',
          controller: 'BrowseController',
        },
        'footer': {
          templateUrl: 'app/main/footer.html',
          controller: 'MainController'
        }
      }
    })
    //state for sign-in
    .state('home.signin', {
      url: 'signin',
      views: {
        'content@': {
          templateUrl: 'app/auth/signin.html',
          controller: 'AuthController'
        }
      }
    })

Our sign in state will keep the header and footer but have a new content.
This is because the “.” symbol in ui-router programatically assigns the state to be a child state of home.
We have to name our content with the @ symbol in order to define what content will look like in our sign in state.


Our html looks like this 

<div ui-view=”header”></div>
<div ui-view=”content”></div>
<div ui-view=”footer”></div>


In our main parent state i.e. ui-sref=”main”
our content can have sub views like this

<div ui-view=”welcome”></div>
<div ui-view=”browse”></div>
<div ui-view=”header”></div>

This subviewing only worsk if we have the @ symbol convention to define the parent state that it refers to. THe convention is as follows CurrentView@parentStateName
Even though our content view may be a simple html page with those three ui-view tags without the @symbol our subviews won’t connect to their relevant files.


Now we can have more states that share the main page headers with their own unique content
    .state('home.signin', {
      url: 'signup',
      views: {
        'content@': {
          templateUrl: 'app/auth/signup.html',
          controller: 'AuthController'
        }
      }
    })




