# Get Started

Learning ESLint from scratch (with Typescript).

## Prerequisites

- Javascript/Typescript knowledge
- Node project

## Create node project

Create a blank directory folder

```bash
npm init -y
```

## Setup Typescript

### Installation

```bash
npm install -D typescript
```

### Configuration

```bash
npx tsc --init --outDir ./dist
```

## ESLint

By the end of this section, it will generate a `.eslintrc` (Javascript / YAML / JSON).

_We will pick JSON, but you can pick anything_

But on the future lessons (*in other branches*), we will pick `YAML`.

```bash
npm init @eslint/config
```

```bash
? How would you like to use ESLint? ...
  To check syntax only
> To check syntax and find problems
  To check syntax, find problems, and enforce code style

? What type of modules does your project use? ...
> JavaScript modules (import/export)
  CommonJS (require/exports)
  None of these

? Which framework does your project use? ...
  React
  Vue.js
> None of these

? Does your project use TypeScript? » No / Yes

? Where does your code run? ...  (Press <space> to select, <a> to toggle all, <i> to invert selection)
  Browser
√ Node

? What format do you want your config file to be in? ...
  JavaScript
  YAML
> JSON
```

**Overall**

```bash
√ How would you like to use ESLint? · problems
√ What type of modules does your project use? · esm
√ Which framework does your project use? · none
√ Does your project use TypeScript? · No / Yes
√ Where does your code run? · node
√ What format do you want your config file to be in? · JSON
Local ESLint installation not found.
The config that you've selected requires the following dependencies:

@typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest eslint@latest
√ Would you like to install them now? · No / Yes
? Which package manager do you want to use? ...
> npm
  yarn
  pnpm
```

```json
{
    "env": {
        "es2021": true,
        "node": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended"
    ],
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
        "ecmaVersion": "latest",
        "sourceType": "module"
    },
    "plugins": [
        "@typescript-eslint"
    ],
    "rules": {}
}
```

## Update package.json

```json
{
    "scripts": {
        "lint": "eslint {src,test}/**/*.ts",
        "lint:fix": "eslint {src,test}/**/*.ts --fix"
    }
}
```

## Create sample typescript file

`src/index.ts`

```ts
let foo = 'bar';
```

## Run lint

This command looks at the `.eslintrc` file and will execute eslint scan based on the configuration given.

In this case, we extended the plugin **recommended** for both `eslint` and `@typescript-eslint` which have default rules such as the `let` should be replaced `const`.

More rules here for `eslint`: https://eslint.org/docs/latest/rules/

```bash
npm run lint
```

**OUTPUT**

```bash
~\learn-eslint\src\index.ts
  1:5  error  'foo' is assigned a value but never used        @typescript-eslint/no-unused-vars
  1:5  error  'foo' is never reassigned. Use 'const' instead  prefer-const

✖ 2 problems (2 errors, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

## Overriding recommendations

We can override the rules by specifying it in the `.eslintrc` file

```diff
{
    ...
    "rules": {
+       "@typescript-eslint/no-unused-vars": "warn"
    }
}
```

```bash
npm run lint
```

**OUTPUT**

```bash
  1:5  warning  'foo' is assigned a value but never used        @typescript-eslint/no-unused-vars
  1:5  error    'foo' is never reassigned. Use 'const' instead  prefer-const

✖ 2 problems (1 error, 1 warning)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

## Auto fix eslint problems

Some problems can be automatically fixed by adding the `--fix` flag as we did in the `package.json` script.

To see list of rules that can be automatically fixed in eslint rules, see here: https://eslint.org/docs/latest/rules/

```bash
npm run lint:fix
```

**OUTPUT**

```bash
  1:7  warning  'foo' is assigned a value but never used  @typescript-eslint/no-unused-vars

✖ 1 problem (0 errors, 1 warning)
```

The fixable problem has been resolved after the `src/index.ts` file has been automatically updated by `ESLint`:

```ts
const foo = 'bar';
```