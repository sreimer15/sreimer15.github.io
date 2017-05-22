---
layout: post
title:  "Solving Power Set with Reduce"
date:   2016-02-01 19:59:21 -0800
categories: Algorithms Recursion Functional-Programming Reduce
---

An interesting problem that I had to solve recently was how to get every subset of any given input array. Generally this outputting of every subset is called the powerset of a given array. A powerset of an array is an array of all possible sub arrays. JavaScript has built in methods tied to the Array object, one of the popular ones for accumulating aspects from an array into a single value is reduce. This is the solution:

{% highlight javascript %}

var powerSet = function(array) {
    return array.reduce(function(previous, current) {
      var copy = previous.slice();
          copy.forEach(function(e) {
          var e2 = e.slice();
          e2.push(current);
          previous.push(e2);
        });
      return previous;
    }, [[]] );
};

{% endhighlight %}




When solving these types of problems one of the best methods we can implement is trying to solve it for the simplest scenario possible.

Skipping the empty array scenario, imagine if we have an array with one element:

What would be the powerset of this array.
var array = [1];

It might be tempting to say that the subset of the array would just be itself. However, [1] actually has two sub array’s. (an empty array) [] and [1]. 
If this is the case, we can imagine that the sub array of any array includes an empty array. In more complicated arrays i.e. [1,2] the subset would involve different combinations of elements:

[],
[1],
[2],
[1,2]

Knowing that our function will always need at least one empty array as a part of our answer, there is one function given to us in javascript, in line with functional programming principles that does really well with starting with an empty array.

Reduce takes an accumulator and a starting value. Since all of our results will be along the lines of [[]] + combinations our reduce function can give us an intuitive way to build up our powerSet Function

we want to create a copy of our array because we want to build up our array 2 times iteratively. First we want the elements then we want to use those elements to build the rest of the array.

Now we use forEach to go through our copy and we need to create an array of arrays, so we take each element and we create an array out of it. 
Our current variable is the current item in our original array, and since we have a copy of an array we can push it into our copy.
Now since all subsets contain themselves we need to push our new array into the final array, as one of the subsets of arrays.

Reduce allows us to keep track of this accumulator (here it is called previous), and it allows us to build larger and larger arrays which is exactly what we want!
