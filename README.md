Vandoc
------

> Van Helsing hunts for vampires, Van Doc hunts for your docs.

Environment and language-agnostic Node tool to get documentation out of files as reusable data.

[See supported right now languages](#supported-languages).

_So far it is only initial draft._

## Purpose

Vandoc intended to be used in environment, when you need to integrate documentation tools with already existing structure (static site generation, server-side rendering etc).

Most popular tools doesn't fit that purpose well, since usually they comes with already built-in documentation pages generators, dev servers and enforcing some specific structure, rolling out their own syntax. All this makes them quite difficult adapt for projects needs, impossible to integrate into existing environment without undesired parts and features and forces to support not only website's specific templating, layout, styles, but also documentation tools' one.

Vandoc takes another approach and does not provide any means for endpoint documentation generation. It is quite low-level tool and only scans your files for documentation (in most cases it is specific documentation comments blocks) and returns it as nicely formatted data, which can be used by any tool that can understand typical object data to output documentation pages whenever your designers wants and how your system can print them.

Another issue with current tools is that if you have mixed codebase, you will be forced to use different tools to generate documentation. Got CSS and JavaScript? To bad for you, since you will need to generate docs with one tools (and its whole ecosystem) for one language, and with completely another tool for another. Not to mention twice payload on already stated issue with integration in current environment above.

Vandoc doesn't make any assumptions about your codebase. In fact, out of box it even doesn't know how to deal with any language. By adding Vandoc drivers or writing your own, allowing Vandoc to deal with any kind of language and its specific documentation ways, be it comment blocks or whatever, parse them and return appropriate data. Vandoc doesn't care what driver makes internally while it returns data in enforced by Vandoc uniform data structure.

This takes us to another point. Vandoc tries to provide uniform interface for all languages, and thus it implies that all drivers should return data with specific, uniform [Vandoc data structure](#vandoc-data-structure), which is easy to understand and easy to use. This make documentation generation very similar for most languages.

Vandoc data structure is quite broad and covers most needs of currently popular languages. Its based on [JSDoc](http://usejsdoc.org/) (and, thus, on Clojure Compiler) specifications with some minor additions, which allows effectively to describe almost any features in any language.

Note, that it doesn't restrict your comments or documentation format. It can be of any kind, but you just have to ensure that driver you use or write can parse them and that result of those documentations can then be dumped into Vandoc data structure.

All this makes Vandoc ideal tools for cases, when you need to add documentation to your already existing project and make use of tools for pages output that already exist in project, or as low-level toolkit to build more common styleguide generation system with pluggable and wide languages syntaxes support.

## How to use

Install Vandoc

```shell
npm install vandoc
```

Install drivers

```shell
npm install vandoc-js vandoc-nunjucks vandoc-scss
```

Scan your files and print results into `vandoc.json`

```shell
vandoc '**/*.{njk,scss,js}' --cwd 'source/' --output 'vandoc.json'
```

Vandoc automatically will try load available drivers and map extensions to drivers in which provided list of default extensions.

Specify extensions mapping if your are using non-standard extensions

```shell
vandoc '**/*.{html,css,js}' --cwd 'source/' --extensions 'html:nunjucks,css:stylus' --output 'vandoc.json'
```

Or use API:

```js
  import vandoc from `vandoc`
  import fs from 'fs'

  vandoc('**/*.{html,styles,js}', {
    cwd: 'source/'
    extensions: {
      html: 'nunjucks',
      styles: 'scss'
    }
  })
    .hunt()
    .then((result) => fs.writeFile('vandoc.json', JSON.stringify(result), 'utf8'))
    .catch((e) => throw new Error(e))
```

Or you can store data directly in memory for further use:

```js
  import vandoc from `vandoc`

  const vandocData = async () => await vandoc('**/*.{html,styles,js}').hunt()

  console.log(vandocData)
```

The result will be data with specific [Vandoc data structure](#vandoc-data-structure).

Based on Vandoc data any tool can generate documentation however it likes. Use [Vanrose](#vanrose) to make work with data easier.

Simple example in Nunjucks

```jinja
{% for v in vanrose(vandoc).language('nunjucks') %}
<article>
  <h1>{{ v.name }}</h1>
  <ul>
    {% for p in v.params %}
    <li><code>{{ p.name }}<{{ p.type }}></code></li>
    {% endfor %}
  </ul>
  <pre><code>{{ v.snippet }}</code></pre>
</article>
{% endfor %}
```

## Supported languages

Out of box Vandoc is language agnostic and it doesn't know how to deal with your codebase. To make it work, you need to install drivers for languages from which you intend to extract documentation comment blocks or write your own driver.

Currently available drivers:

* [JavaScript](#t)
* [Nunjucks](#t)
* [Sass (SCSS)](#t)

## Vandoc data structure

Vandoc data is core of whole library. It ensures that collected by drivers data is nicely formatted and easy to work with. It isn't just an AST, but a full-purpose object with list of all collected blocks and meaningful properties for each block.

Despite it fearsome name, Vandoc data is just an plain old Object, which can be exported into any format and later used by developer for generating his own documentation pages, whether be it statically generated website with Nunjucks, or some SPA.

Vandoc gathers all [drivers data](#drivers-data-sturcture) in following Vandoc data object:

```js
const VandocData = t.dict(Namepath, VandocDriverData)
```

Where `Namepath` is generated for each entry by Vandoc path by processing all `module`, `memberof`, `namespace`, and `filepath` properties of driver results.

Example of such `Namepath`

```
nunjucks:moduleName.nameSpaceName.functionName
```

As in JSDocs, whenever driver and language supports it, Vandoc can generate paths for accessing inner methods:

```
// MyConstructor
// MyConstructor#instanceMember
// MyConstructor.staticMember
// MyConstructor~innerMember
```

`Namepath` also injected by Vandoc into each entry under `namepath` property for easier access.

Namepath serves as an sort of `id`, which allows to access or link specific blocks whenever needed.

### Drivers data structure

Idea of returned by drivers data structure mostly based on JSDocs. It tries to be broad to support vast majority of currently popular languages which might require documentation, including even templating languages and CSS.

Note, that structure is strict and enforced by Vandoc, but many values might be optional and depends on language.

For each comment block documentation occurrence, driver returns data with following structure:

```js
const VandocDriverData = t.struct({
  filepath: t.String,
  language: t.String,
  position: t.Number // in which order appears in source code
  type: t.String,
  name: t.String, // function name
  alias: t.String, // displayed name of function
  description: t.String,
  version: t.String,
  author: t.String,
  copyright: t.String,
  deprecated: t.union([t.String, t.Boolean]),
  ignore: t.Boolean,
  see: t.struct({ // refer to other namepath or description
    link: t.maybe(t.String), // namepath link or any other link, generates anchor
    description: t.String
  })
  tutorial: t.String,
  todo: t.list(t.String),
  section: t.String, // needed? Seems to be covered by namespace
  namespace: t.Struct({
    type: t.String,
    name: t.String // defaults to function name, mandatory for selectors
  })
  memberof: t.String, // namespace it belongs to
  module: t.Struct({
    type: t.String,
    name: t.String // takes filename by default, inner methods will be like filename/method
  })
  global: t.Boolean,
  tags: t.list(t.String),
  constant: t.union([t.String, t.Boolean]),
  variations: t.list(t.struct({
    snippet: t.String,
    description: t.String
  }))
  params: t.list(t.struct({
    types: t.list(t.String),
    name: t.String,
    optional: t.Boolean,
    default: t.String,
    description: t.String
  }))
  requires: t.list(t.String) // module:name it requires
  returns: t.struct({
    types: t.list(t.String),
    description: t.String
  })
  throws: t.list(t.struct({
    types: t.list(t.String),
    description: t.String
  }))
  examples: t.list(t.String),
  snippet: t.String
}, { name: 'VandocDriverData', strict: true })
```

## API

### Vandoc

* CLI and API for input
* Accepts filepaths globs and options
* Loads installed drivers. No additional configuration required, Vandoc will try to discover them.
* Tries to apply drivers based on filepath extensions by default, if driver has defined default extensions, otherwise requires mapping
* Runs drivers against filelepaths glob
* Receives results from drivers in form of data with [Vandoc drivers data structure](#drivers-data-structure) (not just AST)
* Gathers them into Object
* Writes Object to file, to memory or console.logs, depending on settings for further use

`vandoc(filepathGlob<string>, options<object>?)`

* `filepathGlob` is a valid glob for files
* `options` is optional configuration, with following properties:

   * `cwd` — filepath, which will be used as current working directory, to which will be relative `filepathGlob`

      `cwd` will also be substracted from all `Namepath` resolutions

      `extensions` — special mapping for cases, when you use non-standard or generic extensions, thus you will need to map them to specific drivers.

      Example:

      ```js
      extensions: {
        html: 'nunjucks',
        styles: 'scss'
      }
      ```

   * `drivers` — allows to define drivers options.

      Example:

      ```js
      drivers: {
        html: {
          options: {
            ignoreBlocksWith: /^<-- IGNORE ME/
          }
        },
      }
      ```

#### Methods

* `.hunt()` — makes Vandoc to run.

   Returns `Promise` with prepared Vandoc data as result.

### Vandoc Driver:

* Parses glob of provided by Vandoc files
* Extracts only doc comments based on internal or configurated settings (`'/** */`), ignores rest
* Extracts code snippets associate with comments
* Parses comments into AST
* Optionaly parses snippets to improve result data accuracy
* Based on AST and code snippets forms Objects for each comment block with [Vandoc data structure](#drivers-data-structure)
* Returns to Vandoc object

Very poor example of driver:

```js
module.exports = (vandoc) =>
  vandoc.registerDriver('language', ['listOfExtensions']) => {
    const options = vandoc.options({
      some: 'defaultValues'  
    })

    ...

    return run
  }
```

## Vanrose

Supposed to make work with Vandoc data easier.

Head to [Vanrose repository](https://github.com/ArmorDarks/Vanrose) for details.

## Helpful:

* https://github.com/eslint/doctrine
* https://github.com/jdalton/docdown
* http://usejsdoc.org/

## Possible names:

* Ico
* Icon
* Icona
* Iconr
* View
* Vango — ref Vanga, `farsees your docs`
* Vandoc — Van Helsing hunts for Vampires, Van Doc hunts for Docs.
* Vandocing
* VanHeldoc

## Possible logo

Large white open eye on pointing down pale-pink triangle
