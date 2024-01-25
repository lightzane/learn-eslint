# Custom ESLint Rules (Javascript)

Create your own custom ESLint Rules

## Rule Structure

https://eslint.org/docs/latest/extend/custom-rules#rule-structure

```js
module.exports = {
  meta: {
    type: 'suggestion',
    docs: {
      description: 'Description of the rule',
    },
    fixable: 'code',
    schema: [], // no options
  },
  create: function (context) {
    return {
      // callback functions
    };
  },
};
```

# Getting Started

## Name a Plugin

https://eslint.org/docs/latest/extend/plugins#name-a-plugin

```text
eslint-plugin-<plugin-name>
```

## Install dependencies

Create a blank directory folder

```bash
mkdir eslint-plugin-example
cd eslint-plugin-example
```

```bash
npm install eslint -D
```

This will generate `package.json`. Then, update it:

```diff
{
+ "name": "eslint-plugin-example",
+ "main": "src/index.js",
}
```

## Create a Rule

Create custom rule that only `"bar"` should be the value for any `const foo` variable.

`src/rules/enforce-foo-bar.js`

```js
/** @type import('eslint').Rule.RuleModule */
module.exports = {
    meta: {
        type: 'problem',
        docs: {
            description: 'Some description over here...'
        },
        fixable: 'code',
        schema: []
    },
    create(context) {
        return {
            // Performs action in the function on every variable declarator
            VariableDeclarator(node) {
                
                // Check if a `const` variable declaration
                if (node.parent.kind === 'const') {

                    // Check if variable name is `foo`
                    if (node.id.type === 'Identifier' && node.id.name === 'foo') {

                        // Check if value of variable is "bar"
                        if (node.init && node.init.type === 'Literal' && node.init.value !== 'bar') {

                            /*
                             * Report error to ESLint. Error message uses
                             * a message placeholder to include the incorrect value
                             * in the error message.
                             * Also includes a `fix(fixer)` function that replaces
                             * any values assigned to `const foo` with "bar".
                             */
                            context.report({
                                node,
                                message: 'Value other than "bar" assigned to `const foo`. Unexpected value: {{ notBar }}',
                                data: {
                                    notBar: node.init.value
                                },
                                fix(fixer) {
                                    return fixer.replaceText(node.init, '"bar"')
                                }
                            })

                        }

                    }

                }

            }
        }
    }
}
```

`create()`: Returns an object with methods that ESLint calls to “visit” nodes while traversing the abstract syntax tree (**AST** as defined by [ESTree](https://github.com/estree/estree)) of JavaScript code

We can use a tool like **AST Explorer**... see below [References](#references)


## Register Rule

`src/rules/index.js`

```js
const fooBarRule = require('./enforce-foo-bar');

module.exports = {
  'enforce-foo-bar': fooBarRule,
  // other rules...
};
```

## Create Main Entry Point

`src/index.js` - (should match with the `package.json` **main** field)

```js
const rules = require('./rules');

module.exports = {
  rules,
};
```

## Test Locally

`src/rules/enforce-foo-bar.test.js`

```js
const { RuleTester } = require('eslint');
const fooBarRule = require('./enforce-foo-bar');

const ruleTester = new RuleTester({
  // Must use at least ecmaVersion 2015 because
  // that's when `const` variables were introduced.
  parserOptions: {
    ecmaVersion: 2015,
  },
});

// Throws error if the tests in ruleTester.run() do not pass
ruleTester.run(
  'enforce-foo-bar', // rule name
  fooBarRule, // rule code
  {
    // checks
    // 'valid' checks cases that should pass
    valid: [
      {
        code: "const foo = 'bar'",
      },
    ],
    // 'invalid' checks cases that should not pass
    invalid: [
      {
        code: "const foo = 'baz'",
        output: 'const foo = "bar"',
        errors: 1,
      },
    ],
  },
);

console.log('All tests passed!');
```

```bash
node src/rules/enforce-foo-bar.test
```

## Output package

```bash
npm pack
```

This will generate the `<package-name>-<version>.tgz` file which we can place inside other **node project** and install it:

```bash
npm i -D <package-name>-<version>.tgz
```

In our case, it would be `npm i -D eslint-plugin-example-0.0.1.tgz`.

Then we can include it inside the `.eslintrc` file

`.eslintrc.yml`

```diff
extends:
  - eslint:recommended
plugins:
+ - 'example'
rules: {
+ 'example/enforce-foo-bar': error
}
```

`.eslintrc.json`

```json
{
    "extends": ["eslint:recommended"],
    "plugins": [
        "example" // plugin-name
    ],
    "rules": {
        "example/enforce-foo-bar": "error"
    }
}
```

**NOTE** that we are omitting the *eslint-plugin*, if you remember the plugin naming convention: `eslint-plugin-<plugin-name>`

```bash
npm run lint
```

**OUTPUT**

```bash
  1:7  error  Value other than "bar" assigned to `const foo`. Unexpected value: baz  example/enforce-foo-bar

✖ 1 problem (1 error, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

## Add Rule to Recommendations

There are scenarios where we don't want users to explicitly write the rule in `.eslintrc` file. As we can enable the rules by default once they added the plugin. 

### Create recommended config

`src/configs/recommended.js`

```js
module.exports.rules = {
  'enforce-foo-bar': 'error',
};
```

`src/configs/index.js`

```js
const recommended = require('./recommended');

module.exports = {
  recommended,
};
```

This `recommended` field will now be accessible via `'example/recommended'` later in when `.eslintrc` extends the config.

## Register config

`src/index.js`

```diff
+const configs = require('./configs');
 const rules = require('./rules');
 
 module.exports = {
+  configs,
   rules,
 };
```

Do `npm pack` and `npm install` it again on the other project.

## Update the project's .eslintrc file

`.eslintrc.yml`

```diff
extends:
  - eslint:recommended
+ - plugin:example/recommended
plugins:
  - 'example'
rules: {
- 'example/enforce-foo-bar': error
}
```

`.eslintrc.json`

```diff
{
    "extends": [
        "eslint:recommended",
+       "plugin:example/recommended"
    ],
    "plugins": [
        "example"
    ],
    "rules": {
-       "example/enforce-foo-bar": "error"
    }
}
```

# References

- https://astexplorer.net/

This will help you get all the available **AST**.

Syntax: `Javascript`
Parser: `espree` - TODO: Find documentation in ESLint if they are using this parser
