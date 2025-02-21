# graphql-fragment-import ![CI](https://github.com/stefanpenner/grapqhl-fragment-import/workflows/CI/badge.svg)

This monorepo contains three packages:

* `@graphql-fragment-import/cli` - an executable to perform graphql module importing
* `@graphql-fragment-import/lib` - a library to reuse the underlying code in other libraries programmatically (linting, build tools etc)
* `@graphql-fragment-import/eslint-plugin` - an ESLint plugin that augments `@eslint-ast/graphql` with fragment import support


## Import syntax

```graphql

#import "./_my-fragment.graphql"
#import '../../_my-other-fragment.graphql'
#import "some-package/_its-fragment.graphql"
```


## Usage (as an executable)
```sh

npx @graphql-fragment-import/cli <file>
npx @graphql-fragment-import/cli <file> -o <output-file>
npx @graphql-fragment-import/cli <directory> -d <output-directory>
npx @graphql-fragment-import/cli <file> -ir <custom-import-resolver>
```

## Usage (as a library)


```sh
yarn add @graphql-fragment-import/lib
```

or

```sh
npm add --save graphql-fragment-import
```

```js
'use strict';

const fragmentImport = require('@graphql-fragment-import/lib/inline-imports');

const output = fragmentImport(fileContents, {
  resolveImport /* optional: allows the caller to provide an alternative resolution algorithm */,
  resolveOptions: {
    basedir /* required: specifies where to start resolving imported fragments from */,
    ...args /* any other resolve options are passed through to `resolveImport` when resolving imports */
  },
  fs /* optiona: allows for the caller to provide an alternative implementation of node's fs module */
});

output; // contains the grapqhl file, with all imports having been inlined
```

## Usage (ESLint)

```js
// .eslintrc
module.exports = {
  parser: '@eslint-ast/eslint-plugin-graphql/parser',
  parserOptions: {
    schema: 'path/to/schema.graphql',
  },
  plugins: ['@eslint-ast/graphql', '@graphql-fragment-import'],
  extends: [
    'plugin:@eslint-ast/graphql/recommended',
    'plugin:@graphql-fragment-import/recommended',
  ],
  rules: {
    '@graphql-fragment-import/validate-imports': [
      'error',
      // Options only need to be specified if you need a custom import resolver
      // The default is to use require.resolve (i.e. the node resolution algorithm)
      {
        importResolver: require.resolve('ember-cli-addon-aware-resolver'),
      },
    ],
  },
};
```

# Example importable fragments

Given the following files

```graphql
# // file: _my-fragment.graphql

fragment MyFragment on SomeType {
  fieldC
}
```

```graphql
# // file: my-query.graphql
#import './_my-fragment.graphql'

query {
  myQuery {
    fieldA,
    fieldB
    ...MyFragment
  }
}
```

results in:

```sh
npx @graphql-fragment-import/cli ./my-query.graphql
> fragment MyFragment {
>   fieldC
> }
>
> query {
>   myQuery {
>     filedA,
>     fieldB,
>     ...MyFragment
>   }
> }
```
alternatively, we can provide an input dir and output dir, and let the tool convert many files at once:

```js
npx graphql-fragment-import ./ -d ./output/
>   [processed] my-query.graphql -> ${pwd}/output//my-query.graphql
```
