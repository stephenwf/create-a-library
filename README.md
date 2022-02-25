# Create a library

**NOT YET PUBLISHED, BUT YOU CAN FOLLOW MANUALLY**

```
yarn create a-library [your-library-name]
```

## Steps run
```
yarn init
yarn add --dev prettier rollup typescript rollup-library-template ts-jest jest @types/jest @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-json eslint-plugin-prettier tslib cross-env
yarn ts-jest config:init
```


Files added/modified:

**package.json**
```json5
{
  // ... existing pkg ...
  
  "main": "./dist/cjs/index.js",
  "module": "./dist/esm/index.mjs",
  "typings": "dist/index.d.ts",
  "exports": {
    ".": {
      "require": "./dist/cjs/index.js",
      "import": "./dist/esm/index.mjs",
      "default": "./dist/index.umd.js"
    }
  },
  "scripts": {
    "start": "rollup -c --watch --dev",
    "build": "tsc -p . --declaration --emitDeclarationOnly && rollup -c",
    "prepublishOnly": "tsc -p . --declaration --emitDeclarationOnly && rollup -c",
    "test": "jest"
  }
}

```

**rollup.config.js**
```js
import { createRollupConfig, createTypeConfig } from 'rollup-library-template';
import pkg from './package.json';

const IS_DEV = process.env.NODE_ENV !== 'production';

const globalUMDName = 'Kinematic';

const baseConfig = {
  filesize: IS_DEV,
  minify: !IS_DEV,
  extra: {
    treeshake: true,
  }
};

const external = pkg.dependencies || [];

// Roll up configs
export default [
  createTypeConfig({
    source: './.build/types/index.d.ts',
  }),

  // dist/index.umd.js
  createRollupConfig({
    ...baseConfig,
    inlineDynamicImports: true,
    input: './src/index.ts',
    output: {
      name: globalUMDName,
      file: `dist/index.umd.js`,
      format: 'umd',
    },
    nodeResolve: {
      browser: true,
    },
  }),

  // dist/esm/index.mjs
  createRollupConfig({
    ...baseConfig,
    input: './src/index.ts',
    distPreset: 'esm',
    external,
  }),

  // dist/cjs/index.js
  createRollupConfig({
    ...baseConfig,
    input: './src/index.ts',
    distPreset: 'cjs',
    external,
  }),
];
```

**.prettierrc**
```json
{
  "singleQuote": true,
  "printWidth": 120,
  "trailingComma": "es5"
}
```

**eslintrc.json**
```json
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
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "importHelpers": true,
    "skipLibCheck": true,
    "noEmitHelpers": true,
    "lib": [
      "DOM",
      "ESNext"
    ],
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

