# marked

> A full-featured markdown parser and compiler, written in javascript. Built
> for speed.

[![NPM version](https://badge.fury.io/js/marked.png)][badge]

## Install

``` bash
npm install marked --save
```

## Usage

Minimal usage:

```js
console.log(marked('I am using __markdown__.'));
// Outputs: <p>I am using <strong>markdown</strong>.</p>
```

Example using all options:

```js
marked.setOptions({
  gfm: true,
  tables: true,
  breaks: false,
  pedantic: false,
  sanitize: true,
  smartLists: true,
  smartypants: false,
});

// Using async version of marked
marked('I am using __markdown__.', function (err, content) {
  if (err) throw err;
  console.log(content);
});
```

## marked(markdownString, [options], [callback])

### markdownString

Type: `String`

String of markdown source to be compiled.

### options

Type: `Object`

Hash of options. Can also be set using the `marked.setOptions` method as seen
above.

### callback

Type: `Function`

Function called when the `markdownString` has been fully parsed when using
async highlighting. If the `options` argument is omitted, this can be used as
the second argument as seen above:

## Options

### gfm

Type: `Boolean`
Default: `true`

Enable [GitHub flavored markdown][gfm].

### tables

Type: `Boolean`
Default: `true`

Enable GFM [tables][tables].
This option requires the `gfm` option to be true.

### breaks

Type: `Boolean`
Default: `false`

Enable GFM [line breaks][breaks].
This option requires the `gfm` option to be true.

### pedantic

Type: `Boolean`
Default: `false`

Conform to obscure parts of `markdown.pl` as much as possible. Don't fix any of
the original markdown bugs or poor behavior.

### sanitize

Type: `Boolean`
Default: `false`

Sanitize the output. Ignore any HTML that has been input.

### smartLists

Type: `Boolean`
Default: `true`

Use smarter list behavior than the original markdown. May eventually be
default with the old behavior moved into `pedantic`.

### smartypants

Type: `Boolean`
Default: `false`

Use "smart" typograhic punctuation for things like quotes and dashes.

### renderer

Type: `Renderer`
Default: `new Renderer()`

A renderer instance for rendering ast to html. Learn more on the Renderer
section.

## Renderer

Renderer is a the new way for rendering tokens to html. Here is a simple
example:

```javascript
var r = new marked.Renderer()
r.blockcode = function(code, lang) {
  return highlight(lang, code).value;
}

console.log(marked(text, {renderer: r}))
```

You can control anything you want.

### Block Level

- code(code, language)
- blockquote(quote)
- html(html)
- heading(text, level)
- hr()
- list(body, ordered)
- listitem(text)
- paragraph(text)
- table(header, body)
- tablerow(content)
- tablecell(content, flags)

`flags` is an object like this:

```
{
    header: true,
    align: 'center'
}
```

### Span Level

- strong(text)
- em(text)
- codespan(code)
- br()
- del(text)
- link(href, title, text)
- image(href, title, text)

## Access to lexer and parser

You also have direct access to the lexer and parser if you so desire.

``` js
var tokens = marked.lexer(text, options);
console.log(marked.parser(tokens));
```

``` js
var lexer = new marked.Lexer(options);
var tokens = lexer.lex(text);
console.log(tokens);
console.log(lexer.rules);
```

## CLI

``` bash
$ marked -o hello.html
hello world
^D
$ cat hello.html
<p>hello world</p>
```

## Benchmarks

node v0.4.x

``` bash
$ node test --bench
marked completed in 12071ms.
showdown (reuse converter) completed in 27387ms.
showdown (new converter) completed in 75617ms.
markdown-js completed in 70069ms.
```

node v0.6.x

``` bash
$ node test --bench
marked completed in 6448ms.
marked (gfm) completed in 7357ms.
marked (pedantic) completed in 6092ms.
discount completed in 7314ms.
showdown (reuse converter) completed in 16018ms.
showdown (new converter) completed in 18234ms.
markdown-js completed in 24270ms.
```

__Marked is now faster than Discount, which is written in C.__

For those feeling skeptical: These benchmarks run the entire markdown test suite
1000 times. The test suite tests every feature. It doesn't cater to specific
aspects.

node v0.8.x

``` bash
$ node test --bench
marked completed in 3411ms.
marked (gfm) completed in 3727ms.
marked (pedantic) completed in 3201ms.
robotskirt completed in 808ms.
showdown (reuse converter) completed in 11954ms.
showdown (new converter) completed in 17774ms.
markdown-js completed in 17191ms.
```

## Another Javascript Markdown Parser

The point of marked was to create a markdown compiler where it was possible to
frequently parse huge chunks of markdown without having to worry about
caching the compiled output somehow...or blocking for an unnecesarily long time.

marked is very concise and still implements all markdown features. It is also
now fully compatible with the client-side.

marked more or less passes the official markdown test suite in its
entirety. This is important because a surprising number of markdown compilers
cannot pass more than a few tests. It was very difficult to get marked as
compliant as it is. It could have cut corners in several areas for the sake
of performance, but did not in order to be exactly what you expect in terms
of a markdown rendering. In fact, this is why marked could be considered at a
disadvantage in the benchmarks above.

Along with implementing every markdown feature, marked also implements [GFM
features][gfmf].

### High level

You can customize the result with a customized renderer.

``` js
var renderer = new marked.Renderer()

renderer.header = function(text, level) {
  return '<div class="h-' + level + '">' + text + '</div>'
}

var parse = function(src, options) {
  options = options || {};
  return marked.parser(marked.lexer(src, options), options, renderer);
}

console.log(parse('# h1'))
```

The renderer API:

```
code: function(code, lang)
blockquote: function(text)
html: function(html)

heading: function(text, level)
paragraph: function(text)

hr: function()

list: function(contents, isOrdered)
listitem: function(text)

table: function(header, body)
tablerow: function(content)
tablecell: function(text, flags)
// flags: {header: false, align: 'center'}
```

### Pro level

You also have direct access to the lexer and parser if you so desire.

``` js
var tokens = marked.lexer(text, options);
console.log(marked.parser(tokens));
```

``` js
var lexer = new marked.Lexer(options);
var tokens = lexer.lex(text);
console.log(tokens);
console.log(lexer.rules);
```

``` bash
$ node
> require('marked').lexer('> i am using marked.')
[ { type: 'blockquote_start' },
  { type: 'paragraph',
    text: 'i am using marked.' },
  { type: 'blockquote_end' },
  links: {} ]
```

## Running Tests & Contributing

If you want to submit a pull request, make sure your changes pass the test
suite. If you're adding a new feature, be sure to add your own test.

The marked test suite is set up slightly strangely: `test/new` is for all tests
that are not part of the original markdown.pl test suite (this is where your
test should go if you make one). `test/original` is only for the original
markdown.pl tests. `test/tests` houses both types of tests after they have been
combined and moved/generated by running `node test --fix` or `marked --test
--fix`.

In other words, if you have a test to add, add it to `test/new/` and then
regenerate the tests with `node test --fix`. Commit the result. If your test
uses a certain feature, for example, maybe it assumes GFM is *not* enabled, you
can add `.nogfm` to the filename. So, `my-test.text` becomes
`my-test.nogfm.text`. You can do this with any marked option. Say you want
line breaks and smartypants enabled, your filename should be:
`my-test.breaks.smartypants.text`.

To run the tests:

``` bash
cd marked/
node test
```

### Contribution and License Agreement

If you contribute code to marked, you are implicitly allowing your code to be
distributed under the MIT license. You are also implicitly verifying that all
code is your original work. `</legalese>`

## License

Copyright (c) 2011-2013, Christopher Jeffrey. (MIT License)

See LICENSE for more info.

[gfm]: https://help.github.com/articles/github-flavored-markdown
[gfmf]: http://github.github.com/github-flavored-markdown/
[pygmentize]: https://github.com/rvagg/node-pygmentize-bundled
[highlight]: https://github.com/isagalaev/highlight.js
[badge]: http://badge.fury.io/js/marked
[tables]: https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#wiki-tables
[breaks]: https://help.github.com/articles/github-flavored-markdown#newlines
