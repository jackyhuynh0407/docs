---
subtitle: Configurable building blocks that can be attached to any page, partial or layout.
---
# Components

Components are key features of October CMS. Each component implements some functionality that extends your website. Components can output HTML markup on a page, but it is not necessary - other important features of components are [handling AJAX requests](../ajax/introduction.md), handling form postbacks and handling the page execution cycle, that allows to inject variables to pages or implement the website security.

This article describes the components basics and doesn't explain how to use [components with AJAX](../ajax/handlers.md) or [developing components](../../extend/cms-components.md) as part of plugins.

## Introduction

If you use the back-end user interface you can add components to your pages, partials and layouts by clicking the component in the Components panel. If you use a text editor you can attach a component to a page or layout by adding its name to the configuration section of the template file. The next example demonstrates how to add a demo To-Do component to a page.

```ini
title = "Components demonstration"
url = "/components"

[demoTodo]
maxItems = 20
==
...
```

This initializes the component with the properties that are defined in the component section. Many components have properties, but it is not a requirement. Some properties are required, and some properties have default values. If you are not sure what properties are supported by a component, refer to the documentation provided by the developer, or use the Inspector in the October back-end. The Inspector opens when you click a component in the page or layout component panel.

::: aside
If two components with the same name are assigned to a page and layout together, the page component will take priority.
:::

When you refer a component, it automatically creates a page variable that matches the component name (`demoTodo` in the previous example). Components that provide HTML markup can be rendered on a page with the `{% component %}` tag, like the following.

```twig
{% component 'demoTodo' %}
```

::: warning
Using components inside partials has limited functionality, this is described in more detail in the [Dynamic Partials section](./partials.md) of the documentation.
:::

## Components Aliases

If there are two plugins that register components with the same name, you can attach a component by using its fully qualified class name and assigning it an *alias*.

```ini
[October\Demo\Components\Todo demoTodoAlias]
maxItems = 20
```

The first parameter in the section is the class name, the second is the component alias name that will be used when attached to the page. If you specified a component alias you should use it everywhere in the page code when you refer to the component. Note that the next example refers to the component alias:

```twig
{% component 'demoTodoAlias' %}
```

The aliases also allow you to define multiple components of the same class on a same page by using the short name first and an alias second. This lets you to use multiple instances of a same component on a page.

```ini
[demoTodo todoA]
maxItems = 10
[demoTodo todoB]
maxItems = 20
```

## Using External Property Values

By default property values are initialized in the Configuration section where the component is defined, and the property values are static, like this:

```ini
[demoTodo]
maxItems = 20
==
...
```

::: v-pre
However there is a way to initialize properties with values loaded from external parameters - URL parameters or [partial](partials.md) parameters (for components defined in partials). Use the `{{ paramName }}` syntax for values that should be loaded from partial variables:
:::

```ini
[demoTodo]
maxItems = {{ maxItems }}
==
...
```

Assuming that in the example above the component **demoTodo** is defined in a partial, it will be initialized with a value loaded from the **maxItems** partial variable:

```twig
{% partial 'my-todo-partial' maxItems='10' %}
```

You may use dot notation to retrieve a deeply nested value from an external parameter:

```ini
[demoTodo]
maxItems = {{ data.maxItems }}
==
...
```

::: v-pre
To load a property value from the URL parameter, use the `{{ :paramName }}` syntax, where the name starts with a colon (`:`), for example.
:::

```ini
[demoTodo]
maxItems = {{ :maxItems }}
==
...
```

The page that the component belongs to should have a corresponding [URL parameter](pages.md) defined.

```ini
url = "/todo/:maxItems"
```

In the October back-end you can use the Inspector tool for assigning external values to component properties. In the Inspector you don't need to use the curly brackets to enter the parameter name. Each field in the Inspector has an icon on the right side, which opens the external parameter name editor. Enter the parameter name as `paramName` for partial variables or `:paramName` for URL parameters.

## Passing Variables to Components

Components can be designed to use variables at the time they are rendered, similar to [Partial variables](./partials.md), they can be specified after the component name in the `{% component %}` tag. The specified variables will explicitly override the value of the [component properties](../../extend/cms-components.md), including external property values.

In this example, the **maxItems** property of the component will be set to *7* at the time the component is rendered:

```twig
{% component 'demoTodoAlias' maxItems='7' %}
```

## Customizing Default Markup

The markup provided by components is generally intended as a usage example for the Component. In some cases you may wish to modify the appearance and output of a component. Moving the default markup to a theme partial is suitable to completely overhaul a component. Overriding the component partials is useful for cherry picking areas to customize.

### Moving Default Markup to a Partial

Each component can have an entry point partial called **default.htm** that is rendered when the `{% component %}` tag is called, in the following example we will assume the component is called **blogPost**.

```ini
url = "blog/post"

[blogPost]
==
{% component "blogPost" %}
```

The output will be rendered from the plugin directory **components/blogpost/default.htm**. You can copy all the markup from this file and paste it directly in the page or to a new partial, called **blog-post.htm** for example.

```twig
<h1>{{ __SELF__.post.title }}</h1>
<p>{{ __SELF__.post.description }}</p>
```

Inside the markup you may notice references to a variable called `__SELF__`, this refers to the component object and should be replaced with the component alias used on the page, in this example it is `blogPost`.

```twig
<h1>{{ blogPost.post.title }}</h1>
<p>{{ blogPost.post.description }}</p>
```

This is the only change needed to allow the default component markup to work anywhere inside the theme. Now the component markup can be customized and rendered using the theme partial.

```twig
{% partial 'blog-post.htm' %}
```

This process can be repeated for all other partials found in the component partial directory.

### Overriding Component Partials

All component partials can be overridden using the theme partials. If a component called **channel** uses the **title.htm** partial.

```ini
url = "mypage"

[channel]
==
{% component "channel" %}
```

We can override the partial by creating a file in our theme called **partials/channel/title.htm**.

The file path segments are broken down like this:

Segment | Description
------------- | -------------
**partials** | the theme partials directory
**channel** | the component alias (a partial subdirectory)
**title.htm** | the component partial to override

The partial subdirectory name can be customized to anything by simply assigning the component an alias of the same name. For example, by assigning the **channel** component with a different alias **foobar** the override directory is also changed:

```ini
[channel foobar]
==
{% component "foobar" %}
```

Now we can override the **title.htm** partial by creating a file in our theme called **partials/foobar/title.htm**.

## The "View Bag" Component

::: aside
The viewBag component is hidden in the backend panel and is only available for file-based editing. It can also be used by other plugins to store data.
:::

There is a special component included in October CMS called `viewBag` that can be used on any page or layout. It allows ad hoc properties to be defined and accessed inside the markup area easily as variables. A good usage example is defining an active menu item inside a page.

```ini
title = "About"
url = "/about.html"
layout = "default"

[viewBag]
activeMenu = "about"
==

<p>Page content...</p>
```

Any property defined for the component is then made available inside the page, layout, or partial markup using the `viewBag` variable. For example, in this layout the **active** class is added to the list item if the `viewBag.activeMenu` value is set to **about**.

```twig
description = "Default layout"
==
[...]

<!-- Main navigation -->
<ul>
    <li class="{{ viewBag.activeMenu == 'about' ? 'active' }}">About</li>
    [...]
</ul>
```

### AJAX Handlers and Partials

Components may introduce [AJAX handlers](../ajax/introduction.md) and [partials](./partials.md) to the a theme's lifecycle, using a prefix of the component name and two `::` symbols. For example, all the AJAX handlers defined by components are available globally.

```html
data-request="onMyComponentHandler"
```

However, if there is a conflict in naming, the fully qualified name can be used.

```html
data-request="componentName::onMyComponentHandler"
```

Partials rendered from outside the component must use their fully qualified name.

```twig
{% partial 'componentName::component-partial' %}
```

Read more on [component development](../../extend/cms-components.md) to learn about component partials.

#### See Also

::: also
* [CMS Component Development](../../extend/cms-components.md)
:::
