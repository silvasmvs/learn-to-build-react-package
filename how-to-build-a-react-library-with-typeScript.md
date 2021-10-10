# How to build a React library with TypeScript

Today, I am going to show you how to build a React library with TypeScript. Let's get started!

![React component](https://images.unsplash.com/photo-1619410283995-43d9134e7656?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=870&q=80)

## Table of contents

- [How to build a React library with TypeScript](#how-to-build-a-react-library-with-typescript)
  - [Table of contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Practices](#practices)
    - [1. Create new project with package.json file](#1-create-new-project-with-packagejson-file)
      - [Two important points](#two-important-points)
      - [Folder structure](#folder-structure)
    - [2. Configure lerna](#2-configure-lerna)
    - [3. Create new package packages/my-react-package](#3-create-new-package-packagesmy-react-package)
    - [4. Install peerDependencies for packages/my-react-package](#4-install-peerdependencies-for-packagesmy-react-package)
    - [5. Configure TypeScript for **`my-react-packages`**](#5-configure-typescript-for-my-react-packages)
    - [6. Configure Rollup to bundle our package](#6-configure-rollup-to-bundle-our-package)
    - [7. Write code for our package](#7-write-code-for-our-package)
    - [8. Declare module definition in the package.json file](#8-declare-module-definition-in-the-packagejson-file)
  - [Conclusion](#conclusion)
  - [References](#references)

## Prerequisites

- [**`lerna`**](https://lerna.js.org/) - A tool for managing JavaScript projects with multiple packages.
- [**`Yarn workspace`**](https://classic.yarnpkg.com/lang/en/docs/workspaces/) - Setup NodeJS workspace
- [**`React Components`**](https://reactjs.org/docs/components-and-props.html) - Basic knowledge about React Components
- Understand NodeJS module system [**`ECMAScript modules - esm`**](https://nodejs.org/api/esm.html#esm_modules_ecmascript_modules) and [**`CommonJS - cjs`**](https://nodejs.org/docs/latest/api/modules.html#modules_modules_commonjs_modules).

## Practices

### 1. Create new project with package.json file

```json
{
  "name": "learn-to-build-react-package",
  "private": true,
  "description": "Learn to to build react package",
  "keywords": [],
  "author": "PhatNguyen <phatnt.uit@gmail.com> (https://phatnguyenuit.github.io)",
  "workspaces": [
    "packages/*"
  ],
  "devDependencies": {
    "@types/react": "^17.0.6",
    "@types/react-dom": "^17.0.5",
    "lerna": "^4.0.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2"
  }
}

```

#### Two important points

- **`private`** should be turned to **`true`**
- **`workspaces`** contains workspace paths. I use **`packages/*`** to provide that my packages should be implemented under the **`packages`** folder

#### Folder structure

```sh
./learn-to-build-react-package
 |  |-- package.json
 |  |-- packages
 |  |   |-- my-react-package
 |  |   |   |-- package.json
```

[Go back ⏪](#table-of-contents)

### 2. Configure lerna

```json
{
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "independent"
}
```

- Notes:
  - `npmClient`: whether **`npm`** or **`yarn`** client called when running lerna command
  - `useWorkspaces` use workspace flag
  - `version`: we should choose `independent` to make every single package in our workspace has an independent version, not the same)

[Go back ⏪](#table-of-contents)

### 3. Create new package packages/my-react-package

```sh
mkdir packages/my-react-package;
cd packages/my-react-package;
npm init -y;
```

[Go back ⏪](#table-of-contents)

### 4. Install peerDependencies for packages/my-react-package

```sh
npx lerna add react --scope my-react-package --peer;
npx lerna add react-dom --scope my-react-package --peer;
```

Why do we use **`peerDependencies`** ?

=> It is because our package scope is just a MODULE that can be installed by any projects and which must have our package **`peerDependencies`** installed also.

Now our **`my-react-packages`** *peerDependencies* section in the __package.json__ file looks like below

```json
  "peerDependencies": {
    "react": "^17.0.2",
    "react-dom": "^17.0.2"
  }
```

[Go back ⏪](#table-of-contents)

### 5. Configure TypeScript for **`my-react-packages`**

Our **my-react-packages/tsconfig.json** file should look like below:

```json
{
  "compilerOptions": {
    "outDir": "lib/esm",
    "module": "esnext",
    "target": "es5",
    "lib": ["es6", "dom", "es2016", "es2017"],
    "jsx": "react-jsx",
    "declaration": true,
    "moduleResolution": "node",
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "esModuleInterop": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "suppressImplicitAnyIndexErrors": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*.ts*"],
  "exclude": ["node_modules", "lib"]
}
```

Some highlights:

- `outDir` stands for output directory after compiled TypeScript to the `target` ecmascript version `es5`
- Must use `"jsx": "react-jsx"` to use JSX compiler
- Turn on `declaration` to extract type definitions
- Folder "**node_modules**" and "**lib**" must be excluded while compiling TypeScript to JavaScript.

[Go back ⏪](#table-of-contents)

### 6. Configure Rollup to bundle our package

- Install devDependencies

```sh
my-react-packages:~ yarn add -D rollup rollup-plugin-typescript2 typescript
```

- Create new file **`my-react-packages/rollup.config.js`**

```javascript
import typescript from 'rollup-plugin-typescript2';
import pkg from './package.json';

export default {
  input: 'src/index.ts',
  output: [
    {
      file: "./lib/cjs/index.js",
      format: 'cjs',
    },
    {
      file: "./lib/esm/index.js",
      format: 'es',
    },
  ],
  external: [
    ...Object.keys(pkg.peerDependencies || {}),
  ],
  plugins: [
    typescript({
      typescript: require('typescript'),
    }),
  ],
};

```

- Our module exposes two types of module system: CommonJS - cjs and ECMAScript - esm
- All packages in the **`peerDependencies`** section will be treated as external dependencies. It means Rollup does not include them in the bundling process.
- We use Rollup TypeScript plugin to support TypeScript code.

[Go back ⏪](#table-of-contents)

### 7. Write code for our package

// TODO

[Go back ⏪](#table-of-contents)

### 8. Declare module definition in the package.json file

```json
{
  "main": "./lib/cjs/index.js",
  "module": "./lib/esm/index.js",
  "types": "./lib/esm/index.d.ts",
  "files": [
    "/lib"
  ],
}
```

- Here we define the main file for our project is `"./lib/cjs/index.js"`
- Our package also expose esm module at `"./lib/esm/index.js"`
- Type definitions will be includes at `"./lib/esm/index.d.ts"`
- The last important is `"files"`, which tells NPM which files or folders will be packaged. For our package it will be `"lib"` folder

[Go back ⏪](#table-of-contents)

## Conclusion

To sum it up, there are 8 steps to build a React Library with TypeScript:

1. Create new project with package.json file
2. Configure lerna
3. Create new package packages/my-react-package
4. Install peerDependencies for packages/my-react-package
5. Configure TypeScript for **`my-react-packages`**
6. Configure Rollup to bundle our package
7. Write code for our package
8. Declare module definition in the package.json file

Last but not least, thank you for reading through this section! I hope you find this article helpful and solve your concerns when trying to build a React library.

Here is [my full example code](//link) on GitHub repositories.
Reaching to it if you want to explore more.

If you have any questions or feedback, do not hesitate to leave a comment in the box below.

Thank you and see you next time!

[Go back ⏪](#table-of-contents)

## References

- [**`Lerna`**](https://lerna.js.org/)
- [**`Yarn workspace`**](https://classic.yarnpkg.com/lang/en/docs/workspaces/)
- [**`React Components`**](https://reactjs.org/docs/components-and-props.html)
- [**`NodeJS peerDependencies`**](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#peerdependencies)
- [**`ECMAScript modules - esm`**](https://nodejs.org/api/esm.html#esm_modules_ecmascript_modules)
- [**`CommonJS - cjs`**](https://nodejs.org/docs/latest/api/modules.html#modules_modules_commonjs_modules)

[Go back ⏪](#table-of-contents)