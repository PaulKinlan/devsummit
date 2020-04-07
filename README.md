```
npm i
npm run build # (node 12+ required)
```

The `src` folder uses [11ty](https://www.11ty.io).

# Configs

`/confbox.config.js` allows you to configure the start, end, and timezone of the conference.

``` JavaScript
module.exports = {
  /**
   * Origin of the conference, for creating absolute URLs.
   */
  origin: 'https://paulkinlan-devsummit.glitch.me',
  /**
   * Path of the site. / if it's top-level.
   */
  path: '/',
  /**
   * Name of the conference.
   */
  conferenceName: 'Clone Con',
  teaser:
    "Join the Chrome team for our two-day summit to learn about the latest techniques for building for the modern Web, get an early insight into what we're working on, and to share your thoughts on how we can move the platform forward, together.",
  /**
   * Data of the conference venue.
   */
  venue: {
    name: 'Yerba Buena Center for the Arts',
    city: 'San Francisco',
    region: 'CA',
    country: 'US',
    postalCode: '94103',
    streetAddress: '701 Mission St',
    mapsLink: 'https://goo.gl/maps/TBiTuFitnqe1wxPW7',
  },
  /**
   * Conference Twitter account
   */
  twitter: '@ChromiumDev',
  /**
   * Timezone of the conference, in the form [+-]HHMM.
   * Examples: -0700, +0100, +0530
   */
  timezone: '-0800',
  /**
   * Start of conference in the above timezone, in the format: YYYY/MM/DD HH:mm.
   */
  start: '2019/11/11 09:00',
  /**
   * Start of conference in the above timezone, in the format: YYYY/MM/DD HH:mm.
   */
  end: '2019/11/12 17:00',
  /**
   * Additional schedule items. These are merged with the content in /sessions/.
   */
  extraSchedule: [
    // Day 1
    {
      title: 'Break & livestream exclusive lightning talks',
      start: '2019/11/11 11:35',
      end: '2019/11/11 12:15',
      icon: '/schedule/assets/coffee.svg',
      livestreamed: true,
    },
    {
      title: 'Lunch & livestream exclusive lightning talks',
      start: '2019/11/11 13:25',
      end: '2019/11/11 14:35',
      icon: '/schedule/assets/food.svg',
      livestreamed: true,
    },
    {
      title: 'Break & livestream exclusive lightning talks',
      start: '2019/11/11 16:05',
      end: '2019/11/11 16:40',
      icon: '/schedule/assets/coffee.svg',
      livestreamed: true,
    },
    // Day 2
    {
      title: 'Break & livestream exclusive lightning talks',
      start: '2019/11/12 11:45',
      end: '2019/11/12 12:15',
      icon: '/schedule/assets/coffee.svg',
      livestreamed: true,
    },
    {
      title: 'Lunch & livestream exclusive lightning talks',
      start: '2019/11/12 13:25',
      end: '2019/11/12 14:35',
      icon: '/schedule/assets/food.svg',
      livestreamed: true,
    },
    {
      title: 'Break & livestream exclusive lightning talks',
      start: '2019/11/12 16:05',
      end: '2019/11/12 16:35',
      icon: '/schedule/assets/coffee.svg',
      livestreamed: true,
    }
  ],
};

```

# Schedule

The schedule is automatically generated from the meta-data in the each of the talks.

Sessions are stored in the `src/sessions/` folder.

Speakers are stored in the `src/speakers` folder.

## Sessions

To create a session, create a markdown file in the `src/sessions` folder (i.e, `keynote.md`)

Then add some YAML front matter that describes the session. For example:

```YAML
---
layout: layouts/session/index.njk
title: HTML isn't done!
topics:
  - Features
speakers:
  - nicole-sullivan
  - greg-whitworth
start: 2019/11/12 10:35
end: 2019/11/12 11:10
description: The web platform is now ready to pave those cow paths and extend HTML. And we need your help with these brand new shiny components…
youtubeId: ZFvPLrKZywA
---
```

* `start` and `end` will ensure that the session gets added to the schedule. The dates need to be formatted as follows `YYYY/MM/DD HH:mm`
* `speakers` is an ID to the speaker so that their profile can be rendered.
* `youtubeId` is only needed when the session has happened and there is a recording available.

## Speakers

In this current setup, speakers don't have their own pages, but we need to define their metadata so that we can show the speakers against any given session.

Speakers can be defined in the `src/speakers/` folder. The filename is formatted along the following guidelines `[firstname-familyname].md`, with the same format being used as the speaker ID in the session information.

```YAML
---
name: Adam Argyle
title: Google
avatar: /assets/speakers/adam-argyle.jpg
link: https://twitter.com/argyleink
---
```

# Data

All templates have access to the following:

- `conf.utcOffset` - The offset of the conference from the UTC timezone in milliseconds.
- `conf.start` - Start of the conference as a timestamp.
- `conf.end` - End of the conference as a timestamp.

# Helpers

## `{% className cssPath, class %}`

CSS files are processed as [CSS Modules](https://github.com/css-modules/css-modules). Within a template, reference classnames like this:

```njk
{% className cssPath, class %}
```

…and it will output the transformed class name. Eg:

```njk
{% set css = "/_includes/talk/style.css" %}
<h2 class="{% className css, 'talk-header' %}">Building offline-first apps</h2>
<div class="{% className css, 'talk-description' %}">
  …
</div>
```

In the example above, `set` is used to avoid repeating the path to the CSS.

## `{% css page, cssURL %}`

This will output a `<link>` pointing to the `cssURL` unless it's been included already for the current page. This means you can use an include multiple times without loading the CSS multiple times.

- `page` - This is the page object available in every template.
- `cssURL` - CSS url.

Example:

```njk
{% set css = "/_includes/talk/style.css" %}
{% css page, css %}
```

## `confboxAsset(url)`

In templates and CSS, references assets via `confboxAsset('/path/to/asset.jpg')`. This will be replaced with the hashed name of the asset.

```njk
<link rel="stylesheet" href="confboxAsset('/_includes/talk/style.css')">
```

```css
.whatever {
  background: url("confboxAsset(asset.jpg)");
}
```

Assets ending `.js` will be bundled together using Rollup.

## `{% confDate date, format %}`

This will take a `date` and format it for the timezone of the conference (as set in `/confbox.config.js`).

- `date` - The date to display. This can be a `Date` object or a timestamp.
- `format` - A formatting string as used by [date-and-time](https://www.npmjs.com/package/date-and-time#formatdateobj-formatstring-utc).

```njk
<p>The conference starts {% confDate conf.start, 'MMMM DD' %}</p>
```

## `{% isoDate date %}`

Returns an ISO 8601 version of a date. This is suitable for `<time datetime>` and other machine-readable formats like iCal.

- `date` - The date to display. This can be a `Date` object or a timestamp.
