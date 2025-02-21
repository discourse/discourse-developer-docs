#### The pluginAPI

In a nutshell, the PluginAPI is an easy way for you to write JS / jQuery and make Discourse do <s>things</s> amazing things!

[quote="You,"]
Like what?
[/quote]

Here are a couple of examples.

[Brand header theme component:](https://meta.discourse.org/t/brand-header-theme-component/77977)

This theme component uses the pluginAPI to create new widgets. Those widget include a brand header above the Discourse header and a new hamburger menu on mobile. It looks like this:

![49|578x499, 75%](/assets/beginners-guide-53.PNG)

[Discourse category banners:](https://meta.discourse.org/t/discourse-category-banners/86241)

This component uses the pluginAPI to create dynamic banners and place them at the top of each category page, it automatically fetches the category name, description and color and it looks like this:

![50|577x500, 75%](/assets/beginners-guide-54.PNG)

[quote="You,"]
So, where do I start if I want to use the pluginAPI?
[/quote]

You start here:

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/lib/plugin-api.gjs

This is the file in the Discourse repository where all the pluginAPI methods are defined.

I will go through the one's you're most likely to use and provide examples as a way of explaining how they work, but first, a bit of background.

When you use the pluginAPI, you have to use special `<script>` tag attributes in order for things to work properly. You're probably very used to seeing things like

```html
<script>
  alert("Hello world!");
</script>
```

or

```html
<script type="text/javascript">
  alert("Hello world!");
</script>
```

And what you need for the pluginAPI is this

```html
<script type="text/discourse-plugin" version="0.8">
  alert('Hello world!')
</script>
```

The two new things here are the `type` attribute and the `version` attribute.

The `type` attribute is self explanatory. The `version` attribute helps you ensure stability. Say for example you create a theme that uses a pluginAPI method that was just introduced to Discourse core. You would then set the version to one that matches the core pluginAPI version number that introduced the new method.

[quote="You,"]
Why would I need to do that?
[/quote]

In case someone with an outdated Discourse installs your theme. They would then see a message in the console instead of their site breaking.

![51|690x46, 75%](/assets/beginners-guide-55.PNG)

Which brings me to the next point.

##### console.log is your friend!

I cannot begin to emphasize the importance of using `console.log()` enough. If you're ever lost, or not sure about something always use `console.log()`

Let's try this

```html
<script type="text/discourse-plugin" version="0.8">
  console.log(Discourse)
</script>
```

What I've done here is logged `Discourse` which is a global object to the console. Now if I check and see what I get

![52|690x384](/assets/beginners-guide-56.PNG)

you'll notice a vary large amount of things are now available to me to use, for example, site settings, which I highlighted above.

But this is just a warm up, let's try a quick demo

```html
<script type="text/discourse-plugin" version="0.8">
  const settings = Discourse.SiteSettings,
    taggingEnabled = settings.tagging_enabled,
    title = settings.title;

  if (taggingEnabled) {
    console.log("Yay! "+title+" has tagging enabled!")
  } else {
    console.log("Ohh noes! "+title+"Does not allow tagging.")
  }
</script>
```

As it turns out, Theme creator does allow tags to be used, so if I check the console

![53|559x93, 75%](/assets/beginners-guide-57.PNG)

So, I'll say this one more time for good measure, if you're lost at any point use

```js
console.log($(this));
```

and check the console to see what you have to work with.

So, with all of that out of the way, here are the methods currently in the pluginAPI

##### getCurrentUser()

This method allows you get information about the current user. if we try something like this

```html
<script type="text/discourse-plugin" version="0.8">
  const user = api.getCurrentUser();

  console.log(user)
</script>
```

We can easily find things like the current user's username. Now let's try to do something with that information on our previous "hello world!" banner.

```html
<script type="text/discourse-plugin" version="0.8">
  $( document ).ready(function() {
    const user = api.getCurrentUser(),
      username = user.username;

    $('h1.hello-world').html('Hello there '+username+"!")
  });
</script>
```

![image|579x499](/assets/beginners-guide-58.png)

Where `username` will be dynamic and will match the current user's username.

There's a lot more than username for you to use, but I wanted to keep it simple. Use

```html
<script type="text/discourse-plugin" version="0.8">
  const user = api.getCurrentUser();

  console.log(user)
</script>
```

Check the console and see what else you can use.

##### replaceIcon()

```js
api.replaceIcon(source, destination);
```

With this method, you can easily replace any Discourse icon with another. For example, we have a theme component that replaces the heart icon for like with a thumbs-up icon

https://meta.discourse.org/t/change-the-like-icon/87748

that uses something like this

```js
api.replaceIcon("heart", "thumbs-up");
```

And it looks like this

![55|413x78, 75%](/assets/beginners-guide-59.PNG)

##### modifyClass()

You can use this method to extend or overwrite methods in a class like a component or controller (read: [Ember Classes](https://guides.emberjs.com/release/object-model/classes-and-instances/)), but it's also a great way to get information and set variables.

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass('controller:composer', {
    actions: {
      newActionHere() { }
    }
  });
</script>
```

[quote="You,"]
WAIT! I'm so confused! components... controllers?!
[/quote]

Don't get confused by all the new terms here.

> Ember components are used to encapsulate markup and style into reusable content. Components consist of two parts: a JavaScript component file that defines behavior, and its accompanying Handlebars template that defines the markup for the component's UI.

For the purposes of this guide, Think of controllers in the same way.

Let's pick a controller and play around and see what we can achieve. In the Discourse repository, all controllers live here

https://github.com/discourse/discourse/tree/master/app/assets/javascripts/discourse/app/controllers

I'm going to pick the [composer controller](https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/controllers/composer.js) and we're going to try to capture every keypress in the editor. First, let's take a look at what's available for us to use.

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/controllers/composer.js#L578

This looks very close to what I want to achieve, so we start with this

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass("controller:composer", {

  });
</script>
```

Then add the action we want to overwrite as is

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass("controller:composer", {
    actions: {
      typed() {
        this.checkReplyLength();
        this.get("model").typing();
      }
    }
  });
</script>
```

And then we finally add our change

```js
console.log("typed a letter");
```

Which should leave a message in the console at every keystroke

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass("controller:composer", {
    actions: {
      typed() {
        console.log("typed a letter");
        this.checkReplyLength();
        this.get("model").typing();
      }
    }
  });
</script>
```

And we test it

![56|690x147, 75%](/assets/beginners-guide-60.PNG)

Since this is also a thing you might be doing a lot of, let's go through another example.

This time we will try to capture when the user enters / loads the categories page. The categories page is a component. All the components in the Discourse repository live here

https://github.com/discourse/discourse/tree/master/app/assets/javascripts/discourse/app/components

and we can find the one we're after here

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/components/discovery-categories.js

so we're going to need something like this

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass("component:discovery-categories", {

  });
</script>
```

If you want to fire scripts when a component is loaded you can use something like this

```js
didInsertElement: function() {
  this._super();
  // do your work here
}
```

So we end up with this

```html
<script type="text/discourse-plugin" version="0.8">
  api.modifyClass("component:discovery-categories", {
    didInsertElement: function() {
      this._super();
      console.log("Welcome to the categories page!")
    }
  });
</script>
```

Now all that's left is to check and see if it works

![57|690x428, 75%](/assets/beginners-guide-61.PNG)

et voilà :tada:

We can now move on to doing something similar with widgets.

I mentioned earlier, that we'll cover widgets and so here's what you need to know about them before we move on to the next few methods.

[quote="eviltrout, post:1, topic:40347"]
A `Widget` is a class with a function called `html()` that produces the virtual dom necessary to render itself. Here’s an example of a simple `Widget` :
[/quote]

```js
import { createWidget } from "discourse/widgets/widget";

createWidget("my-widget", {
  tagName: "div.hello",

  html() {
    return "hello world";
  },
});
```

[quote="eviltrout, post:1, topic:40347"]
The above code registers a widget called `my-widget` , which will be rendered in the browser as `<div class='hello'>`
[/quote]

With this basic understanding, we can move on to the things you can do with widgets. There are three things you can do to widgets in Discourse themes.

1. Modify them - like we did with controllers and components
2. Decorate them - as in add elements before or after them
3. Create them from scratch

Well, it turns out the pluginAPI has a method for each of these. So let's have a look at those methods, but before we start. Here's where all the widgets live in the Discourse repository

https://github.com/discourse/discourse/tree/master/app/assets/javascripts/discourse/app/widgets

##### reopenWidget()

Let's start with the reopen widget method. This method is similar to what we did with controllers and components. I have a bit of a lengthy example, but it does demonstrate how much flexibility the pluginAPI offers you as theme developer. The example is the [Alternative Logo theme component](https://meta.discourse.org/t/alternative-logo-for-dark-themes/88502)

> This is a theme component that will allow you to add alternative logos for dark / light themes.

At its heart, this theme only overwrites one of the functions of the [home-logo widget](https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/widgets/home-logo.js)

So, we already know the name of the widget we want to reopen. it's `home-logo` and so we start with this

```html
<script type="text/discourse-plugin" version="0.8.13">
  api.reopenWidget("home-logo", {

  });
</script>
```

then find the function that we want to overwrite in that widget. Here I want to change the logo image so this looks promising.

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/widgets/home-logo.js#L31

So we copy that function as is first

[details="Expand snippet"]

```html
<script type="text/discourse-plugin" version="0.8.13">
  api.reopenWidget("home-logo", {
    logo() {
      const { siteSettings } = this;
      const mobileView = this.site.mobileView;

      const mobileLogoUrl = siteSettings.mobile_logo_url || "";
      const showMobileLogo = mobileView && mobileLogoUrl.length > 0;

      const logoUrl = siteSettings.logo_url || "";
      const title = siteSettings.title;

      if (!mobileView && this.attrs.minimized) {
        const logoSmallUrl = siteSettings.logo_small_url || "";
        if (logoSmallUrl.length) {
          return h("img#site-logo.logo-small", {
            key: "logo-small",
            attributes: {
              src: Discourse.getURL(logoSmallUrl),
              width: 33,
              height: 33,
              alt: title
            }
          });
        } else {
          return iconNode("home");
        }
      } else if (showMobileLogo) {
        return h("img#site-logo.logo-big", {
          key: "logo-mobile",
          attributes: { src: Discourse.getURL(mobileLogoUrl), alt: title }
        });
      } else if (logoUrl.length) {
        return h("img#site-logo.logo-big", {
          key: "logo-big",
          attributes: { src: Discourse.getURL(logoUrl), alt: title }
        });
      } else {
        return h("h1#site-text-logo.text-logo", { key: "logo-text" }, title);
      }
    }
  });
</script>
```

[/details]

And then adjust it according to the desired behavior. In my case it was like this

[details="Expand snippet"]

```html
<script type="text/discourse-plugin" version="0.8.13">
  api.reopenWidget("home-logo", {
    logo() {
      const { siteSettings } = this,
        { iconNode } = require("discourse/helpers/fa-icon-node"),
        h = require("virtual-dom").h,
        altLogo = settings.Alternative_logo_url,
        altLogoSmall = settings.Alternative_small_logo_url,
        mobileView = this.site.mobileView,
        mobileLogoUrl = siteSettings.mobile_logo_url || "",
        showMobileLogo = mobileView && mobileLogoUrl.length > 0;
      (logoUrl = altLogo || ""),
      (title = siteSettings.title);
      if (!mobileView && this.attrs.minimized) {
        const logoSmallUrl = altLogoSmall || "";
        if (logoSmallUrl.length) {
          return h("img#site-logo.logo-small", {
            key: "logo-small",
            attributes: { src: logoSmallUrl, width: 33, height: 33, alt: title }
          });
        } else {
          return iconNode("home");
        }
      } else if (showMobileLogo) {
        return h("img#site-logo.logo-big", {
          key: "logo-mobile",
          attributes: { src: mobileLogoUrl, alt: title }
        });
      } else if (logoUrl.length) {
        return h("img#site-logo.logo-big", {
          key: "logo-big",
          attributes: { src: logoUrl, alt: title }
        });
      } else {
        return h("h1#site-text-logo.text-logo", { key: "logo-text" }, title);
      }
    }
  });
</script>
```

[/details]

A quick summary of the changes in the above snippet

1. Replace the variables for the URL of the logo
2. Pull the value of those variables from the theme's settings - we'll talk about theme settings a bit later on.

You can do the same for any function in any widget using the `reopenWidget()` pluginAPI method.

We can now move on to another thing you can do to widgets

##### decorateWidget()

With this method you will be able to add html before or after a widget. You use it like this

```html
<script type="text/discourse-plugin" version="0.8">
  api.decorateWidget('NAME:LOCATION', helper => {

  });
</script>
```

Where name is the NAME of the widget and LOCATION is either `before` or `after` depending on where you want your HTML to show up - before or after the widget.

So this

```html
<script type="text/discourse-plugin" version="0.8">
  api.decorateWidget('post:after', helper => {
    return helper.h('p', 'Hello');
  });
</script>
```

Would add `<p>Hello</p>` after every "post" widget like so

![58|549x500, 75%](/assets/beginners-guide-62.PNG)

and this

```html
<script type="text/discourse-plugin" version="0.8">
  const { iconNode } = require("discourse-common/lib/icon-library");
  api.decorateWidget('header-icons:before', helper => {
      return helper.h('li', [
          helper.h('a.icon', {
              href:'https://foo.bar.com/',
              title: 'Foobar'
          }, iconNode('heart')),
      ]);
  });
</script>
```

would add

```html
<li>
  <a href="https://foo.bar.com/" title="Foobar" class="icon">
    <svg class="fa d-icon d-icon-heart svg-icon svg-node" aria-hidden="true">
      <use xlink:href="#heart"></use>
    </svg>
  </a>
</li>
```

right before the header icons widget.

![59|518x500](/assets/beginners-guide-63.PNG)

This leaves us with one last things you can do with widgets. Create them!

##### createWidget()

Now that you have a basic understanding of widgets, you're in a good place to create one! The pluginAPI has a method that makes this super easy.

Here's a basic example

```html
<script type="text/discourse-plugin" version="0.8">
  const h = require("virtual-dom").h;

  api.createWidget("my-first-widget", {
    tagName: "div.my-widget",

    html() {
      return h('h1', "Hello World!");
    }
  });
</script>
```

Start by requiring the relevant bit from the virtual dom library

```js
const h = require("virtual-dom").h;
```

This allows you to add html element in a more efficient way.

```js
h("h1", "Hello World!");
```

Is the exact same as

```html
<h1>Hello World!</h1>
```

And while it may look like the same amount of code for one element, it scales a lot better.

For a quick example, this - when set up to loop

```js
return h(
  "li.headerLink." + seg[3] + "." + seg[5],
  helper.h(
    "a",
    {
      href: seg[2],
      title: seg[1],
      target: seg[4],
    },
    seg[0]
  )
);
```

can produce

[details="Expand snippet"]

```html
<li class="headerLink vdo">
  <a href="/c/tech" title="Discussions about technology" target="">Tech</a>
</li>
<li class="headerLink vdo">
  <a href="https://www.vat19.com/" title="Buy some cool stuff" target="_blank"
    >Shop</a
  >
</li>
<li class="headerLink vdo">
  <a href="/t/284" title="Mobile OS poll" target="">Your Vote Counts!</a>
</li>
<li class="headerLink vdo keep">
  <a
    href="/latest/?order=op_likes"
    title="Posts with the most amount of likes"
    target=""
    >Most Liked</a
  >
</li>
<li class="headerLink vdm keep">
  <a href="/privacy" title="Our Privacy Policy" target="">Privacy</a>
</li>
```

[/details]

But is a lot more maintainable

[quote="You,"]
again with the blabbering... :roll_eyes:
[/quote]

Ok...fine let's go back to creating a widget. So now we have this

```html
<script type="text/discourse-plugin" version="0.8">
  const h = require("virtual-dom").h;

  api.createWidget("my-first-widget", {
    tagName: "div.my-widget",

    html() {
      return h('h1', "Hello World!");
    }
  });
</script>
```

Which creates the widget. and if you remember, we can mount widgets in Handlebars template. So we can do something like this

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/above-footer/inject-widget"
>
  {{mount-widget widget="my-first-widget"}}
</script>
```

and add bit of SCSS

```scss
@import "common/foundation/variables";

.my-widget {
  display: flex;
  align-items: center;
  justify-content: center;
  background: $primary-high;
  h1 {
    padding: 0.5em;
    color: $secondary;
    margin: 0;
  }
}
```

And we're done, your first Discourse Widget!

![60|546x499, 75%](/assets/beginners-guide-64.PNG)

Here are a couple more examples that you can look at to see `createWidget` in action

Brand header theme component:

[https://github.com/discourse/discourse-brand-header/blob/master/common/header.html](https://github.com/discourse/discourse-brand-header/blob/master/common/header.html)

Discourse Category banners theme component:

[https://github.com/awesomerobot/discourse-category-banners/blob/master/common/header.html](https://github.com/awesomerobot/discourse-category-banners/blob/master/common/header.html)

This covers creating, modifying and decorating widgets. The three things I said you can do to widget with Discourse themes, but I lied :lying_face:

There's actually one more thing you can do with some widgets, change their settings

##### changeWidgetSetting()

Some widgets like the `home-logo` or the `post-avatar` widgets have settings. If a widget has settings, you can easily change those settings with the pluginAPI

For example, the `post-avatar` widget has these

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/widgets/post.js#L157

and the `home-logo` widget has this

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/widgets/home-logo.js#L10

To, change a setting you can do something like this

```html
<script type="text/discourse-plugin" version="0.8">
  api.changeWidgetSetting('WIDGET-NAME', 'SETTING-NAME', 'VALUE');
</script>
```

So to change the avatar `size` setting in the `post-avatar` widget we can use

```html
<script type="text/discourse-plugin" version="0.8">
  api.changeWidgetSetting('post-avatar', 'size', '90');
</script>
```

and to change the `href` setting in the `home-logo` widget we would use

```html
<script type="text/discourse-plugin" version="0.8">
  api.changeWidgetSetting('home-logo', 'href', 'https://some.url.com');
</script>
```

##### addNavigationBarItem()

You use this method to add new items to the navigation bar here

![61|545x499, 75%](/assets/beginners-guide-65.PNG)

To add a new link you use something like this

```html
<script type="text/discourse-plugin" version="0.8">
  api.addNavigationBarItem({
    name: "link-to-movies-category",
    displayName: "movies",
    href: "/c/movies",
    title: "link title"
  })
</script>
```

To add a link to the "movies" category in the navigation menu like so

![62|547x500, 75%](/assets/beginners-guide-66.PNG)

##### addUserMenuGlyph()

You use this method to add a new linked icon to the user menu

![63|690x239, 60%](/assets/beginners-guide-67.PNG)

Here's an example, let's say you want to add a link to the users mentions page, you would then use something like

```html
<script type="text/discourse-plugin" version="0.8">
  api.addUserMenuGlyph({
    label: 'Mentions',
    className: 'mention-link',
    icon: 'at',
    href: '/my/notifications/mentions'
  });
</script>
```

Which creates a new icon that takes the user to their mentions page when clicked

![64|690x226, 60%](/assets/beginners-guide-68.PNG)

##### decorateCooked()

Posts in Discourse are widgets, as such, the contents of a post will not be available for you to target with JS or jQuery. Luckily though, the pluginAPI provides you with a method to reach those contents. The basic usage for this method is

```html
<script type="text/discourse-plugin" version="0.8">
  api.decorateCooked($elem => $elem.children('p').addClass('foo-class'));
</script>
```

This will find all `<p>` tags in the cooked content of a post and add the class `foo-class` to them

![66|690x339, 60%](/assets/beginners-guide-69.PNG)

That's the basics of it, but if you need to do something a little bit more complicated, I suggest [writing your own jQuery mini plugin](https://learn.jquery.com/plugins/basic-plugin-creation/) like so

```js
var tiles_selector = '.cooked div[data-theme-tiles="1"]';
$.fn.dtiles = function () {
  this.each(function () {
    var update = function () {
      $(this).masonry({
        itemSelector: ".lightbox-wrapper",
        transitionDuration: 0,
      });
    };
    this.addEventListener("load", update, true);
  });
  return this;
};

api.decorateCooked(($elem) =>
  $elem.children(tiles_selector).dtiles().addClass("tiles-initialized")
);
```

This is how the [Tiles image gallery](https://meta.discourse.org/t/tiles-image-gallery/81950) theme component works.

##### onToolbarCreate()

This is the method that you would use to add new buttons to the composer toolbar.

![67|690x234, 70%](/assets/beginners-guide-70.PNG)

You would use it like this

```html
<script type="text/discourse-plugin" version="0.8">
  const currentLocale = I18n.currentLocale();
  if (!I18n.translations[currentLocale].js.composer) {
    I18n.translations[currentLocale].js.composer = {};
  }
  I18n.translations[currentLocale].js.composer.my_button_text = "Hey there!";
  I18n.translations[currentLocale].js.my_button_title = "My Button!";

  api.onToolbarCreate(function(toolbar) {
    toolbar.addButton({
      trimLeading: true,
      id: "buttonID",
      group: "insertions",
      icon: 'heart',
      title: "my_button_title",
      perform: function(e) {
        return e.applySurround(
          '<div data-theme="foo">\n\n',
          "\n\n</div>",
          "my_button_text"
        );
      }
    });
  });
</script>
```

Note that the translation strings at the top are **required** and kudos to Simon Cossar for coming up with a way to [dynamically setting them based on the user's locale](https://meta.discourse.org/t/what-should-i18n-be-replaced-with-in-new-versions/90944/2?u=johani)

To add a button with a heart icon that wraps selected text with `<div data-theme="foo"></div>`

![68|690x185, 70%](/assets/beginners-guide-71.png)

<hr>

The pluginAPI has over 40 methods in it and so we'll stop here because I think the methods above are enough for the purposes of this guide.

However, this doesn't mean that you should not be aware of them. As mentioned before, you can find all the methods in the pluginAPI here in the Discourse repository

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/lib/plugin-api.gjs

