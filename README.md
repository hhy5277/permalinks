# permalinks [![NPM version](https://img.shields.io/npm/v/permalinks.svg?style=flat)](https://www.npmjs.com/package/permalinks) [![NPM monthly downloads](https://img.shields.io/npm/dm/permalinks.svg?style=flat)](https://npmjs.org/package/permalinks) [![NPM total downloads](https://img.shields.io/npm/dt/permalinks.svg?style=flat)](https://npmjs.org/package/permalinks) [![Linux Build Status](https://img.shields.io/travis/permalinks/permalinks.svg?style=flat&label=Travis)](https://travis-ci.org/permalinks/permalinks)

> Easily add powerful permalink or URL routing/URL rewriting capablities to any node.js project. Can be used in static site generators, build systems, web applications or anywhere you need to do path or URL transformation.

Please consider following this project's author, [Jon Schlinkert](https://github.com/jonschlinkert), and consider starring the project to show your :heart: and support.

- [Install](#install)
- [Quickstart](#quickstart)
- [API](#api)
- [Docs](#docs)
  * [Structure](#structure)
    + [Alternative syntax](#alternative-syntax)
    + [file](#file)
    + [locals](#locals)
  * [Context](#context)
    + [locals](#locals-1)
    + [file.data](#filedata)
    + [options.data](#optionsdata)
    + [File path properties](#file-path-properties)
  * [Presets](#presets)
  * [Helpers](#helpers)
    + [file helper](#file-helper)
  * [Additional resources](#additional-resources)
- [About](#about)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm install --save permalinks
```

## Quickstart

Add permalinks support to any JavaScript project using node's `require()` system with the following line of code:

```js
const permalink = require('permalinks');
```

_(The main export is a function that automatically creates an instance of `Permalinks` and calls the `permalinks.formt()` method. As an alternative, if you need to access any internal methods, the [API documentation](#api) shows how to create an instance of `Permalinks`.)_

**Usage**

```js
permalink(structure, file[, options]);
```

**Params**

* [structure](https://structure.js.org/): `{string}` (required)  A kind of template that determines what a permalink will look like when it's rendered.
* [file](https://github.com/aconbere/node-file-utils): `{string|object}` (required) Locals, or a file object or path (the file associated with the permalink).
* [locals](https://github.com/active9/Locals): `{object}` (optional) Additional data to use for resolving placeholder values in the structure

**Examples**

```js
console.log(permalink('/:area/:slug/index.html', {area: 'blog', slug: 'idiomatic-permalinks'}));
//=> '/blog/idiomatic-permalinks/index.html'
console.log(permalinks('/blog/:stem/index.html', 'src/about.hbs'));
//=> '/blog/about/index.html'
console.log(permalinks('/archives/:stem-:num.html', 'src/foo.hbs', {num: 21}));
//=> '/archives/foo-21.html'
```

***

## API

### [Permalinks](index.js#L36)

Create an instance of `Permalinks` with the given `options`

**Params**

* `options` **{Options|String}**

**Example**

```js
const Permalinks = require('permalinks').Permalinks;
const permalinks = new Permalinks();
console.log(permalinks.format(':stem/index.html'), {path: 'src/about.hbs'});
//=> 'about/index.html'
```

### [.parse](index.js#L84)

Uses [parse-filepath](https://github.com/jonschlinkert/parse-filepath) to parse the `file.path` on the given file object. This method is called by the [format](#format) method, but you can use it directly and pass the results as `locals` (the last argument) to the `.format` method if you need to override or modify any path segments.

**Params**

* `file` **{Object}**
* `returns` **{Object}**

**Example**

```js
console.log(permalinks.parse({path: 'foo/bar/baz.md'}));
// { root: '',
//   dir: 'foo/bar',
//   base: 'baz.md',
//   ext: '.md',
//   name: 'baz',
//   extname: '.md',
//   basename: 'baz.md',
//   dirname: 'foo/bar',
//   stem: 'baz',
//   path: 'foo/bar/baz.md',
//   absolute: [Getter/Setter],
//   isAbsolute: [Getter/Setter] }
```

### [.format](index.js#L119)

Generate a permalink by replacing `:prop` placeholders in the specified `structure` with data from the given `file` and `locals`.

**Params**

* `structure` **{String}**: Permalink structure or the name of a registered [preset](#preset).
* `file` **{Object|String}**: File object or file path string.
* `locals` **{Object}**: Any additional data to use for resolving placeholders.
* `returns` **{String}**

**Example**

```js
const fp = permalinks.format('blog/:stem/index.html', {path: 'src/about.hbs'});
console.log(fp);
//=> 'blog/about/index.html'
```

### [.preset](index.js#L150)

Define a permalink `preset` with the given `name` and `structure`.

**Params**

* `name` **{String}**: If only the name is passed,
* `structure` **{String}**
* `returns` **{Object}**: Returns the `Permalinks` instance for chaining

**Example**

```js
permalinks.preset('blog', 'blog/:stem/index.html');
const url = permalinks.format('blog', {path: 'src/about.hbs'});
console.log(url);
//=> 'blog/about/index.html'
```

### [.helper](index.js#L194)

Define permalink helper `name` with the given `fn`. Helpers work like any other variable on the context, but they can optionally take any number of arguments and can be nested to build up the resulting string.

**Params**

* `name` **{String}**: Helper name
* `fn` **{Function}**
* `returns` **{Object}**: Returns the Permalink instance for chaining.

**Example**

```js
permalinks.helper('date', function(file, format) {
  return moment(file.data.date).format(format);
});

const structure1 = ':date(file, "YYYY/MM/DD")/:stem/index.html';
const file1 = permalinks.format(structure1, {
  data: {date: '2017-01-01'},
  path: 'src/about.tmpl'
});

const structure2 = ':name(upper(stem))/index.html';
const file2 = permalinks.format(structure2, {
  data: {date: '2017-01-01'},
  path: 'src/about.tmpl'
});

console.log(file1);
//=> '2017/01/01/about/index.html'

console.log(file2);
//=> '2017/01/01/about/index.html'
```

### [.context](index.js#L219)

Add a function for calculating the context at render time. Any number of context functions may be used, and they are called in the order in which they are defined.

**Params**

* `fn` **{Function}**: Function that takes the `file` being rendered and the `context` as arguments. The permalinks instance is exposed as `this` inside the function.
* `returns` **{Object}**: Returns the instance for chaining.

**Example**

```js
permalinks.context(function(file, context) {
  context.site = { title: 'My Blog' };
});

permalinks.helper('title', function() {
  return this.file.data.title || this.context.site.title;
});
```

### [.normalizeFile](index.js#L331)

Normalize the given `file` to be a [vinyl](https://github.com/gulpjs/vinyl) file object.

**Params**

* `file` **{String|Object}**: If `file` is a string, it will be converted to the `file.path` on a file object.
* `file` **{Object}**
* `options` **{Object}**
* `returns` **{Object}**: Returns the normalize [vinyl](https://github.com/gulpjs/vinyl) file.

**Example**

```js
const file = permalinks.normalizeFile('foo.hbs');
console.log(file);
//=> '<File "foo.hbs">'
```

## Docs

**What is a permalink?**

The word "permalink" is a portmanteau for "permanent link". A permalink is a URL to a page, post or resource on your site that is intended to stay the same as long as the site exists.

Most blogging platforms and static site generators offer some level of support or plugins for generating permalinks. To users surfing your site, a permalink represents an entire "absolute" URL, such as `https://my-site.com/foo/index.html`. However, you will probably only need to generate the relative path portion of the URL: `/foo/index.html`.

**How are permalinks created?**

A permalink is created by replacing placeholder values in a [permalink structure][] (like `:slug` in `/blog/:slug/index.html`) with actual data. This data is provided by the user in the form of [locals](https://github.com/active9/Locals), and/or created by parsing the [file path][] (of the file for which we are generating a permalink):

```js
// given this structure
'/blog/:slug/index.html'

// and this data
{ slug: 'my-first-blog-post' }

// the resulting (relative part of the) permalink would be
'/blog/my-first-blog-post/index.html'
```

This is covered in greater detail in the following documentation.

### Structure

A permalink structure is a template that determines how your permalink should look after it's rendered.

Structures may contain literal strings, and/or placeholder strings like `:name`, which will be replaced with actual data when the [format](#format) method is called.

**Examples**

Given a file named `10-powerful-seo-tips.md`, we can change the aesthetics or semantics of the resulting permalink simply by changing the structure. For example:

```js
'/blog/:name/'
//=> blog/10-powerful-seo-tips/
'/blog/:name.html'
//=> blog/10-powerful-seo-tips.html
'/blog/:name/index.html'
//=> blog/10-powerful-seo-tips/index.html
```

With a bit more information, provided as [locals](https://github.com/active9/Locals) or from the [file](https://github.com/aconbere/node-file-utils) itself (such as `file.data`, which commonly holds parsed front-matter data), we can get much more creative:

For example, if `file.data.slug` was defined as `more-powerful-seo-tips`, we might use it like this:

```js
'/blog/:data.slug/index.html'
//=> blog/more-powerful-seo-tips/index.html
```

We can get even more creative using [helpers](https://github.com/fshost/helpers). We might, for example:

* create a helper that parses a date from front-matter, such as `2017-01-02`, into year, month and day
* slugifies a `title`, like "Foo Bar Baz", to be dash-separated and all lower-case
* adds the index of a file from an array of files

```js
'/blog/:date(data.date, "YYYY/MM/DD")/index.html'
//=> blog/:slugify(data.title)/index.html
'blog/:slugify(data.title)/index.html' 
//=> blog/:slugify(data.title)/index.html
'/archives/:num.html'
//=> archives/23.html
```

Your creativity is the only limitation!

#### Alternative syntax

Permalinks uses [handlebars](http://www.handlebarsjs.com/) to resolve templates, which means that you can also/alternatively use handlebar's native mustache syntax for defining permalink structures:

```handlebars
/blog/{{name}}/index.html
/blog/{{foo.bar.baz}}/index.html
/blog/{{year}}/{{month}}/{{day}}/{{slug}}.html
```

**Why handlebars?**

There are a few reasons we decided to use Handlebars over parsing/rendering the `:params` internally:

* Excellent context handling and resolution of variables. We have a lot of experience with templating, parsing and rendering. Most libraries choose the "minimalist" route for resolving `:prop` strings because of all the edge cases that crop up with more advanced features, and it's fast. With Handlebars, we compromise very slightly on speed (although it's still very fast) in exchange for power and reliability.
* Helpers! You can [use helpers](#helpers) to modify any of the variables in your permalink structure. You can even register helpers from great libraries like [template-helpers](https://github.com/jonschlinkert/template-helpers) or [handlebars-helpers](https://github.com/helpers/handlebars-helpers)
* Error handling: handlebars provides great error handling, but we especially like that handlebars allows us to choose what should happen when a missing helper is identified. We use that feature to detect dates and other similar patterns that might need to be parsed before calling other helpers (for example, let's say you define `:month/:year/:day`, but none of those variables exist. handlebars will call the `helperMissing` helper for each param, which gives you the opportunity to, for example, parse `file.data.date` into an object with those properties, and return the values instead of throwing an error)
* Other advanced handlebars features, like subexpressions and object paths (`foo.bar.baz`)

**Example using subexpressions**

```js
'/blog/{{lower (slugify data.title)}}/index.html'
// or 
'/blog/:lower((slugify data.title))/index.html'
```

Note that `:lower(` is effectively converted to `{{lower`, thus only the outermost subexpression will result in double parentheses. This is easier to see in the following example, which has two nested subexpressions:

```js
'/blog/:foo((bar (baz "qux")))/index.html'
```

If the `:param` syntax seems confusing, feel free to stick with the handlebars syntax.

#### file

**Type**: `{String|Object}` (optional if `locals` is passed)

If a file object or string is passed, it will be parsed using node's `path.parse()` method, and merged with locals to create the context for resolving `:props` in the structure.

```js
permalinks(structure, file, locals);
//                     ↑
```

**File handling**

Files may be defined as a string or an object. If defined as a string, the filepath will be converted to an object and set on the `file.path` property.

In other words `'a/b/c.md'` becomes `{ path: 'a/b/c.md' }`.

#### locals

**Type**: (optional if `file` is passed)

Additional data to use for resolving `:props` in the structure

```js
permalinks(structure, file, locals);
//                            ↑
```

### Context

The "context" is an in-memory object that is used to resolve placeholders in permalink [structures](#structures).

The context object is created dynamically before rendering each permalinks, by merging the following objects:

* [file path properties](#file-path-properties):
* [options.data](#options.data):
* [file.data](#file.data):
* [locals](#locals):

#### locals

If a `locals` object is passed as the last argument, it will be merged onto the context to be used for resolving placeholders in the permalink [structure](#structure).

```js
console.log(permalinks('/blog/:name/index.:ext', 'src/about.hbs', {ext: '.html'}));
//=> '/blog/about/index.html'

console.log(permalinks(':category/:name/index.html', 'src/about.hbs', {category: 'blog'}));
//=> 'blog/about/index.html'
```

#### file.data

**Type**: `object`

**Default**: `undefined`

Populate the `file.data` object with additional data to use for a specific file when rendering a permalink structure.

#### options.data

**Type**: `object`

**Default**: `undefined`

Provide additional data to use when rendering permalinks for all files.

#### File path properties

Values on the provided `file` object are merged onto the root of the context and are used to resolve placeholders in the permalink structure. File values can be overridden by [locals](#locals) or [helpers](#helpers).

_A file does not need to be passed_, but if a file is provided with at least a `file.path` property, the path will be parsed to provide as many of the following variables on the context as possible.

| **variable** | **description** | 
| --- | --- |
| `file.base` | Gets and sets base directory. Used for created relative paths. When `null` or `undefined`, it simply proxies the `file.cwd` property. Will always be normalized and have trailing separators removed. Throws when set to any value other than non-empty strings or `null`/`undefined`. |
| `file.path` | Gets and sets the absolute pathname string or `undefined`. This value is always normalized and trailing separators are removed. Throws when set to any value other than a string. |
| `file.relative` | Gets the result of `path.relative(file.base, file.path)`. This is a getter and will throw if set or when `file.path` has not already been set. |
| `file.dirname` | Gets and sets the dirname of `file.path`. Will always be normalized and have trailing separators removed. Throws when `file.dirname` is not exlicitly defined and/or `file.path` is not set. |
| `file.basename` | Gets and sets the basename of `file.path`. Throws when `file.basename` is not exlicitly defined and/or `file.path` is not set. |
| `file.stem` | Gets and sets stem (filename without suffix) of `file.path`. Throws when `file.stem` is not exlicitly defined and/or  `file.path` is not set. |
| `file.name` | Alias for `file.stem`. |
| `file.extname` | Gets and sets extname property of `file.path`. |
| `file.ext` | Alias for `file.extname`. |

**Example**

```
┌──────────────────────────────────────────────┐
│                     file.path                    │
┌─────────────────────┬────────────────────────┐
│      file.dirname     │        file.basename     │
│                       ├──────────┬─────────────┤
│                       │ file.name │              │
│                       │ file.stem │ file.extname │
" /home/user/foo/src    /   about        .tmpl      "
└─────────────────────┴──────────┴─────────────┘
```

A `file.relative` value can only be calculated if both `file.base` and `file.path` exist:

```
┌──────────────────────────────────────────────┐
│                     file.path                    │
┌─────────────────────┬────────────────────────┐
│      file.base        │        file.relative     │
└─────────────────────┴────────────────────────┘
```

### Presets

Easily store and re-use permalink structures.

_(If you're familiar with the popular blogging platform, WordPress, you might also be familiar with the built-in "Permalinks Settings" that WordPress offers. This feature attempts to replicate and improve on that functionality.)_

**Example**

Create a `pretty` preset for automatically formatting URLs, where the [file.stem](#file-path-properties) of a blog post is used as the folder name, followed by `/index.html`:

```js
const permalinks = new Permalinks();
permalinks.preset('pretty', 'blog/:slugify(title)/index.html');

console.log(permalinks.format(':pretty', 'foo/bar/baz.hbs', {title: 'Post One'}));
//=> 'blog/post-one/index.html'
console.log(permalinks.format(':pretty', 'foo/bar/qux.hbs', {title: 'Post Two'}));
//=> 'blog/post-two/index.html'
```

### Helpers

Helper functions can be used to resolve placeholders in permalink structures. For example:

```js
// register a helper function
permalinks.helper('foo', function() {
  // return the value to use to replace "foo"
  return this.file.stem;
});

const url = permalinks.format('/:foo', {path: 'about.hbs'});
console.log(url);
//=> '/about'
```

<details>
<summary><strong>Helper example</strong></summary>

Use a `date` helper to dynamically generate paths based on the date defined in YAML front matter of a file.

```js
var moment = require('moment');
var Permalinks = require('permalinks');
var permalinks = new Permalinks();

var file = {
  path: 'src/about.hbs',
  data: {
    date: '2017-02-14'
  }
};

// "file.data" is merged onto "this.context" 
permalinks.helper('date', function(format) {
  return moment(this.context.date).format(format || 'YYYY/MM/DD');
});

console.log(permalinks.format(':date/:stem/index.html', file));
//=> '2017/02/14/about/index.html'
```

Helpers can also take arguments:

```js
console.log(permalinks.format(':date("YYYY")/:stem/index.html', file));
//=> '2017/about/index.html'
```
</details>

See the [helper unit tests](test) for more examples.

#### file helper

A special built-in `file` helper is called on every file and then removed from the context before rendering.

```js
permalinks.helper('file', function(file, data, locals) {
  // do stuff with file, data and locals
});
```

You can override this helper to modify the context or set properties on files before generating permalinks.

<details>
<summary><strong>`file` helper example</strong></summary>

Use the `file` helper to increment a value for pagination or something similar:

```js
var file = new File({path: 'foo/bar/baz.hbs'});
var permalinks = new Permalinks();
var count = 0;

permalinks.helper('file', function(file, data, locals) {
  data.num = ++count;
});

console.log(permalinks.format(':num-:basename', file));
//=> '1-baz.hbs'
console.log(permalinks.format(':num-:basename', file));
//=> '2-baz.hbs'
console.log(permalinks.format(':num-:basename', file));
//=> '3-baz.hbs'
console.log(count);
//=> 3
```

</details>

### Additional resources

Here is some reading material if you're interested in learning more about permalinks.

* [The ideal WordPress SEO URL structure](https://yoast.com/wordpress-seo-url-permalink/)
* [A Guide To WordPress Permalinks, And Why You Should Never Use The Default Settings](https://www.elegantthemes.com/blog/tips-tricks/wordpress-permalinks)

## About

<details>
<summary><strong>Contributing</strong></summary>

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](../../issues/new).

</details>

<details>
<summary><strong>Running Tests</strong></summary>

Running and reviewing unit tests is a great way to get familiarized with a library and its API. You can install dependencies and run tests with the following command:

```sh
$ npm install && npm test
```

</details>
<details>
<summary><strong>Building docs</strong></summary>

_(This project's readme.md is generated by [verb](https://github.com/verbose/verb-generate-readme), please don't edit the readme directly. Any changes to the readme must be made in the [.verb.md](.verb.md) readme template.)_

To generate the readme, run the following command:

```sh
$ npm install -g verbose/verb#dev verb-generate-readme && verb
```

</details>

### Related projects

You might also be interested in these projects:

* [handlebars](https://www.npmjs.com/package/handlebars): Handlebars provides the power necessary to let you build semantic templates effectively with no frustration | [homepage](http://www.handlebarsjs.com/ "Handlebars provides the power necessary to let you build semantic templates effectively with no frustration")
* [parse-filepath](https://www.npmjs.com/package/parse-filepath): Pollyfill for node.js `path.parse`, parses a filepath into an object. | [homepage](https://github.com/jonschlinkert/parse-filepath "Pollyfill for node.js `path.parse`, parses a filepath into an object.")
* [vinyl](https://www.npmjs.com/package/vinyl): Virtual file format. | [homepage](https://github.com/gulpjs/vinyl#readme "Virtual file format.")

### Author

**Jon Schlinkert**

* [linkedin/in/jonschlinkert](https://linkedin.com/in/jonschlinkert)
* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](https://twitter.com/jonschlinkert)

### License

Copyright © 2018, [Jon Schlinkert](https://github.com/jonschlinkert).
Released under the [MIT License](LICENSE).

***

_This file was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme), v0.6.0, on January 11, 2018._