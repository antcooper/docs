# Control Panel Templates

The control panel is built using Twig templates, so extending it with new pages should feel familiar if you’ve worked with Twig on the front end.

Plugins are automatically given a [template root](template-roots.md) that exposes files in their `templates/` folder. You can `include` or render these templates by prefixing their path with the plugin’s handle.

For example, if a plugin’s handle is `foo` and it had a template at `templates/bar.twig`, you could access it by navigating to `/admin/foo/bar` in your browser (just like the front-end), or from another Twig context (or [controller action](controllers.md#rendering-templates)) as `foo/bar` (or `foo/bar.twig`).

Modules can have templates too, but they will need to manually define a [template root](template-roots.md) before they are accessible.

::: tip
Looking to support full-page interfaces _and_ slideouts? Check out the new [control panel screens](./controllers.md#control-panel-screens) API.
:::

## Page Templates

To add a new full page to the control panel, create a template that extends the [_layouts/cp.twig](repo:craftcms/cms/blob/5.x/src/templates/_layouts/cp.twig) template.

At a minimum, your template should set a `title` variable and define a `content` block:

```twig
{% extends "_layouts/cp.twig" %}
{% set title = "Page Title"|t('plugin-handle') %}

{% block content %}
  <p>Page content goes here</p>
{% endblock %}
```

### Supported Variables

The following variables are supported by the `_layouts/cp.twig` template:

Variable | Description
-------- | -----------
`title` | The page title.
`bodyClass` | An array of class names that should be added to the `<body>` tag.
`fullPageForm` | Whether the entire page should be wrapped in one big `<form>` element (see [Form Pages](#form-pages)).
`crumbs` | An array of breadcrumbs (see [Adding Breadcrumbs](#adding-breadcrumbs)).
`tabs` | An array of tabs (see [Adding Tabs](#adding-tabs)).
`selectedTab` | The ID of the selected tab.
`showHeader` | Whether the page header should be shown (`true` by default).
`mainAttributes` | A hash of HTML attributes that should be added to the `<main>` tag. 

### Available Blocks

The `_layouts/cp` template defines the following blocks, which your template can extend:

Block | Outputs
----- | -------
`actionButton` | The primary Save button.
`content` | The page’s main content.
`contextMenu` | An optional context menus beside the page title (e.g. the revision menu on Edit Entry pages).
`details` | The page’s right sidebar content.
`footer` | The page’s footer content.
`header` | The page’s header content, including the page title and other header elements.
`pageTitle` | The page title (rendered within the `header` block).
`sidebar` | The page’s left sidebar content.
`toolbar` | The page’s toolbar content.

### Adding Breadcrumbs

To add breadcrumbs to your page, define a `crumbs` variable, set to an array of the breadcrumbs.

Each breadcrumb should be represented as a hash with the following keys:

Key | Description
--- | -----------
`label` | The breadcrumb label.
`url` | The URL that the breadcrumb should link to.

For example, the following `crumbs` array defines two breadcrumbs:

```twig
{% set crumbs = [
  {
    label: 'Plugin Name'|t('plugin-handle'),
    url: url('plugin-handle'),
  },
  {
    label: 'Products'|t('plugin-handle'),
    url: url('plugin-handle/products'),
  },
] %}
```

### Adding Tabs

To add tabs to your page, define a `tabs` variable, set to a hash of the tabs, indexed by tab IDs.

Each tab should be represented as a nested hash with the following keys:

Key | Description
--- | -----------
`label` | The tab label.
`url` | The URL or anchor that the tab should link to.
`class` | A class name that should be added to the tab (in addition to `tab`).

For example, the following `tabs` hash defines two tabs, which will toggle the `hidden` class of `<div>` elements whose IDs match the anchors:

```twig
{% set tabs = {
  content: {
    label: 'Content'|t('plugin-handle'),
    url: '#content',
  },
  settings: {
    label: 'Settings'|t('plugin-handle'),
    url: '#settings',
  },
} %}
```

The first tab will be selected by default. You can force a different tab to be selected by default by setting a `selectedTab` variable to the ID of the desired tab:

```twig
{% set selectedTab = 'settings' %}
```

### Form Pages

If the purpose of your template is to present a form to the user, start by setting the `fullPageForm` variable to `true` at the top:

```twig
{% set fullPageForm = true %}
```

When you do that, everything inside the page’s `<main>` element will be wrapped in a `<form>` element, and a Save button and CSRF input will be added to the page automatically for you. It will be up to you to define the `action` input, however.

```twig
{% block content %}
  {{ actionInput('plugin-handle/controller/action') }}
  <!-- ... -->
{% endblock %}
```

Your template can also define the following variables, to customize the form behavior:

Variable | Description
-------- | -----------
`formActions` | An array of available Save menu actions for the form (see [Alternate Form Actions](#alternate-form-actions)).
`mainFormAttributes` | A hash of HTML attributes that should be added to the `<form>` tag.
`retainScrollOnSaveShortcut` | Whether the page’s scroll position should be retained on the subsequent page load when the <kbd>Ctrl</kbd>/<kbd>Command</kbd> + <kbd>S</kbd> keyboard shortcut is used.
`saveShortcutRedirect` | The URL that the page should be redirected to when the <kbd>Ctrl</kbd>/<kbd>Command</kbd> + <kbd>S</kbd> keyboard shortcut is used.
`saveShortcut` | Whether the page should support a <kbd>Ctrl</kbd>/<kbd>Command</kbd> + <kbd>S</kbd> keyboard shortcut (`true` by default).

#### Form Inputs

Craft’s [_includes/forms.twig](repo:craftcms/cms/blob/5.x/src/templates/_includes/forms.twig) template defines several macros that can be used to display your form elements.

Most input types have two macros: one for outputting _just_ the input; and another for outputting the input as a “field”, complete with a label, author instructions, etc.

For example, if you just want to output a date input, but nothing else, you could use the `date` macro:

```twig
{% import '_includes/forms.twig' as forms %}

{{ forms.date({
  id: 'start-date',
  name: 'startDate',
  value: event.startDate,
}) }}
```

However if you want to wrap the input with a label, author instructions, a “Required” indicator, and any validation errors, you could use the `dateField` macro instead:

```twig
{% import '_includes/forms.twig' as forms %}

{{ forms.dateField({
  label: 'Start Date'|t('plugin-handle'),
  instructions: 'The start date of the event.'|t('plugin-handle'),
  id: 'start-date',
  name: 'startDate',
  value: event.startDate,
  required: true,
  errors: event.getErrors('startDate'),
}) }}
```

#### Alternate Form Actions

If you want your form to have a menu button beside the Save button with alternate form actions, define a `formActions` variable, set to an array of the actions.

Each action should be represented as a hash with the following keys:

Key | Description
--- | -----------
`action` | The controller action path that the form should submit to, if this action is selected.
`confirm` | A confirmation message that should be presented to the user when the action is selected, before the form is submitted.
`data` | A hash of any data that should be passed to the form’s `submit` event.
`destructive` | Whether the action should be considered destructive, which will cause it to be listed in red text at the bottom of the menu, after a horizontal rule.
`label` | The action’s menu option label.
`params` | A hash of additional form parameters that should be included in the submission if the action is selected.
`redirect` | The URL that the controller action should redirect to if successful.
`retainScroll` | Whether the page’s scroll position should be retained on the subsequent page load.
`shift` | Whether the action’s keyboard shortcut should include the <kbd>Shift</kbd> key.
`shortcut` | Whether the action should be triggered by a keyboard shortcut.

For example, the following `formActions` array defines three alternative form actions:

```twig
{% set formActions = [
  {
    label: 'Save and continue editing'|t('plugin-handle'),
    redirect: 'plugin-handle/edit/{id}',
    retainScroll: true,
    shortcut: true,
  },
  {
    label: 'Save and add another'|t('plugin-handle'),
    redirect: 'plugin-handle/new',
    shortcut: true,
    shift: true,
  },
  {
    label: 'Delete'|t('plugin-handle'),
    confirm: 'Are you sure you want to delete this?'|t('plugin-handle'),
    redirect: 'plugin-handle',
    destructive: true,
  },
] %}
```

Note that when `formActions` is defined, the `saveShortcut`, `saveShortcutRedirect`, and `retainScrollOnSaveShortcut` variables will be ignored, as it will be up to the individual form actions to define that behavior.
