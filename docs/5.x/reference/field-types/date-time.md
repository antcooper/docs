---
related:
  - uri: time.md
    label: Time-only fields
---

# Date Fields

Date fields give you a date picker, and optionally a time picker as well.

<!-- more -->

You can also pick minimum and maximum dates that should be allowed, and if you’re showing the time, you can choose what the minute increment should be.

## Settings

Date fields have the following settings:

- **Show date** or **Show date and time**\
    If **Show date and time** is selected, the following settings will be visible:
    - **Minute Increment** – number of minutes that timepicker suggestions should be incremented by. (Authors can manually enter a specific time.)
    - **Show Time Zone** – whether authors should be able to choose the time zone, rather than the system’s.
- **Min Date** – the earliest date that should be allowed.
- **Max Date** – the latest date that should be allowed.

## Development

### Querying Elements with Date Fields

When [querying for elements](element-queries.md) that have a Date field, you can filter the results based on the Date field data using a query param named after your field’s handle.

Possible values include:

| Value | Fetches elements…
| - | -
| `':empty:'` | that don’t have a selected date.
| `':notempty:'` | that have a selected date.
| `'>= 2018-04-01'` | that have a date selected on or after 2018-04-01.
| `'< 2018-05-01'` | that have a date selected before 2018-05-01
| `['and', '>= 2018-04-01', '< 2018-05-01']` | that have a date selected between 2018-04-01 and 2018-05-01.
| `['or', '< 2018-04-01', '> 2018-05-01']` | that have a date selected before 2018-04-01 or after 2018-05-01.

Date values are always assumed to be in the system timezone, which is set in **Settings** &rarr; **General**.

::: code
```twig
{# Fetch entries with a selected date in the next month #}
{% set start = now|atom %}
{% set end = now|date_modify('+1 month')|atom %}

{% set entries = craft.entries()
  .myFieldHandle(['and', ">= #{start}", "< #{end}"])
  .all() %}
```
```php
// Fetch entries with a selected date in the next month
$start = (new \DateTime())->format(\DateTime::ATOM);
$end = (new \DateTime('+1 month'))->format(\DateTime::ATOM);

$entries = \craft\elements\Entry::find()
    ->myFieldHandle(['and', ">= ${start}", "< ${end}"])
    ->all();
```
:::

::: tip
The [atom](../twig/filters.md#atom) filter converts a date to an ISO-8601 timestamp.
:::

Craft 3.7 added support for using `now` in date comparison strings:

::: code
```twig
{# Fetch entries with a selected date in the past #}
{% set pastEntries = craft.entries()
  .myFieldHandle('< now')
  .all() %}
{# Fetch entries with a selected date now onward #}
{% set futureEntries = craft.entries()
  .myFieldHandle('>= now')
  .all() %}
```
```php
// Fetch entries with a selected date in the past
$pastEntries = \craft\elements\Entry::find()
    ->myFieldHandle('< now')
    ->all();
// Fetch entries with a selected date now onward
$futureEntries = \craft\elements\Entry::find()
    ->myFieldHandle('>= now')
    ->all();
```
:::

::: tip
Me mindful of [template caching](../twig/tags.md#cache) when comparing against the current time!
:::

### Working with Date Field Data

If you have an element with a Date field in your template, you can access its value by its handle:

::: code
```twig
{% set value = entry.myFieldHandle %}
```
```php
$value = $entry->myFieldHandle;
```
:::

That will give you a [DateTime](http://php.net/manual/en/class.datetime.php) object that represents the selected date, or `null` if no date was selected.

::: code
```twig
{% if entry.myFieldHandle %}
  Selected date: {{ entry.myFieldHandle|datetime('short') }}
{% endif %}
```
```php
if ($entry->myFieldHandle) {
    $selectedDate = \Craft::$app->getFormatter()->asDatetime(
        $entry->myFieldHandle,
        'short'
    );
}
```
:::

Craft and Twig provide several Twig filters for manipulating and outputting dates, which you can use depending on your needs:

- [date](../twig/filters.md#date)
- [time](../twig/filters.md#time)
- [datetime](../twig/filters.md#datetime)
- [duration](../twig/filters.md#duration)
- [timestamp](../twig/filters.md#timestamp)
- [atom](../twig/filters.md#atom)
- [rss](../twig/filters.md#rss)
- [date_modify](https://twig.symfony.com/doc/3.x/filters/date_modify.html)

#### Timezones

Craft treats all dates as though they are in the system’s timezone, except when one is set explicitly for a date field.

The returned `DateTime` object’s timezone will be set, accordingly. If you wish to display the date in a _different_ timezone than it was defined, use the `timezone` argument supported by Craft’s [date](../twig/filters.md#date), [datetime](../twig/filters.md#datetime) and [time](../twig/filters.md#time) Twig filters.

::: tip
This flexibility is only a feature of date fields, and native element properties (like entries’ _post date_) are always stored in the system timezone.
:::

### Saving Date Fields

If you have an element form, such as an [entry form](https://craftcms.com/knowledge-base/entry-form), that needs to contain a Date field, you can create a `date` or `datetime-local` input.

If you just want the user to be able to select a date, use a `date` input:

```twig
{% set currentValue = entry is defined and entry.myFieldHandle
  ? entry.myFieldHandle|date('Y-m-d', timezone='UTC')
  : '' %}

<input type="date" name="fields[myFieldHandle]" value="{{ currentValue }}">
```

If you want the user to be able to select a time as well, use a `datetime-local` input:

```twig
{% set currentValue = entry is defined and entry.myFieldHandle
  ? entry.myFieldHandle|date('Y-m-d\\TH:i', timezone='UTC')
  : '' %}

<input type="datetime-local" name="fields[myFieldHandle]" value="{{ currentValue }}">
```

#### Customizing the Timezone

By default, Craft will assume the date is posted in UTC. As of Craft 3.1.6 you can post dates in a different timezone by changing the input name to `fields[myFieldHandle][datetime]` and adding a hidden input named `fields[myFieldHandle][timezone]`, set to a [valid PHP timezone](http://php.net/manual/en/timezones.php):

```twig
{# Use the timezone selected under Settings → General Settings → Time Zone #}
{% set tz = craft.app.getTimezone() %}

{# Or set a specific timezone #}
{% set tz = 'America/Los_Angeles' %}

{% set currentValue = entry is defined and entry.myFieldHandle
  ? entry.myFieldHandle|date('Y-m-d\\TH:i', timezone=tz)
  : '' %}

<input type="datetime-local" name="fields[myFieldHandle][datetime]" value="{{ currentValue }}">
{{ hiddenInput('fields[myFieldHandle][timezone]', tz) }}
```

Or you can let users decide which timezone the date should be posted in:

```twig
{% set currentValue = entry is defined and entry.myFieldHandle
  ? entry.myFieldHandle|date('Y-m-d\\TH:i', timezone='UTC')
  : '' %}

<input type="datetime-local" name="fields[myFieldHandle][datetime]" value="{{ currentValue }}">

<select name="fields[myFieldHandle][timezone]">
  <option value="UTC" selected>UTC</option>
  <option value="America/Los_Angeles">Pacific Time</option>
  {# ... #}
</select>
```

#### Posting the Date and Time Separately

If you’d like to post the date and time as separate HTML inputs, give them the names `fields[myFieldHandle][date]` and `fields[myFieldHandle][time]`.

The date input can either be set to the `YYYY-MM-DD` format, or the current locale’s short date format.

The time input can either be set to the `HH:MM` format (24-hour), or the current locale’s short time format.

::: tip
To find out what your current locale’s date and time formats are, add this to your template:

```twig
Date format: <code>{{ craft.app.locale.getDateFormat('short', 'php') }}</code><br>
Time format: <code>{{ craft.app.locale.getTimeFormat('short', 'php') }}</code>
```

Then refer to PHP’s [date()](http://php.net/manual/en/function.date.php) function docs to see what each of the format letters mean.
:::

This strategy can by combined with [timezone customization](#customizing-the-timezone).
