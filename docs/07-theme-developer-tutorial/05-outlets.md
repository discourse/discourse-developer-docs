
#### Handlebars templates

Discourse is a modern web app. Traditional HTML by itself is not flexible enough due to the dynamic nature of content on Discourse. Just like SCSS makes working with CSS a lot easier, using Handlebars templates makes working with HTML less of a hassle.

If you're already familiar with Handlebars templates, then great! If not, don't worry about it and just think of Handlebars template as html on steroids.

##### Modifying Discourse templates

The easiest way to add html to any template is to find a plugin-outlet

[quote="You,"]
plugin what?
[/quote]

Well as it turns out most Discourse templates have things like this in them

```hbs
{{plugin-outlet name="topic-above-post-stream" args=(hash model=model)}}
```

This particular one comes from this template

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/templates/topic.hbs#L13

[quote="You,"]
Sure, whatever you say... but what can I do with this information?
[/quote]

Well, you can do something like this (I'll explain in a bit)

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/topic-above-post-stream/foobar"
>
  <div style="height: 200px; width: 200px;background: red"></div>
</script>
```

Now let's break this down a bit.

```html
<script type="text/x-handlebars" data-template-name="/connectors/topic-above-post-stream/foobar">
```

Look at the `data-template-name` attribute above. Notice anything familiar?

Well, when you need to add raw html or Handlebars expressions like this

```html
<div style="height: 200px; width: 200px;background: red"></div>
```

or this

```hbs
{{d-button class="btn-small cancel-edit" icon="times"}}
```

Discourse wants to know where you want to add those new elements. Plugin-outlets are just the way to do that. When I specify

```
data-template-name="/connectors/topic-above-post-stream/
```

in the Handlebars script tag, I am literally saying to Discourse, take the content of this script tag and inject it where the `topic-above-post-stream` plugin outlet is.

Naming is critical. so be sure to follow this

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/PLUGIN-OUTLET-NAME/UNIQUE-NAME"
></script>
```

Remember the example I gave you above?

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/topic-above-post-stream/foobar"
>
  <div style="height: 200px; width: 200px;background: red"></div>
</script>
```

Here's what that results in

![43|579x499, 75%](/assets/beginners-guide-47.PNG)

I've just added a random red box above the post stream on topic pages.

Let's try something else

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/category-title-before/foobar2"
>
  <div style="height: 25px; width: 25px;background: red"></div>
</script>
```

Try to read :point_up: this and see if you can figure out what I'm trying to do there...

Well, I added a tiny box before the title of every category.

here it is

![44|578x500, 75%](/assets/beginners-guide-48.PNG)

[quote="You,"]
Ok, but how do I find those plugin-outlets?
[/quote]

The easiest way to find plugin-outlets is to search for `PluginOutlet` in the Discourse repository

[https://github.com/discourse/discourse/search?q=PluginOutlet](https://github.com/discourse/discourse/search?q=PluginOutlet)

[quote="You,"]
Well, what if there's no plugin-outlet for the exact area I want to target?
[/quote]

You can submit a PR to add it! As long as the plugin-outlet has valid uses, we'll gladly add it to Discourse core.

[quote="You,"]
Well, all of this is about adding new elements, what if I want to remove an element or modify an existing element?
[/quote]

Well, this is where overriding Handlebars templates comes in

##### Overriding Discourse templates

Just like you can add new elements to templates, you can make changes to existing elements by overriding the template. A word of caution first. When you override a template, you're essentially using a new template in its place. While this is not inherently a bad thing. it does come with increased maintenance.

If you override a template (the `topic-list-item` for example) and subsequent changes are made to that template in Discourse core, you need to make sure you update your template to make sure everything works as expected. Think of it like maintaining a fork on Github.

[quote="You,"]
You're blabbering again...
[/quote]

Fine.. :expressionless:

Here's how to override a template

```html
<script type="text/x-handlebars" data-template-name="application"></script>
```

It's very similar to how you would modify a template with a plugin-outlet, the difference being the way you specify the `data-template-name` attribute.

When you specify a template to override, you need to know its name. In the Discourse repository, all templates live here (bookmark that page)

https://github.com/discourse/discourse/tree/master/app/assets/javascripts/discourse/app/templates

What you need to add as the `data-template-name` attribute is the template name minus the extension so

```
application.hbs
```

becomes

```
data-template-name="application"
```

While not obvious in the example above, this is actually the path of the file relative to the `templates` folder. `application.hbs` lives at the root of that folder, so nothing else needs to be added. However, if the template you want to target is inside a subfolder inside the `templates` folder, you need to specify that as well.

For example

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/templates/list/topic-list-item.hbr

is inside the `list` sub-folder in the `templates` folder, so to target it you need to write

```
data-template-name="list/topic-list-item.hbr"
```

So, we're going to take this:

```html
<script
  type="text/x-handlebars"
  data-template-name="list/topic-list-item.hbr"
></script>
```

Copy / paste the contents of the core template inside it first
https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/templates/list/topic-list-item.hbr

And then make whatever modifications we need there. For example, we can remove all poster avatars using something like this

[details="Expand snippet"]

```html
<script type="text/x-handlebars" data-template-name="list/topic-list-item.hbr">
  {{#if bulkSelectEnabled}}
    <td class="bulk-select">
      <input type="checkbox" class="bulk-select">
    </td>
  {{/if}}

  {{!--
    The `~` syntax strip spaces between the elements, making it produce
    `<a class=topic-post-badges>Some text</a><span class=topic-post-badges>`,
    with no space between them.
    This causes the topic-post-badge to be considered the same word as "text"
    at the end of the link, preventing it from line wrapping onto its own line.
  --}}
  <td class='main-link clearfix' colspan="{{titleColSpan}}">
    <span class='link-top-line'>
      {{~raw-plugin-outlet name="topic-list-before-status"}}
      {{~raw "topic-status" topic=topic}}
      {{~topic-link topic class="raw-link raw-topic-link"}}
      {{~#if topic.featured_link}}
        {{~topic-featured-link topic}}
      {{~/if}}
      {{~raw-plugin-outlet name="topic-list-after-title"}}
      {{~#if showTopicPostBadges}}
        {{~raw "topic-post-badges" unread=topic.unread newPosts=topic.displayNewPosts unseen=topic.unseen url=topic.lastUnreadUrl newDotText=newDotText}}
      {{~/if}}
    </span>

    {{discourse-tags topic mode="list" tagsForUser=tagsForUser}}
    {{#if expandPinned}}
      {{raw "list/topic-excerpt" topic=topic}}
    {{/if}}
    {{raw "list/action-list" topic=topic postNumbers=topic.liked_post_numbers className="likes" icon="heart"}}
  </td>

  {{#unless hideCategory}}
    {{#unless topic.isPinnedUncategorized}}
      {{raw "list/category-column" category=topic.category}}
    {{/unless}}
  {{/unless}}

  {{#if showPosters}}
    {{raw "list/posters-column" posters=topic.posters}}
  {{/if}}

  {{raw "list/posts-count-column" topic=topic}}

  {{#if showParticipants}}
    {{raw "list/posters-column" posters=topic.participants}}
  {{/if}}

  {{#if showLikes}}
  <td class="num likes">
    {{#if hasLikes}}
      <a href='{{topic.summaryUrl}}'>
        {{number topic.like_count}} {{d-icon "heart"}}</td>
      </a>
    {{/if}}
  {{/if}}

  {{#if showOpLikes}}
  <td class="num likes">
    {{#if hasOpLikes}}
      <a href='{{topic.summaryUrl}}'>
        {{number topic.op_like_count}} {{d-icon "heart"}}</td>
      </a>
    {{/if}}
  {{/if}}

  <td class="num views {{topic.viewsHeat}}">{{number topic.views numberKey="views_long"}}</td>

  {{raw "list/activity-column" topic=topic class="num" tagName="td"}}
</script>
```

[/details]

![45|579x500, 75%](/assets/beginners-guide-49.PNG)

Notice the red X. This part of the topic list comes from another template and so you'd need to find that and remove it as well for your design to be consistent... but the change we just made removed all avatar images from the topic list.

As well as removing elements from template, you can add new ones, just like with plugin-outlets and you can also move things around are reorder the template to your liking.

So, let's try to add a sidebar on desktops next to the latest topic list.

For this we're going to need to override the `components/topic-list` template. Or this

https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/templates/components/topic-list.hbs

Here's the html for the (basic) sidebar

```html
<div class="sidebar">
  <div class="card"></div>
  <div class="card"></div>
  <div class="card"></div>
</div>
```

and here's the SCSS

```scss
@import "common/foundation/variables";

.sidebar {
  background: $secondary-high;
  padding: 0 1em;
  .card {
    background: $secondary;
    height: 200px;
    width: 200px;
    padding: 0.5em;
    margin: 0.5em 0;
    box-sizing: border-box;
  }
}

table.topic-list {
  display: flex;
}
```

We're going to to add the HTML to the `components/topic-list` template and make other adjustments like so

[details="Expand snippet"]

```html
<script type="text/x-handlebars" data-template-name="components/topic-list">
  <div class="sidebar">
    <div class="card"></div>
    <div class="card"></div>
    <div class="card"></div>
  </div>
  {{plugin-outlet
    name="before-topic-list-body"
    args=(hash
      topics=topics
      selected=selected
      bulkSelectEnabled=bulkSelectEnabled
      lastVisitedTopic=lastVisitedTopic
      discoveryList=discoveryList
      hideCategory=hideCategory)
    tagName=""
    connectorTagName=""}}
  <tbody>
      {{raw "topic-list-header"
        canBulkSelect=canBulkSelect
        toggleInTitle=toggleInTitle
        hideCategory=hideCategory
        showPosters=showPosters
        showLikes=showLikes
        showOpLikes=showOpLikes
        showParticipants=showParticipants
        order=order
        ascending=ascending
        sortable=sortable
        listTitle=listTitle
        bulkSelectEnabled=bulkSelectEnabled}}
    {{#each filteredTopics as |topic|}}
      {{topic-list-item topic=topic
                        bulkSelectEnabled=bulkSelectEnabled
                        showTopicPostBadges=showTopicPostBadges
                        hideCategory=hideCategory
                        showPosters=showPosters
                        showParticipants=showParticipants
                        showLikes=showLikes
                        showOpLikes=showOpLikes
                        expandGloballyPinned=expandGloballyPinned
                        expandAllPinned=expandAllPinned
                        lastVisitedTopic=lastVisitedTopic
                        selected=selected
                        tagsForUser=tagsForUser}}
      {{raw "list/visited-line" lastVisitedTopic=lastVisitedTopic topic=topic}}
    {{/each}}
  </tbody>
</script>
```

[/details]

And here's our basic sidebar :tada:

![46|580x500, 75%](/assets/beginners-guide-50.PNG)

As an aside: mobile templates are inside the `mobile` subfolder in the `templates` folder. You would do the exact same thing if you want to modify any mobile template. Just be mindful of the path and the file name in the `data-template-name` attribute like we discussed before.

Template overrides also work with Discourse plugins. When overriding a plugin template, you always need to start your `data-template-name` with `javascripts`. After that, you'll want to add the path of the template relative to the `templates` folder just as you would with core template overrides.

The path to a plugin's template should look something like

> **plugin-name/assets/javascripts/discourse/templates/template-name.hbs**

And the override would look like this

```html
<script
  type="text/x-handlebars"
  data-template-name="javascripts/template-name"
></script>
```

Remember to include a subfolder before the template name if one exists!

##### Mounting widgets

Since modifying / overriding templates is something you might be doing quite a bit, let's do something a little bit more advanced. We're going to mount a widget and add it to a template.

[quote="You,"]
There you go again...backup a bit...what's a widget?
[/quote]

Since we're still in the Handlebars section of the guide, I'm not going to spend any time explaining what widgets are. I'll do that later, but for now I can give you examples of some Discourse widgets.

- The header is a widget
- The header logo is widget
- The hamburger menu is a widget
- The categories list inside the hamburger menu is widget

These are just examples of widgets. For now, you can think of them as blocks. All Discourse widgets live here

https://github.com/discourse/discourse/tree/master/app/assets/javascripts/discourse/app/widgets

[quote="You,"]
Why so fancy? :face_with_raised_eyebrow:
[/quote]

Widgets render faster and _more faster is more better_ :stuck_out_tongue:

[quote="You,"]
I still don't get the point of all of this.
[/quote]

I think the next example will help. First, let's pick a widget. I'll pick the `home-logo` widget, or this

![47|559x499, 80%](/assets/beginners-guide-51.PNG)

We're going to create a footer theme component and dynamically add the site logo to it. For this I chose to go with the plugin-outlet route instead of overriding a template because there's a plugin-outlet that works for our purposes

```hbs
{{plugin-outlet name="below-footer" args=(hash showFooter=showFooter)}}
```

which you can find [here](https://github.com/discourse/discourse/blob/main/app/assets/javascripts/discourse/app/templates/application.hbs#L28)

So, based on our previous discussion about plugin-outlets, we're going to need something like this

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/below-footer/fancy-footer"
></script>
```

Now, mounting a widget is pretty simple, all you need to know is the widget's name. That's it. We already know the name of the widget we want to use and it's `home-logo` and so here's what we need in order to mount it and add it to the template via the plugin-outlet

```hbs
{{mount-widget widget="home-logo"}}
```

Now we add a bit of HTML around it like so

```html
<script
  type="text/x-handlebars"
  data-template-name="/connectors/below-footer/fancy-footer"
>
  <div class="footer">
    <div class="wrap">
      <ul>
        <li><a href="/about">About</a></li>
        <li><a href="/Privacy">Privacy</a></li>
        <li><a href="/TOS">Terms of Service</a></li>
      </ul>
      <div class="footer-logo">
        {{mount-widget widget="home-logo"}}
      </div>
    </div>
  </div>
</script>
```

and a sprinkle of SCSS

```scss
@import "common/foundation/variables";

.footer {
  background: $primary-low;
  .wrap {
    display: flex;
  }
  ul {
    display: flex;
    flex: 1;
    margin: 0;
  }
  li {
    list-style: none;
    margin: 1em;
    font-size: $font-up-1;
    color: $secondary;
  }
  .footer-logo {
    display: flex;
    align-items: center;
    img {
      max-height: 40px;
    }
  }
}
```

and…

![48|576x499, 75%](/assets/beginners-guide-52.PNG)

:tada:

[quote="You,"]
So far, we've covered a lot of SCSS, HTML, and Handlebars, what about JS / jQuery? Are there any cool things I can do with those?
[/quote]

I'm glad you asked. Meet the pluginAPI :sunglasses:

