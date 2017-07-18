Vandoc (concept)
----------------

> Van Helsing hunts for vampires, Van Doc hunts for your docs.

Environment and language-agnostic Node tool to get documentation out of codebase files as reusable data.

[See supported right now languages](#supported-languages).

What it does:

* Provides a unified input and output interface to work with your codebase documentation independently of languages syntax or documentation methods.
* Orchestrate and runs your whole codebase against Vandoc drivers.
* Forms and returns strictly structured object with all your codebase documentation data, for all languages.

What it does not:

* Does not make any assumptions about your languages syntax. Vandoc drivers do.
* Does not make any assumptions about how you document your code. Vandoc drivers do.
* Does not know how to parse and extract documentation from your codebase. Vandoc drives do.
* Does not generate documentation pages or styleguide. It should be done based on received Vandoc data, or by using Vandoc-driven tools.
* Doest not do anything else.

## The Reasons

Vandoc dedicated to creating single, unified interface for working with any language and any documentation method which will return only pure reusable data, and nothing more.

Its purpose is to be easily integrable documentation tool into existing environments, with already defined structure and means of views rendering, be it static site generation, React-driven SPA application, or anything else. Since the result of Vandoc work is pure data, it can be used anywhere and anyhow.

Most popular tools does not fit that purpose well. They come with already built-in documentation pages generators, dev servers, enforcing specific structure and relaying on its own documentation methods. They are difficult to adapt for projects needs or gracefully integrate into existing environment. Since usually they end up living for their own, they require additional maintenance, completely detached from already existing project specific templates, layouts, styles.

Vandoc takes another approach and does not provide any means for endpoint documentation generation. It is low-level tool and only scans your files for documentation (in most cases it is specific documentation comments blocks) and returns it as nicely formatted data, which can be used later by existing project views to output documentation pages whenever your designers wants and how your system can.

Another issue with current tools is that mixed codebase requires different tools to generate documentation. Got CSS and JavaScript? You will need to generate docs with one tools (and its whole ecosystem) for one language, and with completely another tool for another. Not to mention increased payload on already stated issue with integration in current environment. Beside, such wide detachment of documentation tools from each other and current environment makes it really hard to express true relationships between elements from different ecosystems, like CSS and React components.

Vandoc does not make any assumptions nor about your codebase languages, nor about documentation methods those languages use. In fact, out of box Vandoc doesn't know how to deal with any language at all. But Vandoc drivers do. Drivers makes Vandoc understand any kind of language and its specific documentation ways, they are fully responsible for parsing and returning appropriate data. Since documentation tool no longer directly attached to your language or its documentation approach, be it comment blocks or some static typing system like Flow or TypeScript, single instance of Vandoc applicable almost for any language and documentation method flavour is just a matter of driver choice.

This takes us to another point. Vandoc implies standardization and provides uniform interface for all languages and documentation methods. No matter what language, what documentation methods it use, what drivers do internally to extract it, the result should be data with uniform, the only accepted by Vandoc structure.

[Vandoc data structure](#vandoc-data-structure) is broad and designed to describe effectively any popular language entities, be easy to understand and easy to use when generating your own documentation and styleguides. It is inspired by [JSDoc](http://usejsdoc.org/) specifications with some required additions to fit other languages. Standardized output gives freedom to internal language's documentation methods and ways, which tied to used driver, but ensures that end user (developer) always receive same formatted structured data, which can be safely used for documentation generation.

All this to facilitate Vandoc purpose at being universal and adaptable documentation collection tool, ease integration into already existing projects, or being a low-level toolkit to build more common styleguide generation system with pluggable and wide languages syntaxes and documentation methods support.

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

Out of box Vandoc is language agnostic and does not know how to deal with your codebase. To make Vandoc recognize languages and docummentation methods they use, install appropriate drivers, or write your own driver.

Planned drivers:

* [JavaScript (JSDocs)](#placeholder)
* [Nunjucks (JSDocs)](#placeholder)
* [Sass (subset of JSDocs)](#placeholder)

## Vandoc data structure

Vandoc data is core of whole library. It ensures that collected by drivers data is nicely formatted and easy to work with. It isn't just an AST, but a full-purpose object with list of all collected entities and describing set of properties for each entity.

Despite it fearsome name, Vandoc data is just an plain old Object, which can be exported into any format and later used by developer for generating his own documentation pages, whether be it statically generated website with Nunjucks, or some SPA.

Vandoc gathers all [drivers data](#drivers-data-sturcture) in following Vandoc data object:

```js
const VandocData = t.dict(Namepath, VandocDriverData)
```

Where `Namepath` is generated for each entry by Vandoc path by processing all `language`, `module`, `memberof`, `namespace`, and `source.filepath` properties of driver results.

Example of such `Namepath`

```
nunjucks:moduleName/path.nameSpaceName.functionName
```

As in JSDocs, whenever driver and language supports it, Vandoc can generate paths for accessing inner methods:

```
// MyConstructor
// MyConstructor#instanceMember
// MyConstructor.staticMember
// MyConstructor~innerMember
```

`Namepath` also injected by Vandoc into each entry under `namepath` property for easier access.

Namepath serves as an sort of `id`, which allows to access or link specific blocks.

### Drivers data structure

Idea of returned by drivers data structure mostly based on JSDocs. It tries to be broad to support vast majority of popular languages which might require documentation, including even templating languages and CSS.

Structure data is strict and does not allow additional properties to ensure that developers always receive data in same format, but it is eligible for expansion in future Vandoc releases if some language can not express its entities without such additions.

For each entity with documentation occurrence, driver returns data with following structure:

```js
const VandocDriverData = t.struct({
  language: t.String,
  source: t.struct({
    snippet: t.String, // maybe redundant, since can be assembled from comment and code
    linestart: t.Number,
    lineend: t.Number
    position: t.Number // in which order appears in source code
    filepath: t.String,
    filename: t.String,
  }),
  comment: t.struct({
    snippet: t.String, // raw comment
    linestart: t.Number,
    lineend: t.Number
  }),
  code: t.struct({
    snippet: t.String, // raw code block, mostly useful for CSS rulesets
    body: t.String // body of the function (only for functions, mixins etc.)
    linestart: t.Number,
    lineend: t.Number
  }),
  type: t.String,
  name: t.String, // function name or selector
  alias: t.String, // displayed name of function
  description: t.String,
  version: t.String,
  author: t.String,
  copyright: t.String,
  deprecated: t.union([t.String, t.Boolean]),
  private: t.Boolean, // usually used to exlude from docs generation, unless `--private` flag provided
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
  modifiers: t.list(t.struct({ // should it make to Vandoc? KSS-specific thing
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
  // module:name it requires
  // If no language provided in the begining, it will assume requirement within current language
  // Otherwise with `language` can be referenced modules from any language
  // Also, probably, we can accept a filename for scope definition
  requires: t.list(t.String),
  usedBy: t.list(t.String), // back-reference entities, which requires it. Maybe should be part of Vanrose
  returns: t.struct({
    types: t.list(t.String),
    description: t.String
  })
  throws: t.list(t.struct({
    types: t.list(t.String),
    description: t.String
  }))
  examples: t.list(t.strcut({ // also serves as Markup from KSS
    language: t.String,
    caption: t.String,
    snippet: t.String
  }))
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
* Remains of CSSDoc http://webkrauts.de/artikel/2007/stylesheets-kommentieren-mit-cssdoc
* https://doclets.io/