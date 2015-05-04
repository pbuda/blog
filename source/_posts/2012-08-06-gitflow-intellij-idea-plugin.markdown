---
layout: post
title: "GitFlow Intellij Idea plugin"
date: 2012-08-06 13:37
comments: true
categories: 
---
I am creating a few side projects currently and I use git-flow Git extension for each of them, but I lack the support from Intellij Idea in this regard – mainly the starting and stopping of features and releases.

So because of that I’ve started a small project to include said functionality in Idea. It’s nothing major as apparently Jetbrains will include such plugin in some version of Intellij, but I need it now and I’m interested in how to write plugins for Idea 
What will the plugin be able to do?

git flow init -d
git flow feature start/finish
git flow release start/finish
The first version I will release will be able to do 1 and 2 while option 3 will be available at some later time. But I don’t know yet when I will release it, so stay tuned if you’re interested.

The project is of course at [GitHub](https://github.com/pbuda/gitflowidea)