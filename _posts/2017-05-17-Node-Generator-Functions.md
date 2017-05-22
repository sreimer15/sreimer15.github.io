---
layout: post
title:  "Creating a CLI App with ES6/7 Generators and Async Await"
date:   2017-05-17 19:59:21 -0800
categories: Node.js Express Generators ES6/7ObjectID BackEnd RESTful-API
---

Node's more recent versions expose two very powerful api's that make asynchronous programming more intuitive Async/Await and Generators. By combining the two we can create intuitive UI's. In this two part series we are going to talk about Generator functions first and then Async/Await [Generating-Promises/Async-Await] We are going to use the two to create a CLI that will simulate getting posts and user information. We are going to do this by using [JSON API] to mock our posts comments and users. In order to build our CLI we are going to use the readline package to more easily provide an interface from inputs to outputs. The bulk of our generator function is going to provide the user feedback on load. 

We have previously defined renderer and blog functions which we shall use to render lines of text to the screen and to retrieve data using HTTP verbs. We used an interesting patten of generating promises and creating different types of wrappers in order to make some of our code more intuitive [Generating-Promises/Async-Await], which we described in a previous blog post. We created these functions because Async/Await is merely a wrapper of the .then promise API, so we want to generate promises that we can await from in our tests as well as our blog. 

```js
import blog from './blog';
import renderers from './renderers';

const { renderProfileInfo, viewingPostWithComments, userCommentFeedback } = renderers;
const { getUserProfileInfo, getPostsWithComments, postComment } = blog;
```

Let's first display the entirety of the example assuming we have our renderers and blog controller functions from above.


```js
const rl = readline.createInterface(process.stdin, process.stdout);

function handleFeedback(message, renderer) {
  if (message.errorMessage) {
    console.log(renderer(message));
    rl.close();
    return;
  } else {
    console.log(renderer(message));
    return renderer(message);
  }
}

async function *blogGenerator() {
  const userId = yield 'pass me a user id to find: ';
  const userProfileInfo = await getUserProfileInfo(userId);
  handleFeedback(userProfileInfo, renderProfileInfo);
  const postId = yield 'which post id do you want me to find? ';
  const postsWithComments = await getPostsWithComments(postId);
  handleFeedback(postsWithComments, viewingPostWithComments)
  const comment = yield 'what comment do you want to post? ';
  const name = yield 'what name do you want to post under? ';
  const commentObj = { body: comment, postId, name: name }
  const commentPost = await postComment(commentObj);
  handleFeedback(commentPost.body, userCommentFeedback)
  console.log('thank you for posting!')

}
async function runBlog() {
  const newBlog = blogGenerator();

  const firstPrompt = await newBlog.next();

  rl.setPrompt(firstPrompt.value);
  rl.prompt();
  rl.on('line', async (line) => {
    const newPrompt = await newBlog.next(line);
    if (newPrompt.done) {
      rl.close();
    }
    rl.setPrompt(newPrompt.value);
    rl.prompt();
  }).on('close', () => {
    console.log('thank you for trying!');
    process.exit(0);
  })
}

runBlog();

```

In order to use Async/Await in a function we have to define it using the async keyword. Since we also want it to be a generator function we use the * synatx as well. This gives us access to the yield keyword as well. 

```js
async function *runBlog() { ... }
```
The keyword yield pauses our function and creates a function invocation that the caller of the function can call with .next().  Within our function declaration yield also takes a value that our callee will have on .next() ing the generator function. .next() returns an object that has a .done and a .val property, this allows the user of hte function to see whether there is more to the function  (by calling .done) and also pull out a value from it with .value. 

This is what gives our generator its namesake. On declaration we can define breakpoints in our function with yield, the callee can get the information it needs from the value and done properties, and can provide input back into the function by passing a value to .next(value). Think of .next as going to the next yield statement, and running whatever logic is necesssary that the callee doesn't have to be concerned about. The generator pattern provides a form of communication between function and client, which can be really useful in certain applications. Back to the code!

In our generator, we can define simple strings that will feed right into our readline prompts. Furthermore the callee, can pass in a value to the next function meaning that at function declaration we can wrangle our asynchronous code and create a back and forth between user and program. 
```js
const userId = yield 'pass me a user id to find: ';
// userId is the result of our calleee calling .next();
// if we imagine hte following calleee
function callee() {
  generator.next('hello');
}

// later in the generator function function
// userId would equal hello
const userProfile = await getUserProfileInfo(userId);
// so on and so forth
```

Reading the code, once you get used to the yield syntax the generator function lets us seperate user interaction and backend interaction. This functional style also means our code does not rely on global variables and will perform the same way every time.

Now is a good time to talk about what readline is doing in creating our cli.
```js
async function runBlog() {
  const newBlog = blogGenerator();

  const firstPrompt = await newBlog.next();

  rl.setPrompt(firstPrompt.value);
  rl.prompt();
  rl.on('line', async (line) => {
    const newPrompt = await newBlog.next(line);
    if (newPrompt.done) {
      rl.close();
    }
    rl.setPrompt(newPrompt.value);
    rl.prompt();
  }).on('close', () => {
    console.log('thank you for trying!');
    process.exit(0);
  })
}
```

we first instantiate our blogGenerator as newBlog. This will give us a reference to constantly pull values out of in our main thread.
```js
const newBlog = blogGenerator();

const firstPrompt = await newBlog.next(); // pass me a user id to find
```
Our firstPrompt variable is now a simple string that we can bring to the user by using readline's setPrompt and then prompt() functions. We can then start a continous loop by using rl's on('line') event handler. This line handler will occur whenever the user types in an answer to one of our prompts, creating a feed back loop. Our base case is the done property of our generator function, which will trigger our close handler closing our interaction.

```js
  // sets prompt to: pass me a user id to find
  rl.setPrompt(firstPrompt.value);
  // displays it to the console.
  rl.prompt();
  rl.on('line', async (line) => {
    // line is the user's input
    // pull our object from the generator function
    const newPrompt = await newBlog.next(line);
    // if we are at the end of our generator function .done will be true and we can close out
    if (newPrompt.done) {
      rl.close();
    }
    // otherwise our generator function will return a new string to continue our interaction
    rl.setPrompt(newPrompt.value);
    rl.prompt();
  }).on('close', () => {
    console.log('thank you for trying!');
    process.exit(0);
  })  
```

Generator functions provide a very clean flow for handling user interaction. I am going to talk more about how async await is working in the next blog post [Generating-Promises/Async-Await]. Async Await is our portal into allowing our code to look very synchronous and succint.

[JSON API]: http://jsonapi.org/
[Generating-Promises/Async-Await]: github.com/sreimer15/async-await
[Generator Functions]:  github.com/sreimer15/
