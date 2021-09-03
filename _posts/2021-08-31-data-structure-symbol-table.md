---
layout: post
title: Binary Search Tree
date: 2021-08-25 00:00:00
categories: [algorithm, data structure]
tags: [beginner, C++, code]
last_modified_at: 2021-08-25
---

# Symbol Table
  The term symbol table(dictionaries) use to describe an abstract mechanism where we save information
 (*a value*) that we can later search for and retrieve by specifying *a key*.  
Three classic data structures that can support efficient symbol-table implementations:
binary search trees, red-black trees, and hash tables.  
  These data structures support key-value pairs that provide two main operations: insert (put) a 
new pair into the table and search for (get) the value associated with a given key.

{% highlight C++ %}
#include <map>

std::map<std::string, int> TABLE{{"key1", 1}};
TABLE.insert({"key2", 2});
{% endhighlight %}

## 1. Elementary implementation
  We could create a symbol table by using:
* Linked list implementation (each Node has Key and Value): this kind of implementation creates an *unordered sequence*. And put, get, delete, contains operations have complexity O(N)
* Double array implementation (2 arrays for Keys and corresponding Values): the input key shall be *sorted(ordered sequence)*, so the operations shall be faster since we can use binary search on sorted array O(logN).

## 2. Binary Search Tree
  A binary tree is a data structure which arrange element in a tree like structure (root with branch to 2 nodes).
Every nodes in tree is a root of a subtree, each node has a key and every node's key is:
* larger than all keys in its left subtree
* smaller than all keys in its right subtree
![bst](/assets/img/blogs/2021_09_04/bst.png)
A node is comprised of 4 fields:
* Key
* Value
* reference to root of left subtree
* reference to root of right subtree

## 3. Hash map
