---
title: Call and Initialize JavaScript | Commerce Frontend Development 
description: Learn about the methods for calling and initializing JavaScript components in Adobe Commerce and Magento Open Source.
---

# Call and initialize JavaScript

This topic describes different ways to call and initialize JavaScript in Adobe Commerce and Magento Open Source:

-  Insert a JavaScript component in `.phtml` page templates.
-  Call Javascript components that require initialization in Javascript (`.js`) files.

We strongly recommend that you use the described approaches and do not add inline JavaScript.

## Insert a component in a PHTML template

Depending on your task, you can use declarative or imperative notation to insert a JS component into a PHTML template. Use declarative notation if a component requires initialization and imperative notation in other cases.

### Declarative notation

Using the declarative notation to insert a JS component prepares all the configuration on the backend and outputs it to page source using standard tools. Use declarative notation if your JavaScript component requires initialization.

You have two options for specifying declarative notation:

-  Using the `data-mage-init` attribute

   > This is used to target a specific HTML element. It is easier to implement and is commonly used for jQuery UI widgets. This method can only be implemented on the specified HTML tag. For example, `<nav data-mage-init='{"<component_name>": {...}}'></nav>`. This is preferred for its concise syntax, and direct access to the HTML element.

-  Using the `<script type="text/x-magento-init"> ... </script>` tag

   > This is used to target either a CSS selector or `*`. If the CSS selector matches multiple HTML elements, the script will run for each matched HTML element. For `*`, no HTML element is selected and the script will run once with the HTML DOM as its target. This method can be implemented from anywhere in the codebase to target any HTML element. This is preferred when direct access to the HTML element is restricted, or when there is no target HTML element.

Consider the example of adding a custom carousel JS:

1. Copy the `<carousel_name>.carousel.js` file to the `app/design/frontend/<package_name>/<theme_name>/web/js/<carousel_name>/` directory.
1. Add your RequireJS module at `app/design/frontend/<package_name>/<theme_name>/web/js/carousel.js`.

   ```javascript
    define(['jquery','<carousel_name>'], function($)
    {
        return function(config, element)
        {
            $(element).<carousel_name>(config);
        };
    });
   ```

1. Add the RequireJS config to the `app/design/frontend/<package_name>/<theme_name>/requirejs-config.js` file.

    ```javascript
    var config = {
        map: {
            '*': {
                    'carousel': 'js/carousel',
                    '<carousel_name>': 'js/<carousel_name>/<carousel_name>.carousel'
                }
            }
    };
    ```

You now have two options for specifying declarative notation:

-  Use the `data-mage-init` attribute to insert the carousel in a certain element:

    ```html
    <div data-mage-init='{"carousel":{"option": value}}'>
        <div class="item">Item 1</div>
        ...
        <div class="item">Item n</div>
    </div>
    ```

-  Use with `<script type="text/x-magento-init"/>`:

    ```html
    <div id="<carousel_name>" class="carousel">
        <div class="item">Item 1</div>
        ...
        <div class="item">Item n</div>
    </div>

    <script type="text/x-magento-init">
        {
            "#<carousel_name>": {
                "carousel": {"option": value}
            }
        }
    </script>
    ```

#### Use the `data-mage-init` attribute

Use the `data-mage-init` attribute to insert a JS component in a specified HTML element. The following example inserts a JS component in the `<nav/>` element:

```html
<nav data-mage-init='{"<component_name>": {...}}'></nav>
```

When the Javascript is inserted into the specified element, the script is called only for this particular element. It is not automatically called for other elements of this type on the page.

#### How `data-mage-init` is processed

On DOM ready, the `data-mage-init` attribute is parsed to extract component names and configuration to be applied to the element. Depending on the type of the inserted JS component, processing is performed as follows:

-  If an object is returned, the initializer tries to find the `<component_name>` key. If the corresponding value is a function, the initializer passes the `config` and `element` values to this function. For example:

  ```javascript
  return {
    '<component_name>': function(config, element) { ... }
  };
  ```

Where `<component_name>` is a native JS component, for example: `menu`, `collapsible`, `tooltip` ...

```html
<nav data-mage-init='{"tooltip": {"content": "<?= /* @noEscape */ $content ?>"}}'></nav>
```

Or a custom JS component, implemented with a component path: `Vendor_Module/js/component`, or as an alias declared in `requirejs-config.js`.

```html
<nav data-mage-init='{"Vendor_Module/js/component": {"status":"<?= /* @noEscape */ $block->getStatus(); ?>"}}'></nav>
```

Read more about [locate JS components](debug.md).

-  If a function is returned, the initializer passes the `config` and `element` values to this function. For example:

  ```javascript
  return function(config, element) { ... };
  ```

-  If neither a function nor an object with the `"<component_name>"` key are returned, then the initializer tries to search for `"<component_name>"` in the jQuery prototype. If found, the initializer applies it as `$(element).<component_name>;(config)`. For example:

  ```javascript
  $.fn.<component_name> = function() { ... };
  return;
  ```

-  If none of the previous cases is true, the component is executed with no further processing. Such a component does not require either `config` or `element`. The recommended way to declare such components is [using the `<script>` tag](#use-the-script-typetextx-magento-init-tag).

#### Use the `<script type="text/x-magento-init">` tag

To call a JS component on an HTML element without direct access to the element or with no relation to a certain element, use the `<script type="text/x-magento-init">` tag and attribute syntax shown in the following example.

```html
<script type="text/x-magento-init">
{
    // components initialized on the element defined by selector
    "<element_selector>": {
        "<js_component1>": ...,
        "<js_component2>": ...
    },
    // components initialized without binding to an element
    "*": {
        "<js_component3>": ...
    }
}
</script>
```

Where:

-  `<element_selector>` is a [selector] (in terms of querySelectorAll) for the element on which the following JS components are called.
-  `<js_component1>` and `<js_component2>` are the JS components being initialized on the element with the selector specified as `<element_selector>`.
-  `<js_component3>` is the JS component called with no binding to an element.

The following example provides a working code sample of a widget call using `<script>`. Here the `accordion` and `navigation` widgets are added to the element with the `#main-container` selector, and the `pageCache` script is inserted with no binding to any element.

```html
<script type="text/x-magento-init">
{
    "#main-container": {
        "navigation": <?php echo $block->getNavigationConfig(); ?>,
        "accordion": <?php echo $block->getNavigationAccordionConfig(); ?>
    },
    "*": {
        "pageCache": <?php echo $block->getPageCacheConfig(); ?>
    }
}
</script>
```

### Imperative notation

Use imperative notation in the PHTML template to include raw JavaScript code on the pages to execute specified business logic. This method uses the `<script>` tag without the `type="text/x-magento-init"` attribute as shown in the following example:

```html
<script>
require([
    'jquery',
    'accordion'  // the alias for "mage/accordion"
], function ($) {
    $(function () { // to ensure that code evaluates on page load
        $('[data-role=example]')  // we expect that page contains the <tag data-role="example">..</tag> markup
            .accordion({ // now we can use "accordion" as jQuery plugin
                header:  '[data-role=header]',
                content: '[data-role=content]',
                trigger: '[data-role=trigger]',
                ajaxUrlElement: "a"
            });
    });
});
</script>
```

<InlineAlert variant="success" slots="text" />

For better control when scripts are executed, use a declarative syntax rather than an imperative syntax. When using imperative syntax, the ability to leverage existing JS classes is lost and can block the rendering of the page.

## Calling components that require initialization

To call a widget with JS code, use a notation similar to the ([accordion](jquery-widgets/accordion.md) widget. It is initialized on the `[data-role=example]` element as below):

```javascript
$('[data-role=example]').accordion();
```

To initialize a widget with options, use notation similar to the following:

```javascript
$(function () { // to ensure that code evaluates on page load
    $('[data-role=example]')  // we expect that page contains markup <tag data-role="example">..</tag>
        .accordion({ // now we can use "accordion" as jQuery plugin
            header:  '[data-role=header]',
            content: '[data-role=content]',
            trigger: '[data-role=trigger]',
            ajaxUrlElement: 'a'
        });
});
```

In a similar way, you can initialize any JS component that a returns callback function accepting a `config` object and `element` (a DOM node).

For example:

```javascript
define ([
    'jquery',
    'mage/gallery/gallery'
], function ($, Gallery) {

    $(function () { // to ensure that code evaluates on page load
        $('[data-role=example]')  // we expect that page contains markup <tag data-role="example">..</tag>
            .each(function (index, element) {
                Gallery({
                    options:  {},
                    data: [{
                        img: 'https://c2.staticflickr.com/8/7077/27935031965_facd03b4cb_b_d.jpg'
                    }],
                    fullscreen: {}
                }, element);  // 'element' is single DOM node.
            });
    });
});
```

## Execute data-mage-init and x-magento-init in dynamic content

To trigger `.trigger('contentUpdated')` on the element when dynamic content is injected, use:

```javascript
$.ajax({
    url: 'https://www.example.com',
    method: 'POST',
    data: {
        id: '1'
    },
    success: function (data) {
        $('.example-element').html(data)
                             .trigger('contentUpdated')
    }
});
```

[selector]: https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector
