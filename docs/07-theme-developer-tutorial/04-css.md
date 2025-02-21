---
title: Theme Developer Tutorial: 4. CSS in Themes
short_title: "4: CSS"
id: theme-developer-tutorial-css
---

Discourse uses [SCSS](https://sass-lang.com/) to simplify styling and increase maintainability. I won't get into the details of why it is like that. I'll just leave it at recommending that you use SCSS instead of CSS when possible in your themes. It scales a lot better and your future self will thank you!

##### SCSS variables

Because Discourse uses SCSS, you can use variables like so

```scss
$font-stack: Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
```

It's very easy to change one line at the top of your sheet to change either the color or font when compared to changing hard-coded colors for tens of different elements.

Additionally, you can use Discourse core variables in your theme. This includes color schemes and lot of other things. Going back to the "Hello World!" banner theme from earlier, we had this CSS

```scss
.hello-world-banner {
  height: 300px;
  width: 100%;
  background: red;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1em;
}

.hello-world {
  font-size: 8em;
  color: white;
}
```

Notice how the background color is hard-coded to `red` and the font color for the heading is hardcoded to `white`

Let's try to use the color scheme variables instead. These look like so

```css
var(--primary)
var(--secondary)
var(--tertiary)
var(--quaternary)
var(--header_background)
var(--header_primary)
var(--highlight)
var(--danger)
var(--success)
var(--love)
```

and the names should sound familiar to you because they are the same as here

![6|619x500, 75%](/assets/beginners-guide-36.PNG)

If we attempt to use those color instead of hard-coded values, we'd end up with this

```scss
.hello-world-banner {
  height: 300px;
  width: 100%;
  background: var(--quaternary);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1em;
}

.hello-world {
  font-size: 8em;
  color: var(--secondary);
}
```

and here's the result

![33|690x490, 65%](/assets/beginners-guide-37.PNG)

now, if the active color scheme changes, your theme will adjust itself magically!

![34|599x500, 74%](/assets/beginners-guide-38.PNG)

This is just an example of one of the variable types you can use in your theme. We have a lot more and there's a topic detailing those here:

https://meta.discourse.org/t/how-to-use-discourse-core-variables-in-your-theme/77551

##### A better way of finding CSS selectors

The amount of elements in Discourse can look a bit overwhelming from a re-styling stand point. However, this feeling is rooted in trying to use traditional approaches on a very modern web app like Discourse. You can honestly write 4-8k lines of pure "traditional" CSS and you'd barely be close to a full theme.

This is part of the reason why I recommend you use SCSS.

let's assume you want to style all the buttons in Discourse. Well, you can use DevTools and try to find every variation of every button and style it, or you can try a different approach. The approach is to reuse whatever you can.

##### Reuse Discourse SCSS

Let's try to restyle all the buttons in Discourse by using this approach

1. Open [DevTools](https://developers.google.com/web/tools/chrome-devtools/)
2. Highlight a button
3. Find the origin stylesheet for its styles
4. Copy the selectors

With screenshots

1.  ![35|690x339, 75%](/assets/beginners-guide-39.PNG)

2.  ![36|690x347, 75%](/assets/beginners-guide-40.PNG)

3.  ![37|690x369, 75%](/assets/beginners-guide-41.PNG)

4.  ![38|690x339, 75%](/assets/beginners-guide-42.PNG)

There, now you have organised SCSS selectors that match those used on Discourse core.

[details="Expand snippet"]

```scss
// --------------------------------------------------
// Buttons
// --------------------------------------------------

// Base
// --------------------------------------------------

.btn {
  display: inline-block;
  margin: 0;
  padding: 6px 12px;
  font-weight: 500;
  font-size: $font-0;
  line-height: $line-height-medium;
  text-align: center;
  cursor: pointer;
  transition: all 0.25s;

  &:active,
  &.btn-active {
    text-shadow: none;
  }
  &[disabled],
  &.disabled {
    cursor: default;
    opacity: 0.4;
  }
  .fa {
    margin-right: 7px;
  }
  &.no-text {
    .fa {
      margin-right: 0;
    }
  }
}

.btn.hidden {
  display: none;
}

// Default button
// --------------------------------------------------

.btn {
  border: none;
  color: $primary;
  font-weight: normal;
  background: $primary-low;

  &[href] {
    color: $primary;
  }
  &:hover,
  &.btn-hover {
    background: $primary-medium;
    color: $secondary;
  }
  &[disabled],
  &.disabled {
    background: $primary-low;
    &:hover {
      color: dark-light-choose($primary-low-mid, $secondary-high);
    }
    cursor: not-allowed;
  }

  .d-icon {
    opacity: 0.7;
    line-height: $line-height-medium; // Match button text line-height
  }
  &.btn-primary .d-icon {
    opacity: 1;
  }
}

// Primary button
// --------------------------------------------------

.btn-primary {
  border: none;
  font-weight: normal;
  color: $secondary;
  background: $tertiary;

  &[href] {
    color: $secondary;
  }
  &:hover,
  &.btn-hover {
    color: #fff;
    background: dark-light-choose($tertiary, $tertiary);
  }
  &:active,
  &.btn-active {
    @include linear-gradient($tertiary, $tertiary);
    color: $secondary;
  }
  &[disabled],
  &.disabled {
    background: $tertiary;
  }
}

// Danger button
// --------------------------------------------------

.btn-danger {
  color: $secondary;
  font-weight: normal;
  background: $danger;
  &[href] {
    color: $secondary;
  }
  &:hover,
  &.btn-hover {
    background: scale-color($danger, $lightness: -20%);
  }
  &:active,
  &.btn-active {
    @include linear-gradient(scale-color($danger, $lightness: -20%), $danger);
  }
  &[disabled],
  &.disabled {
    background: $danger;
  }
}

// Social buttons
// --------------------------------------------------

.btn-social {
  color: #fff;
  &:hover {
    color: #fff;
  }
  &[href] {
    color: $secondary;
  }
  &:before {
    margin-right: 9px;
    font-family: FontAwesome;
    font-size: $font-0;
  }
  &.google,
  &.google_oauth2 {
    background: $google;
    &:before {
      content: $fa-var-google;
    }
  }
  &.instagram {
    background: $instagram;
    &:before {
      content: $fa-var-instagram;
    }
  }
  &.facebook {
    background: $facebook;
    &:before {
      content: $fa-var-facebook;
    }
  }
  &.cas {
    background: $cas;
  }
  &.twitter {
    background: $twitter;
    &:before {
      content: $fa-var-twitter;
    }
  }
  &.yahoo {
    background: $yahoo;
    &:before {
      content: $fa-var-yahoo;
    }
  }
  &.github {
    background: $github;
    &:before {
      content: $fa-var-github;
    }
  }
}

// Button Sizes
// --------------------------------------------------

// Small

.btn-small {
  padding: 3px 6px;
  font-size: $font-down-1;
}

// Large

.btn-large {
  padding: 9px 18px;
  font-size: $font-up-1;
  line-height: $line-height-small;
}

.btn-flat {
  background: transparent;
  border: 0;
  outline: 0;
  line-height: $line-height-small;
  .d-icon {
    opacity: 0.7;
  }
}
```

[/details]

Once you make the changes to fit your needs and save, you will already have created a theme component that changes the way all buttons in Discourse appear.

Obviously, you'd ideally only save the selectors you intend on modifying and remove unchanged rules.

Much easier than finding all the buttons / selectors one by one, no? :wink:

##### Reuse Discourse classes

On a similar note, you can make your life a lot easier by using Discourse classes in your html instead of rewriting the styles. Here's what I mean, let's say you want to add a couple of buttons above the header. You'd start with something like

```html
<div>
  <button>Click me!</button>
  <button>Don't click me!</button>
</div>
```

in the Header section of a theme. This html by itself would look like this:

![39|671x500, 75%](/assets/beginners-guide-43.PNG)

Now, this doesn't look right, it needs to be styled. There are two ways to do this, you can either write the SCSS needed to style these new elements, which can take a bit of time, or you can simply reuse Discourse core classes.

For example, if you check the `#main-outlet` element which wraps around the entire content, you'll find it has the class `wrap`

![40|690x193, 70%](/assets/beginners-guide-44.PNG)

Now if we reuse that class, along with a couple of other classes in the example html we end up with this

```html
<div class="wrap">
  <button class="btn btn-primary">Click me!</button>
  <button class="btn btn-danger">Don't click me!</button>
</div>
```

and it looks a bit better, even though we haven't added any CSS.

![41|580x500, 75%](/assets/beginners-guide-45.PNG)

Once you're sure you can't add any more reusable classes from Discourse core, you can then write your custom css and classes like so

html

```html
<div class="wrap foobar">
  <button class="btn btn-primary">Click me!</button>
  <button class="btn btn-danger">Don't click me!</button>
</div>
```

SCSS

```scss
.foobar {
  display: flex;
  justify-content: flex-end;
  background: var(--secondary-high);
  padding: 0.5em 0;
  button {
    margin: 0.25em;
  }
}
```

![42|580x500, 75%](/assets/beginners-guide-46.PNG)

And again, since we have not hard-coded any colors in the design, it will follow the current active color scheme.

[quote="You,"]
Ok, but so far you've only demonstrated adding html in very limited areas like the header and footer. How do I add html to other places?
[/quote]

This is where Handlebars templates come in.
