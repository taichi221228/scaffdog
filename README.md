![scaffdog](./docs/assets/repo-banner.png)

<p align="center">
  scaffdog is Markdown driven scaffolding tool.
</p>

<p align="center">
  <a href="https://github.com/cats-oss/scaffdog/actions?workflow=CI"><img src="https://img.shields.io/github/workflow/status/cats-oss/scaffdog/CI?logo=github&style=flat-square" alt="GitHub Workflow Status" /></a>
  <a href="https://www.npmjs.com/package/scaffdog"><img src="https://img.shields.io/npm/v/scaffdog?style=flat-square" alt="npm" /></a>
  <a href="./LICENSE"><img src="https://img.shields.io/github/license/cats-oss/scaffdog?label=license&style=flat-square" alt="MIT LICENSE" /></a>
</p>

# scaffdog

Multiple files can be output in a document, and flexible scaffolding is possible with a simple but powerful template syntax :dog2:

## Table of Contents

- :dog: [Features](#features)
- :feet: [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Quick Start](#quick-start)
- :fire: [Migration](#migration)
- :wrench: [Configuration](#configuration)
  - [Simple Configuration](#simple-configuration)
  - [Global Variables](#global-variables)
  - [Custom Helpers](#custom-helpers)
- :poodle: [Commands](#commands)
  - [`scaffdog init`](#scaffdog-init)
  - [`scaffdog generate`](#scaffdog-generate)
  - [`scaffdog template`](#scaffdog-template)
  - [`scaffdog list`](#scaffdog-list)
- :memo: [Templates](#templates)
  - [Structure](#structure)
  - [Syntax](#syntax)
  - [Pipe chain](#pipe-chain)
  - [Attributes](#attributes)
  - [Variables](#variables)
  - [Helpers](#helpers)
- :blue_book: [FAQ](#faq)
- :dog2: [Contributing](#contributing)
  - [Development scripts](#development-scripts)
- :gem: [License](#license)

## Features

- :pencil: **Markdown driven**
  - You can define a template with `<h1>` and code block.
  - It will be a Documetable template !
  - Define meta information with extended syntax using [Front Matter](https://jekyllrb.com/docs/front-matter/).
- :hammer_and_pick: **Simple & Powerful template syntax**
  - A simple syntax similar to mustache.js. (`{{ inputs.value }}`)
  - Function execution similar to pipe syntax. (`{{ inputs.value | fn }}`)
  - Provide minimum necessary functions in built-in.
- :rocket: **Ready to use**
  - You can quickly start using `$ scaffdog init`.

## Getting Started

### Installation

`scaffdog` can be installed globally, but we recommend installing it locally on the project.

```bash
$ npm install --save-dev scaffdog
```

### Quick Start

In the following tutorial you can start using `scaffdog` immediately !

#### Setup

By default, it stores the template file in the directory `.scaffdog`.

Creating directories and initial templates can be done with the `init` subcommand.

```bash
$ npx scaffdog init
```

This will create a file called `.scaffdog/hello.md`. Let's scaffold immediately using the `hello` template!

```bash
$ npx scaffdog generate hello

? Please select the output destination directory. .
? Please enter any text. pretty-dog

✨ Completed scaffolding !

    ✔ pretty-dog.md

```

Congratulations :tada: The first file was created.

```bash
$ cat pretty-dog.md

Let's create a template with reference to the document!
https://github.com/cats-oss/scaffdog/#templates
```

After this, please customize the `hello.md` template with reference to the document and try to see if you can do what you expect. :+1:

## Migration

There are important changes in the major update.

See [Migration Guide](../../MIGRATION.md).

## Configuration

scaffdog uses the object exported in `.scaffdog/config.js` as the configuration.

### Simple Configuration

In the `files` field, specify the path pattern of the document used by scaffdog. The `files` field is required.

```javascript
module.exports = {
  files: ['./*'],
};
```

### Global Variables

You can define the global variables available in the template in the `variables` field.

```javascript
module.exports = {
  files: ['./*'],
  variables: {
    key: 'value',
  },
};
```

### Custom Helpers

You can define the custom helpers available in the template in the `helpers` field.

```javascript
module.exports = {
  files: ['./*'],
  helpers: [
    // Using Key-Value
    {
      trim: (context, value) => value.trim(),
    },

    // Using Helper Registry
    (registry) => {
      registry.set('padstart', (context, value, str) =>
        value.padStart(size, str || ' '),
      );
      registry.set('padend', (context, value, str) =>
        value.padEnd(size, str || ' '),
      );
    },
  ],
};
```

The `context` passed to the first argument of the helper function has the following structure.

```typescript
type Variable =
  | string
  | {
      [key in string | number]: Variable;
    }
  | Variable[];

type VariableMap = Map<string, Variable>;

type Helper = (context: Context, ...args: any[]) => string;

type HelperMap = Map<string, Helper>;

type Context = {
  cwd: string;
  variables: VariableMap;
  helpers: HelperMap;
};
```

## Commands

### `scaffdog init`

Prepare for using scaffdog. By default it creates a `.scaffdog` directory and creates a simple template.

```bash
USAGE
  $ scaffdog init

OPTIONS
  -d, --templateDir=templateDir  [default: .scaffdog] Directory where to load scaffdog templates from.
  -h, --help                     show CLI help
```

### `scaffdog generate`

Scaffold using the specified template. If you do not specify the template name and execute it, interactively select the template.

```bash
USAGE
  $ scaffdog generate [TEMPLATENAME]

OPTIONS
  -d, --templateDir=templateDir  [default: .scaffdog] Directory where to load scaffdog templates from.
  -h, --help                     show CLI help
  -n, --dryRun                   Output the result to stdout.
```

### `scaffdog template`

Creating a template with the specified name.

```bash
USAGE
  $ scaffdog template NAME

ARGUMENTS
  NAME  Template name.

OPTIONS
  -d, --templateDir=templateDir  [default: .scaffdog] Directory where to load scaffdog templates from.
  -h, --help                     show CLI help
```

### `scaffdog list`

Print a list of available templates.

```bash
USAGE
  $ scaffdog list

OPTIONS
  -d, --templateDir=templateDir  [default: .scaffdog] Directory where to load scaffdog templates from.
  -h, --help                     show CLI help
```

## Templates

### Structure

Template documents are defined with `<h1>` and code blocks. `<h1>` is interpreted as the file name and code blocks as the template body.

Meta information is defined using [Front Matter](https://jekyllrb.com/docs/front-matter/).

````markdown
---
name: 'utility'
root: 'src/utils'
output: '**/*'
ignore: []
questions:
  name: 'Please enter a filename.'
---

# `{{ inputs.name }}.js`

```javascript
export const {{ inputs.name | camel }} = () => true;
```

# `__tests__/{{ inputs.name }}.test.js`

```javascript
import { {{ inputs.name | camel }} } from '../{{ inputs.name }}';

describe('{{ inputs.name | camel }}', () => {
  test('__TODO__', () => {
    expect({{ inputs.name | camel }}()).toBe(true);
  });
});
```
````

### Syntax

Between `{{` and `}}` is interpreted as a tag. Whitespace in tag contents is ignored.

#### Output of variable

The given variable name is output. See the [Variables](#variables) section for the variables that can be used.

```
{{ <identifier> }}
```

Example:

```
{{ inputs.value }}
```

#### Whitespaces

Trim the expression to be expanded and the space and line feed around it.  
Use `{{-` to trim before the tag, and `-}}` to trim after the tag.

```
{{- <expression> -}}
```

Example:

```
before {{- "text" }} after
before {{ "text" -}} after
before {{- "text" -}} after
```

Output:

```
beforetext after
before textafter
beforetextafter
```

#### Comment out

You can use comment out to keep the template readable. Of course, it is not deployed as a template.

```
{{ /* a comment */ }}
```

#### Call helper

Execute the helper with the specified name. Arguments are separated by whitespace. See the [Helpers](#helpers) section for the helpers that can be used.

```
{{ <helper> <argument> ... }}
```

Example:

```
{{ relative "../" }}
```

### Pipe chain

You can chain output values or helper results with pipes. This is a helper similar to shell.

```
{{ <identifier> | <helper> }}
{{ <identifier> | <helper> <argument> ... }}
```

Example:

```
{{ inputs.value | upper }}
{{ inputs.value | replace "$.ts" ".js" | pascal }}
{{ basename | replace extname ".js" | pascal }}
```

### Attributes

List of attributes that can be specified with Front Matter.

| key         | required | type       | description                                                                                                                       |
| :---------- | :------- | :--------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| `name`      | `true`   | `string`   | Name of template.                                                                                                                 |
| `root`      | `true`   | `string`   | The directory as the starting point of the output destination.                                                                    |
| `output`    | `true`   | `string`   | Directory starting from `root` and being a destination candidate. You can use glob syntax. (see [globby][glob-patterns] document) |
| `ignore`    | `false`  | `string[]` | Directory to exclude from candidate output destination. You can use glob syntax. (see [globby][glob-patterns] document)           |
| `questions` | `false`  | `object`   | Message to display when accepting input.                                                                                          |

[glob-patterns]: https://github.com/sindresorhus/globby#globbing-patterns

`questions` defines the question to be used in prompt with key-value. The values you answer can be used in a variable called `inputs`. (e.g. `inputs.key1`, `inputs.value`)

```
---
questions:
  # Shortest syntax, using `input` prompt.
  key1: 'Message'

  # Using `input` prompt.
  key2:
    message: 'Message'

  # Using `input` prompt, with default value.
  key3:
    message: 'Message'
    initial: 'Initial Value'

  # Using `list` prompt.
  key4:
    message: 'Message'
    choices: ['A', 'B', 'C']
---
```

### Variables

List of variables available in the template. You need to be aware that the file name and the variables available in the template body are different.

#### File name

| key      | description                              | example |
| :------- | :--------------------------------------- | :------ |
| `inputs` | The object value received at the prompt. |         |

#### Template body

| key             | description                                                      | example                       |
| :-------------- | :--------------------------------------------------------------- | :---------------------------- |
| `inputs`        | The object value received at the prompt.                         |                               |
| `output.name`   | The name of the output destination file excluding the extension. | `scaffdog`                    |
| `output.base`   | The name of the output destination file including the extension. | `scaffdog.js`                 |
| `output.ext`    | The destination file name extension.                             | `.js`                         |
| `output.dir`    | The destination directory name.                                  | `src`                         |
| `output.path`   | The path of the destination file.                                | `src/scaffdog.js`             |
| `output.abs`    | The absolute path of the destination file.                       | `/path/to/src/scaffdog.js`    |
| `document.name` | The document name.                                               | `hello`                       |
| `document.path` | The path of the document file.                                   | `/path/to/.scaffdog/hello.md` |

### Helpers

When invoked on a pipe, the previous processing result is passed to the first argument.

| name       | arguments                                               | description                                                                                                         |
| :--------- | :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------ |
| `camel`    | `[value: string]`                                       | Conversion to a camel case.                                                                                         |
| `snake`    | `[value: string]`                                       | Conversion to a snake case.                                                                                         |
| `pascal`   | `[value: string]`                                       | Conversion to a pascal case.                                                                                        |
| `kebab`    | `[value: string]`                                       | Conversion to a kebab case.                                                                                         |
| `constant` | `[value: string]`                                       | Conversion to a constant case.                                                                                      |
| `upper`    | `[value: string]`                                       | Conversion to a upper case.                                                                                         |
| `lower`    | `[value: string]`                                       | Conversion to a lower case.                                                                                         |
| `replace`  | `[value: string, pattern: string, replacement: string]` | Replace `pattern` with`replacement`. `pattern` is specified as a string, but it is treated as a regular expression. |
| `trim`     | `[value: string]`                                       | Alias for `String.prototype.trim`.                                                                                  |
| `ltrim`    | `[value: string]`                                       | Alias for `String.prototype.trimStart`.                                                                             |
| `rtrim`    | `[value: string]`                                       | Alias for `String.prototype.trimEnd`.                                                                               |
| `eval`     | `[code: string]`                                        | Executes the specified code and returns the result.                                                                 |
| `date`     | `[format?: string]`                                     | See the [dayjs](https://day.js.org/docs/en/display/format) documentation for format details.                        |
| `noop`     | `[]`                                                    | Returns an empty string.                                                                                            |
| `define`   | `[value: string, key: string]`                          | Defines a local variable in the template scope.                                                                     |
| `relative` | `[path: string]`                                        | Convert the path from the template file to the path from the destination file.                                      |
| `read`     | `[path: string]`                                        | Read the specified file. The contents of the loaded file are also expanded as a template.                           |

## FAQ

<details>
  <summary>Is it possible to define variables in the template?</summary>

scaffdog does not support variable definitions syntactically.

However, it is possible to define variables in combination with helper functions :tada:

See the following example.

```markdown
{{- output.path | replace "^src" "" | define "out" | noop -}}
{{ output.path }}
{{ out }}
{{ out | replace "/" "-" }}
```

The output is as follows:

```markdown
src/utils/input.js
utils/input.js
utils-input.js
```

Defined a variable called `out` which is a processed version of` output.path`. This is made up of some magic.

1. Save the calculation result to `key` with `define` helper function.
1. Convert to empty string with `noop` helper function so that the result is not output.
1. The variable definition tag trims whitespaces. (use `{{-` and `-}}`)

</details>

## Contributing

We are always welcoming your contribution :clap:

1. Fork (https://github.com/cats-oss/scaffdog) :tada:
1. Create a feature branch :coffee:
1. Run test suite with the `$ yarn test` command and confirm that it passes :zap:
1. Commit your changes :memo:
1. Rebase your local changes against the `master` branch :bulb:
1. Create new Pull Request :love_letter:

Bugs, feature requests and comments are more than welcome in the [issues](https://github.com/cats-oss/scaffdog/issues).

### Development scripts

#### `yarn test`

Run Unit test with [ava](https://github.com/avajs/ava).

```bash
$ yarn test
```

#### `yarn lint`

Run lint with [ESLint](https://github.com/eslint/eslint).

```bash
$ yarn lint
```

#### `yarn format`

Run formatting with [ESLint](https://github.com/eslint/eslint) (`--fix`) and [Prettier](https://github.com/prettier/prettier).

```bash
$ yarn format
```

## License

[MIT © Cyberagent, Inc](./LICENSE)

![thank you for reading!](./docs/assets/repo-footer.png)
