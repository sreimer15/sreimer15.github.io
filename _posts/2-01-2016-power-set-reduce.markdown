---
layout: post
title:  "Solving Power Set with Reduce"
date:   2016-02-01 19:59:21 -0800
categories: Algorithms Recursion Functional-Programming Reduce
---

var powerSet = function(array) {
    return array.reduce(function(previous, current) {
      var copy = previous.slice();
          copy.forEach(function(e) {
          console.log(e)
          var e2 = e.slice();
          e2.push(current);
          previous.push(e2);
        });
      return previous;
    }, [[]] );
};

var array = [1,2,3];

console.log(powerSet(array));

An interesting problem that I had to solve recently was how to get every subset of any given input array. Generally this outputting of every subset is called the powerset of a given array. 

When solving these types of problems one of the best methods we can implement is trying to solve it for the simplest scenario possible.

Skipping the empty array scenario, imagine if we have an array with one element
var array = [1];
It might be tempting to say that the subset of the array would just be itself. However, [1] actually has two sub array’s. (an empty array) [] and [1]. If this is the case, we can imagine that the sub array of any array includes an empty array. In more complicated arrays i.e. [1,2] the subset would involve different combinations of elements. 
[]
[1]
[2]
[1,2]

Knowing that our function will always need at least one , there is one function given to us in javascript, in line with functional programming principles that does really well with starting with an empty array.

Reduce takes an accumulator and a starting value. Since all of our results will be along the lines of [[]] + Combinations our reduce function can give us an intuitive way to build up our powerSet Function

we want to create a copy of our array because we want to build up our array 2 times iteratively. First we want the elements then we want to use those elements to build the rest of the array.
