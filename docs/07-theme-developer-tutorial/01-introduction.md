---
title: Theme Developer Tutorial: 1. Introduction
short_title: "1: Introduction"
id: theme-developer-tutorial-overview
---

#### The structure of this tutorial

Since this topic is going to be long and will cover a wide variety of subjects, it's good to take a step back and describe its structure a bit. I will be using a lot of headings. The reason for this is that say at some point in the future you want a quick refresher (something that I do a lot), you can easily navigate to the section you need to look at. The headings for different sections are listed in the table of contents on the right side. Clicking on any of those items takes you to that section.

Finally, for the purposes of this guide, and unless otherwise specified, the terms "theme" and "themes" here refer to both themes and to theme components.

#### The scope of this topic

Let's start by highlighting what this topic is all about.

> Introduction to Discourse theme development for developers with little or no previous experience working on Discourse themes. Developers learn how to create Discourse themes that either modify the design of a forum or add new functionality. While this topic assumes no previous experience working on Discourse themes, it does assume some experience in the languages it covers.

Those languages are listed below - each is a link to a good place to read more about the language.

1. [HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)
2. [CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS)
3. [JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript) / [jQuery](https://learn.jquery.com/about-jquery/)
4. [SCSS](https://sass-lang.com/guide)  
5. [Handlebars](https://handlebarsjs.com/guide/)

It's also a good idea to [know your way around Github](https://guides.github.com/activities/hello-world/)

[quote="You,"]
What?! That's a lot of things and we haven't even started yet! :scream:
[/quote]

**Don't stress about it!** You won't need to learn / know all of these to create a theme. I have to include everything here for this guide to be a good reference point.

The beauty of Discourse themes is that they will go as far as you take them.

Want to create a simple CSS theme component that adds hover effects to titles?
**You can.**

Want to create a mega-complex monolith theme that uses SCSS / Handlebars / Ajax and completely overhauls all the things?
**You can.**

[quote="You,"]
So, why did you start with all of that then?
[/quote]

In order to scope this tutorial.

Think of it this way. This tutorial will **not** teach you how to write js conditional statements. It will, however, teach you how to find and feed Discourse-specific bits into conditional statements.

Ok, now we covered the scope and structure, we can move on to the bits you're actually interested in reading.
