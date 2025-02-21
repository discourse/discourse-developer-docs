---
title: Theme Developer Tutorial: 3. Creating your first theme
short_title: "3: Creating Themes"
id: theme-developer-tutorial-creating
---

#### Hello World (HTML / CSS)

Let's start with a basic local theme. We're going to add a big "Hello World!" banner under the Discourse header. Like I mentioned at the intro, this topic will not teach you how to write CSS / HTML so I won't be explaining those but here are the bits we're going to need for this theme.

html

```html
<div class="hello-world-banner">
  <h1 class="hello-world">Hello World!</h1>
</div>
```

CSS

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

So, since we want our banner here

![9|690x245, 75%](/assets/beginners-guide-9.PNG)

Then the best place to add the html for it would be the After Header section in the theme editor or here

![10|690x339, 75%](/assets/beginners-guide-10.PNG)

and the CSS goes here

![11|690x331, 75%](/assets/beginners-guide-11.PNG)

Be sure to hit the save button and then hit the **preview** button

and if we check to see...

![12|690x406, 75%](/assets/beginners-guide-12.PNG)

There! You've just created your first Discourse theme! :tada:

#### Hello World (JS)

We will now try to do something similar but with JS. Let's try to create an alert that says "Hello World!" when you first load Discourse.

The script we're going to need is

```html
<script>
  alert("Hello world!");
</script>
```

Now, since this is a script, we need be a little bit more careful where to add to ensure that it fires. You have one of three options:

![13|567x500, 75%](/assets/beginners-guide-13.PNG)

1. The `</head>` section
2. The Header section
3. The `</body>` section

Adding the script to any other section will cause it not to fire. I prefer to keep scripts in the `</head>` or Header sections. So, let's add it to the `</head>` section like so:

![14|690x373, 60%](/assets/beginners-guide-14.PNG)

Save, and check if it worked

![15|690x244, 60%](/assets/beginners-guide-15.jpg)

Hooray! :tada: Your second basic Discourse theme is done.

[quote="You,"]
great... but these are all local themes. How do I create a remote theme?
[/quote]

#### Hello World (Remote)

As we discussed earlier, remote themes live in Github repositories. So, let's go ahead and create a new repository and add a license to it

![15|690x472, 60%](/assets/beginners-guide-16.PNG)

When in doubt, Select MIT for the license.

Now we have a repository! But it's a bit...empty. So let's fix that. We're going to recreate the (HTML / CSS) "Hello World!" theme you created locally earlier.

If you remember, We added

```html
<div class="hello-world-banner">
  <h1 class="hello-world">Hello World!</h1>
</div>
```

To the After Header subsection in the common section

![10|690x339, 60%](/assets/beginners-guide-17.PNG)

So we need to do the same with your first remote theme. If you recall from earlier, I mentioned that if you need to add html to the common After Header section of a theme, you need your remote repository to contain a folder named `common`, with a file named `after_header.html` in it. So, let's create that and add the markup for the "Hello World!" banner there.

![17|485x499, 88%](/assets/beginners-guide-18.PNG)

Then commit the new file.

We also need to add the banner CSS. So, we need to create a file named `common.scss` in the same `common` folder and add the CSS to it. So this

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

Is added like so:

![18|485x500](/assets/beginners-guide-19.PNG)

You can then commit the new `common.scss` file.

Now, your first remote theme does not have any settings or assets, so the only file left for you to make is the `about.json` file. As we discussed before. `about.json` files are **required** for remote themes to work.

[quote="You,"]
Ok great, what do I put in that about.json file then?
[/quote]

This:

```json
{
  "name": "My first remote theme",
  "about_url": "https://some.url.com",
  "license_url": "https://github.com/GitHubUsername/my-first-remote-theme/blob/master/LICENSE"
}
```

1. the name of your theme
2. The "About" URL for your theme, which shows up here for users of your theme:

![19|566x499, 75%](/assets/beginners-guide-20.PNG)

3. The URL for the theme's license, which you can get by clicking on the license file of your repository (or you can use any license URL you have like [here)](https://desandro.mit-license.org/)

![20|435x500, 98%](/assets/beginners-guide-21.PNG)

The link to the theme's license show's up here in for users of your theme:

![21|568x500, 75%](/assets/beginners-guide-22.PNG)

So, now you know what to add to the `about.json` file of your theme, let's go ahead and make one. **It needs to be at the root of your repository.**

![22|485x500, 88%](/assets/beginners-guide-23.PNG)

Commit the `about.json` file and your theme should be ready to be imported.

Let's try to import your first remote theme. Copy the repository's link from here

![23|517x500, 82%](/assets/beginners-guide-24.PNG)

Then go to the theme editor and click on Import here

![24|567x500, 75%](/assets/beginners-guide-25.PNG)

Then on "From the web" and paste the repository link in the input like so

![25|690x498, 60%](/assets/beginners-guide-26.PNG)

Click on Import...and... Magic!

![26|568x500, 75%](/assets/beginners-guide-27.PNG)

Try to preview your first remote theme to make sure everything looks right by clicking here

![27|568x500, 75%](/assets/beginners-guide-28.PNG)

and....:drum:

![12|690x406, 62%](/assets/beginners-guide-29.PNG)

There it is! Your first remote Discourse Theme! :tada::tada:

You can share the repository URL with anyone and they will be able to install your theme with only a few clicks but we can take this a step further!

[quote="You,"]
What do you mean?
[/quote]

This.

#### Creating previews on Theme Creator

Not long ago, we introduced [Theme Creator](https://meta.discourse.org/t/theme-creator-create-and-show-themes-without-installing-discourse/84942)

Theme creator is a tool that allows theme developers to

1. Create themes (without installing Discourse)
2. Have content to test themes on (not so easy on an empty install)
3. Create previews for themes (super easy to show your work!)

[quote="You,"]
So how do I use it?
[/quote]

While logged in here on Meta, visit

https://theme-creator.discourse.org

hit login... done!

You now have an account on theme creator and can create and share themes.

Once logged in, you'll see this:

![28|690x466, 75%](/assets/beginners-guide-30.jpg)

Now click on My themes and you'll be taken to a familiar interface:

![29|690x490, 75%](/assets/beginners-guide-31.PNG)

The reason this interface looks familiar is that it's the same you would see on your own Discourse install. However, with theme creator, you now have access to it even if you don't have Discourse installed!

Now, let's try to import your first remote theme to theme creator like we did earlier

![25|690x498, 60%](/assets/beginners-guide-32.PNG)

Now, let's create a preview link for your theme on theme creator. All you need is to click here and give it a name.

![30|690x489, 75%](/assets/beginners-guide-33.PNG)

copy the link

![31|690x260, 75%](/assets/beginners-guide-34.PNG)

Done! now you can share this link with anyone and they will be able to preview your theme on a live Discourse install

Here's mine:

[https://theme-creator.discourse.org/theme/Johani/my-first-remote-theme](https://theme-creator.discourse.org/theme/Johani/my-first-remote-theme)

Now, when users click that link they will see

![32|690x491, 75%](/assets/beginners-guide-35.PNG)

Very cool, huh? :wink::+1:

#### Develop and preview changes live on Theme Creator

With our [theme command line tool](https://meta.discourse.org/t/discourse-theme-cli-console-app-to-help-you-build-themes/82950) (Theme CLI) you can work on your theme locally and preview your changes as you make them on the Theme Creator site (or on a [local dev install](https://meta.discourse.org/tags/dev-install)).

To learn more about how, check out the https://meta.discourse.org/t/beginners-guide-to-using-theme-creator-and-theme-cli-to-start-building-a-discourse-theme/108444
