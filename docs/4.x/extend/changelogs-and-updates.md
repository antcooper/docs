# Changelogs and Updates

When you [publish](plugin-store.md) your plugin in the Plugin Store, its changelog is automatically scraped and displayed in a dedicated tab, as well as in Craft’s <Journey path="Utilities, Updates" /> screen wherever it is installed.

::: tip
Refer to [Craft’s own changelog](https://github.com/craftcms/cms/blob/4.x/CHANGELOG.md) as an example of syntax and best practices.
:::

## Setting Up a Changelog

Create a `CHANGELOG.md` file at the root of your plugin’s repo, where you can start documenting release notes for your plugin. Use something like this as a starting point:

```markdown
# Release Notes for <Plugin Name>
```

Within the changelog, releases must be listed in **descending order** (newest-to-oldest). (When displaying available plugin updates, the Plugin Store will stop parsing a plugin’s changelog as soon as it finds a version that is older than or equal to the user’s installed version.)

## Version Headings

Version headings in your changelog must follow this format:

```markdown
## X.Y.Z - YYYY-MM-DD
```

There’s a little wiggle room on that:

- Other text can come before the version number, like the plugin’s name.
- A 4th version number is allowed (e.g. `1.2.3.4`).
- Prerelease versions are allowed (e.g. `1.0.0-alpha.1`).
- The version can start with `v` (e.g. `v1.2.3`).
- The version can be hyperlinked (e.g. `[1.2.3]`).
- Dates can use dots as separators, rather than hyphens (e.g. `2017.01.21`).

Any H2s that don’t follow this format will be ignored, including any content that follows them leading up to the next H2.

## Release Notes

All content that follows a version heading (up to the next H2) will be treated as the release notes for the update.

When writing release notes, we recommend that you follow the guidelines at [keepachangelog.com](https://keepachangelog.com/), but all forms of [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/#GitHub-flavored-markdown) are allowed. The only thing that is *not* allowed is actual HTML code, which will be escaped.

### Alerts

You can include notes and warnings in your release notes using [GitHub’s alert syntax](https://github.com/orgs/community/discussions/16925):

```markdown
> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

> [!IMPORTANT]  
> Crucial information necessary for users to succeed.

> [!WARNING]  
> Critical content demanding immediate user attention due to potential risks.
```

These alerts will each be styled appropriately within the plugin’s changelog in the Plugin Store and within the Updates utility. Updates which contain alerts will automatically be expanded in the Updates utility.

### Links

If you have any reference-style links in your release notes, you will need to define the URLs *before* the following version heading:

```markdown
## 2.0.1 - 2017-02-01
### Fixed issue [#123]

[#123]: https://github.com/pixelandtonic/foo/issues/123

## 2.0.0 - 2017-01-31
### Added
- New [superFoo] config setting

[superFoo]: https://docs.my-plugin.tld/config#superFoo
```

## Critical Updates

If an update contains a fix for a critical security vulnerability or other dangerous bug, you can alert your users about it by adding `[CRITICAL]` to the end of the version heading:

```markdown
## 2.0.1 - 2017-01-21 [CRITICAL]
### Fixed
- Reverted change to `$potus` due to security vulnerabilities
```

When Craft finds out that a critical update is available, it will post a message about it to the top of all control panel pages, and give the update special attention on the <Journey path="Utilities, Updates" /> page.
