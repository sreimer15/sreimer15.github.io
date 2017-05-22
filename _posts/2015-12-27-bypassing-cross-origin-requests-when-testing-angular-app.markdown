---
layout: post
title:  "Bypassing Cross Origin Requests When Testing your Angular App"
date:   2015-12-27 19:59:21 -0800
categories: Angular Cross-Origin
---

Angular can be a very efficient way of managing your file-structure and cleaning up messy code, but adds some extra steps when it comes to loading your page for testing purposes. 

When serving files from our local disk, we run into problems with Cross Origin Requests. Browsers like Firefox have manners of bypassing local Cross Origin Requests, and we have optional bypassing in Google Chrome. However, instead of going into your settings we can test our app by simply running an npm module straight from our terminal. 

If we have node installed, we can open our command line, go into the root directory of our project and run 

<pre> >> npm install http-server --save </pre> <br> 

This installs the http-server node module, allowing us to bypass cross origin requests and serve our files from an http server. Once Installed stay in your root directory and run: 

<pre> >> http-server </pre> <br> 

This will serve up all the files in your directory, which is what you should want! 


