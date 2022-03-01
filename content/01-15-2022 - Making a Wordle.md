---
title: "Making a Wordle"
date: 2022-01-15T10:11:34+01:00
draft: false
description: "Wiki wiki wild wild wordle."
post_type: "blog"
tags: [ "Professional" ]
---

[I made a spin off of Wordle](https://squirdle.farts.ie) as a programming exercise on my parental leave. This is a document of the things I learned. The repository is [here](https://github.com/Abban/squirdle).

Learnings follow.

## Pseudorandom Generation

The daily word is selected from a list [pseudorandomly](https://github.com/Abban/squirdle/blob/4f918a213efdd107570ba06043e0f256045ea517/src/library/wordResource.ts#L5) using a seed generated from the current day's date.

## Lazy Local Storage

There are a few things stored in local storage, and a handy way to not have to keep checking that the items exist without too much code branching is to [lazy load](https://github.com/Abban/squirdle/blob/main/src/library/statsResource.ts) it.

The lazy loader can also be used to automatically [create the new game state item](https://github.com/Abban/squirdle/blob/4f918a213efdd107570ba06043e0f256045ea517/src/library/gameStateResource.ts#L22) for the word if the player isn't currently on the same day as the existing storage item.

## Heavy top level App Controller

The Vuejs [top level App component](https://github.com/Abban/squirdle/blob/main/src/App.vue) became a bit large, and I would say it's almost a god class. Next time, I'd try and pull functionality out of it.

## Typescript

I decided to use Typescript as I like types. I couldn't imagine making something even as small as this without it.

## Mobile Safari and vh
You can't use 100vh as the height of an element in mobile Safari because [they broke it on purpose!](https://bugs.webkit.org/show_bug.cgi?id=141832#c5)

