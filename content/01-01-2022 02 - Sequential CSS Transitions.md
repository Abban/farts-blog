---
title: "Sequential CSS Transitions"
date: 2022-01-01T10:11:34+01:00
draft: false
description: "While making this journal I discovered a neat trick for animating list elements sequentially without having to use javascript."
post_type: "blog"
tags: [ "Professional" ]
---

While making this journal I discovered a neat trick for animating list elements sequentially without having to use javascript.

It involves [CSS Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) and [animation-delay](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-delay).

## The Trick
 
 What I didn't know before is that you can set a CSS variable as a html inline style and that can be picked up to be used by your stylesheets. So when you loop out elements you can do something like this:
 
 ```html
 <div v-for="(post, index) in posts" :key="index" :style="{'--index': index}" class="post">
     My post is in here here
 </div>
 ```
 
See the `--index` property that's being set? That will put an incremental number into your elements that can then be used in your stylesheets. Pretty cool. That example uses Vuejs to loop but you can do it however you want. So, in your CSS you can do this:

```css
.post {
    animation: showPosts 1s ease-in-out;
    animation-fill-mode: forwards;
    animation-delay: calc(0.1s * var(--index));
}
@keyframes showPosts {
    0% {
        transform: translateX(60px);
        opacity: 0;
    }
    100% {
        transform: translateX(0);
        opacity: 1;
    }
}
```

The `--index` property is per element so the first one will be 0, second 1 etc meaning the animation is delayed slightly more the further down the list it is. You can see it working on [my homepage](https://farts.ie).