additional date-time classes that complement those in js-joda
==============================================

[![npm version](https://badge.fury.io/js/js-joda-locale.svg)](https://badge.fury.io/js/js-joda-locale)
[![Build Status](https://travis-ci.org/js-joda/js-joda-locale.svg?branch=master)](https://travis-ci.org/js-joda/js-joda-locale)
![Sauce Test Status](https://saucelabs.com/buildstatus/js-joda-locale)
[![Coverage Status](https://coveralls.io/repos/js-joda/js-joda-locale/badge.svg?branch=master&service=github)](https://coveralls.io/github/js-joda/js-joda-locale?branch=master)
[![Tested node version](https://img.shields.io/badge/tested_with-current_node_LTS-blue.svg?style=flat)]()
[![Greenkeeper badge](https://badges.greenkeeper.io/js-joda/js-joda-locale.svg)](https://greenkeeper.io/)

![Sauce Test Status](https://saucelabs.com/browser-matrix/js-joda-locale.svg)

## Motivation

Implementation of locale specific funtionality for js-joda, providing function not implemented in js-joda core

Especially this implements patterns elements to print and parse locale specific dates

### Usage ###

also see examples in  [examples folder](examples/)

#### Dependencies

The implementation requires cldr data provided by the [cldr-data](https://github.com/rxaviers/cldr-data-npm) package 
and uses [cldrjs](https://github.com/rxaviers/cldrjs) to load the data.
This is necessary to display and parse locale specific data, e.g DayOfWeek or Month Names.

The cldr data is a peer dependency of this package, meaning it must be provided/`npm install`ed by users of `js-joda-locale`

Since the complete cldr-data package can be quite large, the examples and documentation below show ways to dynamically
load or reduce the amount of data needed.

The implementation of `js-joda-locale` also requires `js-joda-timezone` package e.g. to parse and output timezone names and offsets

### Node

Install joda using npm

```shell
    npm install js-joda
    npm install js-joda-timezone
    npm install cldr-data
    npm install cldrjs
    npm install js-joda-locale
```

To enable js-joda-locale you will need to provide it to js-joda as a plugin via the `use` function

```javascript
jsJoda.use('js-joda-locale')
```
since `js-joda-locale` requires `js-joda-timezone` it will also need to be provided, as shown 
in the following examples

NOTE: if you are using a destructuring assignment in ES6, it should only be performed *after* 
the plugin `use` 

### es5

```javascript
    var joda = require('js-joda').use(require('js-joda-timezone')).use(require('../dist/js-joda-locale'));    
    var zdt = joda.ZonedDateTime.of(2016, 1, 1, 0, 0, 0, 0, joda.ZoneId.of('Europe/Berlin'));
    console.log('en_US formatted string:', zdt.format(joda.DateTimeFormatter.ofPattern('eeee MMMM dd yyyy GGGG, hh:mm:ss a zzzz, \'Week \' ww, \'Quarter \' QQQ').withLocale(joda.Locale.US)));
```
also see [examples/usage_node.js](examples/usage_node.js) or [examples/usage_node_build.js](examples/usage_node_build.js) 

### es6

```ecmascript 6
   import { use as jsJodaUse } from 'js-joda';
   import jsJodaTimeZone from 'js-joda-timezone';
   import jsJodaLocale from 'js-joda-locale';
   
   const { DateTimeFormatter, Locale, ZonedDateTime, ZoneId } = jsJodaUse(jsJodaTimeZone).use(jsJodaLocale);
   
   const zdt = ZonedDateTime.of(2016, 1, 1, 0, 0, 0, 0, ZoneId.of('Europe/Berlin'));
   console.log('en_US formatted string:', zdt.format(DateTimeFormatter.ofPattern('eeee MMMM dd yyyy GGGG, hh:mm:ss a zzzz, \'Week \' ww, \'Quarter \' QQQ').withLocale(Locale.US)));
```

also see the [example](examples/usage_es6.js)

### Use prebuilt locale packages

Since the process described [below](#without-prebuilt-locale-packages) requires a lot of setup and internal knowledge, 
we provide the possibility to create prebuilt locale packages when installing `js-joda-locale`. This basically automates the steps described 
in the description how to setup webpack for use with js-joda-locale below

To optimize the build speed and output it is important to only load the required `cldr-data` files, so you should
in `package.json` file define which parts of cldr-data to download and install

(for more information see the [cldr-data-npm docs](https://github.com/rxaviers/cldr-data-npm#locale-coverage))
```
...
"cldr-data-coverage": "core",
"cldr-data-urls-filter": "(cldr-core|cldr-numbers-modern|cldr-dates-modern)"
...
```
(data-coverage `core` only downloads data for the most popular languages / locales, while the urls-filter defines 
which parts of cldr-data are required for `js-joda-locale` to work)

Without further configuration the `postinstall` script of `js-joda-locale` will create packages for a core set of locales
(currently `en`, `es` and `de`), these can be required using

```js
var jsJodaLocale = require('js-joda-locale/build/package/en')
```

The configuration can be adapted by creating a config section for `js-joda-locale` in your package.json file, like this:

```json
"js-joda-locale": {
    "packages": {
      "core": [
        "de.*",
        "en.*",
        "es.*"
      ],
      "others": [
        "zh",
        "hi",
        "ru"
      ],
    },
    "stats": false,
    "debug": false
  }
```

This would result in two pacakges being built, one containing all `en.*`,`es.*`, `de.*` locales called `core` and one only containing the `zh`, `hi`, `ru` locales called `others`.
These could then be required using

```js
var jsJodaLocale = require('js-joda-locale/build/package/core')
```
or
```js
var jsJodaLocale = require('js-joda-locale/build/package/others')
```

Note that when specifying locales a regex style pattern is possible, but if you *only* specify a specific 
cldr-data locale, only this data can be used, e.g. if only the locale `en` is specified, only
the default data (in this case for the `en_US` style would be loaded), using `en_GB` would *NOT* 
be possible but either require an extra package, several locales in one package or a regex pattern like `en.*`

### Without prebuilt locale packages

It is also possible to not use the prebuilt packages. This results in a more complex setup, but also more control over the complete process. 

### Browser
- using requirejs to load
- might also be possible with the bower version of cldr-data
 
see the [example](examples/usage_browser.html)

### Packaging with webpack, minimizing package size
 
Since the cldr-data files can still be quite large, it is possible to only load the files needed for your application

Also possible would be to use webpack to reduce the overall size of the cldr-data (similar approaches should work with 
different packaging tools than webpack). 

So the following tips are just one way to get the general idea on how to reduce the size of needed cldr-data, we use this
for our karma testing setup in [karma.conf.js](karma.conf.js) and to build the prebuilt locale packages

In `package.json` file define which parts of cldr-data to download and install

(for more information see the [cldr-data-npm docs](https://github.com/rxaviers/cldr-data-npm#locale-coverage))
```
...
"cldr-data-coverage": "core",
"cldr-data-urls-filter": "(cldr-core|cldr-numbers-modern|cldr-dates-modern)"
...
```
(data-coverage `core` only downloads data for the most popular languages / locales, while the urls-filter defines 
which parts of cldr-data are required for `js-joda-locale` to work)

In e.g. webpack.config.js, define which parts/locales of the cldr-data files should end up in the final package

You can for example use the `null-loader` to disable loading cldr-data except for the absolutely required parts/locales

```js
use: [{ loader: 'null-loader' }],
resource: {
    // don't load everything in cldr-data
    test: path.resolve(__dirname, 'node_modules/cldr-data'),
    // except the actual data we need (supplemental and de, en, fr locales from main)
    exclude: [
        path.resolve(__dirname, 'node_modules/cldr-data/main/de'),
        path.resolve(__dirname, 'node_modules/cldr-data/main/en'),
        path.resolve(__dirname, 'node_modules/cldr-data/main/fr'),
        path.resolve(__dirname, 'node_modules/cldr-data/supplemental'),
    ],
}
``` 
or (as we do for our prebuilt packages) use the CldrDataIgnorePlugin, provided in `utils/CldrDataIgnorePlugin.js`
```json
    "plugins": [
        new CldrDataIgnorePlugin(modulesDir, locales)),
    ]

```
where modulesDir is the absolute path to `node_modules` and `locales` is an array of locales to use as they can be defined 
for the prebuilt packages. This will only load the absolutely required files for js-joda-locale, it is what we use internally
for the prebuilt packages and to build packages for our karma tests as well.

Depending on your usecase it might also be necessary to define a  "faked" cldr-data module that loads 
the cldr-data files, this is necessary at least if the code needs to run in the browser since the 
cldr-data load uses modules not available in browser (e.g. `fs`)

```js
    // add cldr-data load workaround
    resolve = {
        alias: {
            'cldr-data$': path.resolve(__dirname, 'test/utils/karma_cldrData.js'),
        }
    };

``` 

These should be the minimum required parts for js-joda-locale 

see the [karma.conf.js](karma.conf.js)

## Implementation details

provides methods for the following pattern letters of the [DateTimeFormatterBuilder](https://js-joda.github.io/js-joda/esdoc/class/src/format/DateTimeFormatterBuilder.js~DateTimeFormatterBuilder.html#instance-method-appendPattern~DateTimeFormatter.html#static-method-ofPattern) 
and [DateTimeFormatter](https://js-joda.github.io/js-joda/esdoc/class/src/format/DateTimeFormatter.js~DateTimeFormatter.html#static-method-ofPattern) 
classes of js-joda

Localized Text
- `a` for am/pm of day
- `G` for era
- `q`/`Q` for localized quarter of year

Zone Text
- `z` for time zone name
- `Z` for localized ZoneOffsets
- `O` for localized ZoneOffsets

Week Information
- `w` for week-of-year
- `W` for week-of-month
- `Y` for week-based-year
- `e` for localized day-of-week
- `c` for localized day-of-week

some of these are only partially localized, e.g. `Q` only if three or more `Q` are used, one or two `Q` also 
work with plain `js-joda` without using `js-joda-locale`

## License

* js-joda-locale is released under the [BSD 3-clause license](LICENSE)
* The author of joda time and the lead architect of the JSR-310 is Stephen Colebourne.

