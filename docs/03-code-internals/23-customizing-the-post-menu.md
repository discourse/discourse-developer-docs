## Customizing the post menu

The post menu includes the actions a user can take on a post, e.g., like, flag, edit, reply, etc.

You may need to customize this area to suit your needs. Discourse provides the `post-menu-buttons` transformer that allows adding new buttons, removing, replacing, or rearranging them.

### This looks complex. Where do I start?

First and foremost, not every customization in the post menu requires using the API.

As an admin, if you only need to change which core buttons are displayed, in which order, and/or which ones are readily available and which ones require clicking first on the button. You can change the values of two site settings:

* `post menu`: Configures the visibility and order of the default post menu items.
* `post menu hidden items`: The menu items to hide by default in the post menu unless the `Show More` button is clicked.

These two settings are enough for most of the basic customizations an admin may require when setting up a community.

As you may have noticed above, there is a caveat when using these two settings: they only apply to core buttons.

**What is a core button?**

Core buttons are the ones that ship by default with every Discourse installation to cover the basic features an user may need when interacting with a post. They are not added by plugins or themes.
  
Currently these are the core buttons: `read`, `like`, `copyLink`, `share`, `flag`, `edit`, `bookmark`, `delete`, `admin`, `reply`

Notice that not every one of them is visible by default, `share` for example, is not added to the post menu unless your settings are changed. 
 
Some of them, also, are only displayed when certain criteria is met.

If you're a plugin or theme author and requires more advanced capabilities, like for example customizing buttons provided by plugins, to fine tune the post menu to your needs. Then you need to use our API.

### The customization API

To customize the buttons, you need to register a value transformer like below:

```js

withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context }) => {
      // your customizations ...
    }
  );
});
```

As with any value transformer, your callback function will receive an object containing two parameters:

* `value` is a DAG (direct acyclic graph) API object which provides the following methods to customize the buttons:
    * `add`: adds a new button
    * `replace`: replaces an existing button with a new component
    * `delete`: deletes an existing button
    * `reposition`: repositions an existing button
    * `has`: checks if a button exists
    * `entries`: return the list of registered buttons

* `context`: provides context data for the customizations:
    * `post`: the post for which the post menu is being created;
    * `state`: other context information about the current state of the post/post menu
    * ` API that allows hiding or displaying an existing button label
    * `buttonKeys`: object containing the keys of the core buttons
    * `firstButtonKey`: key of the first registered button
    * `lastHiddenButtonKey`: key of the last hidden button
    * `lastItemBeforeMoreItemsButtonKey`: key of the last item before the `Show More` button
    * `secondLastHiddenButtonKey`: key of the second last hidden button

> One basic concept that you need to be aware of, to properly use our API is that each button is identified by a key string. You will need to refer to this key to use the DAG methods.

#### Adding a button

To add a button you need to use the `add(key, Component, position)` method provided by the DAG object.

Your need at least to provide a `key` for your button and the `Component` which defines your button. 

It is a good practice to prefix your plugin/theme key with a unique identifier to avoid collisions. If your plugin is called `my-awesome-plugin` then your keys should be `my-awesome-plugin-...`.

`Component` is an Ember component that should follow a few patterns for better results. See more [below](link-to-component-anatomy).

The `position` argument is optional, it indicates where your new button should be positioned. 

Please refer to the [`reposition` section]() for more information about the expected values for this argument and how the positioning works, but in a nutshell, `position` is an object that can contain two keys:

* `before`: array containing the keys of the buttons that should be positioned **before** you new button
* `after`: array containing the keys of the buttons that should be positioned **after** you new button

example:

```js
import TestButton from "../components/test-button";

withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { lastItemBeforeMoreItemsButtonKey } }) => {
      dag.add(
        "my-awesome-plugin-test-button", // your new button key
        TestButton, // your new button component
        {
          // tells the DAG to place your component after the last item before 
          // the Show More button
          after: lastItemBeforeMoreItemsButtonKey,  
        }
      );
    }
  );
});
```

#### Replacing a button

You may want to simply replace an existing button with another customized component. This can be done for example to change the conditions in which the button is displayed, customize the action performed, etc.

To replace a button you need to use the `replace(key, Component, position)` method provided by the DAG object.

Similarly to the `add` method, you need to provide a `key`, which in this case corresponds to the the key of the button which will be replaced, and the `Component` which defines your new button. See [below](link-to-component-anatomy) for more information on how to define the component.

 `position` can also, optionally, be provided to rearrange the position in which the button is displayed.

example:

```js
import NewEditButton from "../components/new-edit-button";

withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { buttonKeys } }) => {
      dag.replace(
        buttonKeys.EDIT, // the key to the edit button
        NewEditButton, // your new edit button component
        {
          // tells the DAG to place the edit button after the Show More and 
          // Reply buttons
          after: [buttonKeys.SHOW_MORE, buttonKeys.REPLY],
        }
      );
    }
  );
});
```

#### Anatomy of a button component

#### Deleting a button

To delete a button from the post menu you need to use the `delete(key)` method provided by the DAG object.

`key` on this case refers to the key of the button which will be removed.

```js
withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { buttonKeys } }) => {
      dag.delete(buttonKeys.LIKE); // deletes the Like button
    }
  );
});
```

#### Repositioning a button

Sometimes you may just want to change the order in which the buttons are displayed.

"But above you said we can use the `post menu` setting to order the buttons?", you may ask. Well, this is true, but as you may recall there was the caveat that this only applies to core buttons.

You may need for example to alter the placement of a plugin button in a theme or, in a more advanced use case, you may want to reorder a button based in context information from the post for example.

For cases where you need to fine tune the placement of a button, you need to use the `reposition(key, position)` method from the DAG object.

As arguments for the method, you need to provide a `key` which in this case corresponds to the key of the button that will be repositioned, and a new `position`.

The `position` argument expects an object which can contain two keys:

```
   { 
     before, // key or array of keys of button to be placed before
     after   // key or array of keys of button to be placed after
   }
```

Both `before` or `after` will accept either the key of a button or an array containing multiple keys of buttons. You need to provide at least one of them in you `position` object.

**Examples:**

```js
withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { buttonKeys } }) => {
      dag.reposition(buttonKeys.EDIT, { // repositions the Edit button
        before: buttonKeys.REPLY, // it should be placed before the Reply button
        after: buttonKeys.SHOW_MORE, // and after the Show More button
      });
    }
  );
});
```

```js
withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { buttonKeys } }) => {
      dag.reposition(buttonKeys.LIKE, { // repositions the Like button
         // it should be placed before the Copy and Share buttons
        before: [buttonKeys.COPY_LINK, buttonKeys.SHARE],
      });
    }
  );
});
```

Something very important you need to be aware of how the DAG algorith works, is that it will try to respect all the positions that were requested, but it won't necessarily do it just before or after the requested places.

For example, suppose you have these buttons in the following order: `Like, Delete, Edit, Copy`.

Now suppose you try to use the API to reposition `Like` to be placed after `Delete`.

```js
withPluginApi("1.34.0", (api) => {
  api.registerValueTransformer(
    "post-menu-buttons",
    ({ value: dag, context: { buttonKeys } }) => {
      dag.reposition(buttonKeys.LIKE, {
        after: buttonKeys.DELETE,
      });
    }
  );
});
```

The following results are valid:

- `Delete, Like, Edit, Copy`
- `Delete, Edit, Like, Copy`
- `Delete, Edit, Copy, Like`

Repositioning a button to be `after` another button or set of buttons don't mean necessarily it will be placed immediately after. The same applies to `before`.

This is a undesired effect of how the DAG algorithm works. We'll try to improve this behavior in the future, but for now this is something you need to consider when positioning your buttons.

To mitigate this and achieve your desired results you can try a few strategies:

- switch between `before` and `after`
  If placing your button `after` another doesn't yield the desired result, try placing it `before` another one.
  In the example, instead of placing `Like` after `Delete` you could try placing it before `Edit`

  ```js
  withPluginApi("1.34.0", (api) => {
    api.registerValueTransformer(
      "post-menu-buttons",
      ({ value: dag, context: { buttonKeys } }) => {
        dag.reposition(buttonKeys.LIKE, {
          before: buttonKeys.EDIT,
        });
      }
    );
  });
  ```  

- provide both `before` and `after`
  Most of the times you can get better results providing both `before` and `after` references.
  In the example, instead of just placing `Like` after `Delete` you could try also placing it before `Edit`

  ```js
  withPluginApi("1.34.0", (api) => {
    api.registerValueTransformer(
      "post-menu-buttons",
      ({ value: dag, context: { buttonKeys } }) => {
        dag.reposition(buttonKeys.LIKE, {
          before: buttonKeys.EDIT,
          after: buttonKeys.DELETE,
        });
      }
    );
  });
  ```  

- provide more than one key as reference
  Both `before` and `after` can be provided as arrays containing multiple keys to be used as reference.
  In the example, instead of just placing `Like` after `Delete` you could try also placing it before `Edit` and `Copy`

  ```js
  withPluginApi("1.34.0", (api) => {
    api.registerValueTransformer(
      "post-menu-buttons",
      ({ value: dag, context: { buttonKeys } }) => {
        dag.reposition(buttonKeys.LIKE, {
          before: [buttonKeys.EDIT, buttonKeys.COPY],
          after: buttonKeys.DELETE,
        });
      }
    );
  });
  ```  

  > **A word of caution**
  > Don't overuse references. Try keeping them as few as possible, if you provide too much references for a button this may force the DAG to switch other buttons to meet the specified criteria.