---
title: Theme Developer Tutorial: 2. Overview of a Theme
short_title: "2: Overview"
id: theme-developer-tutorial-overview
---

#### What are themes anyway?

In a previous guide, I described themes / theme components as:

[quote]
A theme or theme component is a set of files packaged together designed to either modify Discourse visually or to add new features.
[/quote]

[quote]

#### Themes

In general, themes are not supposed to be compatible with each other because they are essentially different standalone designs. Think of themes like skins, or like launchers on Android. You can have multiple launchers installed but you can't use two of them at the same time. Your default Discourse installation comes with two themes
[/quote]

[quote]

#### Theme Components

We use the phrase theme-component to describe packages that are more geared towards customising one aspect of Discourse. Because of their narrowed focus, theme components are almost always compatible with each other. This means that you can have multiple theme components running at the same time under any theme. You can think of theme components like apps on your phone.
[/quote]

Those definitions come from the https://meta.discourse.org/t/beginners-guide-to-using-discourse-themes/91966 (probably worth glancing over if you haven't done so already) which is geared towards users of Discourse themes and not Discourse theme developers. However, this is a good base to start with.

[quote="You,"]
So, what's different from a developer's perspective?
[/quote]

Beyond the definitions above, here's what you need to know as a theme developer.

Themes can only amend the front end and have no access to the back-end. If this makes little to no sense to you, you don't need to worry about it for now.

#### Remote and local themes

Discourse themes and theme components can either be

1. Local
2. Remote

Local themes are themes created / stored on a Discourse install.

You create a brand new local theme by clicking here:

![2024-03-12_ss-install-theme|690x454, 75%](/assets/beginners-guide-1.jpeg)

...and then here:

![2024-03-12_ss-new-theme|475x500, 75%](/assets/beginners-guide-2.jpeg)

After making your changes. These can be exported and saved / shared by clicking here:

![2024-03-12_ss-export-theme|690x454, 75%](/assets/beginners-guide-3.jpeg)

...but sharing a file somewhere is not the ideal way to share a theme publicly, so we have remote themes.

Remote themes are Discourse themes that live in repositories on Github. This makes it easy to share themes. You create a theme and share the link to the repository, then users can install the theme using that link by clicking `Install` and then adding the repository location:

![2024-03-12_ss-import-theme-git|690x350, 50%](/assets/beginners-guide-4.png)

All the themes in the #theme categories are remote themes. Here's an example of what one of them looks like on [Github](https://github.com/discourse/discourse-brand-header)

![Capture2|690x481, 72%](/assets/beginners-guide-5.PNG)

#### Theme files and folders

Let's look at the interface for the theme editor:

![5|672x500, 75%](/assets/beginners-guide-6.PNG)

Notice how there are three main sections.

1. Common
2. Desktop
3. Mobile

As you may have already guessed, this allows your theme to target different device types, or apply your changes to both. Anything you change in the common tab will apply to desktop and mobile. Anything you change in the desktop or mobile tab will only apply to that respective device type.

You can preview the mobile view on a desktop device by appending `?mobile_view=1` to the end of the URL (`?mobile_view=0` switches back to Desktop).

Now let's look at the subsections under those.

1. CSS
2. `<head>`
3. Header
4. After Header
5. `</body>`
6. Footer
7. Embedded CSS

First a little bit about those subsections:

1. CSS: You can add CSS and SCSS here. Whatever you add is compiled automatically on save and added as a separate `.css` stylesheet to the `<head>` section if the theme is active.

2. `<head>`: You can add html here (including script tags). Anything you add here is inserted just before the close tag of the `<head>` close tag or `</head>`

3. Header: You can also add html here. Anything you add here is inserted at the top of the `<body>` tag before the Discourse Header

4. After Header: Same as above. You can add html here. However, anything you add here will be inserted below the Discourse header but above the rest of the page content

5. `</body>`: You can use html here. Anything you add is inserted at the very bottom of the `<body>` tag or just before the `</body>` tag

6. Footer: You can also use html here but it's is inserted just after the end of the page content or just below the `<div id="main-outlet">` close tag

7. Embedded CSS: You can add CSS and SCSS here as well, the difference is that whatever you add here is only applied to Discourse when it's embedded on another site. At the moment we support embedding [comments](https://meta.discourse.org/t/embedding-discourse-comments-via-javascript/31963) and [topic lists](https://meta.discourse.org/t/embedding-a-list-of-discourse-topics-in-another-site/125911).

This covers all the subsections of the common section of the theme editor. The desktop and mobile sections are exactly the same except that they will only target their respective devices and they don't have the Embedded CSS subsection.

Now that you understand what those sections are. Here are (most of) the files that a theme can contain:

```
common/common.scss
common/head_tag.html
common/header.html
common/after_header.html
common/body_tag.html
common/footer.html
common/embedded.scss

desktop/desktop.scss
desktop/head_tag.html
desktop/header.html
desktop/after_header.html
desktop/body_tag.html
desktop/footer.html

mobile/mobile.scss
mobile/head_tag.html
mobile/header.html
mobile/after_header.html
mobile/body_tag.html
mobile/footer.html
```

We're pretty serious about making sure file names are intuitive and so I hope that you can glance at the file name and figure out which section it relates to.

I said most of the files above because there are also other files a theme can / must include.

A theme can include assets like fonts and images and those go into an `assets` folder.

```
assets/background.jpg
assets/font.woff2
assets/icon.svg
```

A remote theme must include an `about.json` file in order for it to importable. The `about.json` file lives at the root of the theme and its contents look like this

```json
{
  "name": "Theme name",
  "about_url": "Theme about url",
  "license_url": "Theme license url",
  "assets": {
    "asset-variable": "assets/background.svg"
  },
  "color_schemes": {
    "color scheme name": {
      "primary": "000000",
      "secondary": "000000",
      "tertiary": "000000",
      "quaternary": "000000",
      "header_background": "000000",
      "header_primary": "000000",
      "highlight": "000000",
      "danger": "000000",
      "success": "000000",
      "love": "000000"
    }
  }
}
```

Themes can also include settings. The settings live in a `settings.yml` file that also live at the root of the theme directory. A theme settings file looks like this:

```yaml
whitelisted_fruits:
  default: apples|oranges
  type: list

favorite_fruit:
  default: orange
  type: enum
  choices:
    - apple
    - banana
```

This is just an example of theme settings. More details about them will follow.

So, here's an example of what a finished theme would look like:

```
about.json

assets/font.woff2
assets/background.jpg
assets/icon.svg

common/common.scss
common/head_tag.html
common/header.html
common/after_header.html
common/body_tag.html
common/footer.html
common/embedded.scss

desktop/desktop.scss
desktop/head_tag.html
desktop/header.html
desktop/after_header.html
desktop/body_tag.html
desktop/footer.html

mobile/mobile.scss
mobile/head_tag.html
mobile/header.html
mobile/after_header.html
mobile/body_tag.html
mobile/footer.html

settings.yml
```

pretty straightforward huh?

[quote="You,"]
Do I need to include all of these for every theme?
[/quote]

No! The only thing required is the `about.json` file for remote themes. Everything else is optional and should only be added if you need it.

#### Color schemes

You probably noticed the `color_schemes` section in the `about.json` file example above. Well, themes can introduce new color schemes!

[quote="You,"]
uhm... backup a bit. What are color schemes?
[/quote]

A Color scheme is a set of colors you choose that is used to automatically color all the elements in Discourse.

The interface looks like this

![6|619x500, 75%](/assets/beginners-guide-7.PNG)

and the colors are linked to the values you enter in the `about.json` file we discussed earlier. So

```json
"color_schemes": {
  "Foo bar": {
      "primary": "cccccc",
      "secondary": "111111",
      "tertiary": "009dd8",
      "quaternary": "9E9E9E",
      "header_background": "131418",
      "header_primary": "cccccc",
      "highlight": "9E9E9E",
      "danger": "96000e",
      "success": "1ca551",
      "love": "f50057"
    }
}
```

would create a new color scheme that looks like this

![7|617x500, 75%](/assets/beginners-guide-8.PNG)

Discourse then takes those colors (if that color scheme is active) does a bit of magic to them (which we'll cover later) and creates a few variations of those colors to style all the elements. This removes the need to write a gazillion lines of CSS just to change the theme colors across the board.

[quote="You,"]
Can you stop blabbering and let's create some themes already?!
[/quote]

Fine. :sob:
