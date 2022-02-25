# Create a library

```
yarn create a-library
```

## Steps run
```
yarn init
yarn add --dev prettier rollup typescript rollup-library-template ts-jest jest @types/jest @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-json eslint-plugin-prettier tslib
yarn ts-jest config:init
```


Files added:

**package.json**
```json
{
  "main": "./dist/cjs/index.js",
  "module": "./dist/esm/index.mjs",
  "typings": "dist/index.d.ts"
  "exports": {
    ".": {
      "require": "./dist/cjs/index.js",
      "import": "./dist/esm/index.mjs",
      "default": "./dist/index.umd.js"
    }
  }
  "scripts": {
    "build": "tsc -p . --declaration --emitDeclarationOnly && rollup -c",
    "prepublishOnly": "tsc -p . --declaration --emitDeclarationOnly && rollup -c",
    "test": "jest"
  }
}

```

**.prettierrc**
```json
{
  "singleQuote": true,
  "printWidth": 120,
  "trailingComma": "es5"
}
```

**eslintrc.js**
```
{
    "root": true,
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module"
    },
    "plugins": [
        "@typescript-eslint",
        "prettier",
        "json"
    ],
    "extends": [
        "plugin:json/recommended",
        "eslint:recommended",
        "plugin:@typescript-eslint/eslint-recommended",
        "plugin:@typescript-eslint/recommended"
    ],
    "rules": {
        "prettier/prettier": "error",
        "@typescript-eslint/naming-convention": "off",
        "@typescript-eslint/explicit-module-boundary-types": "off",
        "@typescript-eslint/semi": "warn",
        "@typescript-eslint/no-unused-vars": "off",
        "@typescript-eslint/no-explicit-any": "off",
        "curly": "warn",
        "no-throw-literal": "warn",
        "semi": "off"
    },
    "ignorePatterns": [
        "**/*.d.ts"
    ]
}
```



**tsconfig.json**
```
{
  "compilerOptions": {
    "baseUrl": ".",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "downlevelIteration": false,
    "experimentalDecorators": true,
    "importHelpers": true,
    "skipLibCheck": true,
    "noEmitHelpers": true,
    "lib": [
      "dom",
      "es2020"
    ],
    "jsx": "react",
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "sourceMap": true,
    "resolveJsonModule": true,
    "outDir": "./.build/types",
    "declaration": true,
    "emitDeclarationOnly": true,
    "rootDir": "./src"
  },
  "include": [
    "./src/**/*",
    "./src/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```


**.gitignore**
```
.build
node_modules
dist
pkg-tests/ts/yarn.lock
```

**pkg-tests/**
Various test imports for your modules. (see: https://github.com/IIIF-Commons/vault/tree/main/pkg-tests)

**.github/workflows/build-test.yml**
```yaml

   
name: Yarn build + test

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [ '16', '14' ]

    name: Node ${{ matrix.node }} build
    steps:
      - uses: actions/checkout@v1
      - name: Get yarn cache
        id: yarn-cache
        run: echo "::set-output name=dir::$(yarn cache dir)"
      - uses: actions/cache@v1
        with:
          path: ${{ steps.yarn-cache.outputs.dir }}
          key: ${{ runner.os }}-${{ matrix.node }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
      - run: yarn install --frozen-lockfile --non-interactive
      - run: yarn run build
      - run: yarn run test
      - run: node pkg-tests/node-load.js
      - run: node pkg-tests/node-load.mjs
      - run: node pkg-tests/node-umd.js
```
