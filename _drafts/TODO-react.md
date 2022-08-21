
# React usage notes

## Installation

### Linux

**1 - Install / Update NVM (Node Version Manager)**

Install or update nvm. You should run the install script using one of the following commands:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
# or
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

Now to start using nvm run `source ~/.bashrc`, or close and reopen the terminal.

Ref: https://github.com/nvm-sh/nvm#installing-and-updating

**Install and start using the latest LTS version of NodeJS**

```sh
# Install the latest LTS version
nvm install --lts
# Use the latest LTS version
nvm use --lts
# Attempt to upgrade to the latest working `npm` on the current node version
nvm install-latest-npm
```

NOTE: If `nvm install-latest-npm` throws error, then close and reopen the terminal and try again.

Otherwise you could choose an specific version from `nvm ls-remote --lts` and install it with, for example for 'v16.14.0', `nvm install 16.14.0`.

### Windows

**1 - Install / Update NVM for Windows**

NOTE: Before to install NVM for Windows, uninstall any existing versions of Node.js (otherwise you'll have conflicting versions). Delete any existing Node.js installation directories (e.g., `%ProgramFiles%\nodejs`) that might remain. NVM's generated symlink will not overwrite an existing (even empty) installation directory.

Download and run the latest installer from https://github.com/coreybutler/nvm/releases. If NVM for Windows doesn't appear to work immediately after installation, restart the terminal/powershell (not the whole computer).

NOTE: To upgrade NVM for Windows, download and run the new installer. It will safely overwrite the files it needs to update without touching your node.js installations. **Make sure you use the same installation and symlink folder**. If you originally installed to the default locations, you just need to click "next" on each window until it finishes.

**Install and start using the latest LTS version of NodeJS**

Run the following commands in an Admin shell. **You'll need to start powershell or Command Prompt as Administrator to use NVM for Windows**.

```sh
# Install the latest LTS version
nvm install lts

# Use the latest LTS version
nvm use lts

# Attempt to upgrade to the latest working `npm` on the current node version
nvm install-latest-npm
```

## Create and run an initial app

### Using the create-react-app CLI

[Create React App](https://create-react-app.dev/) is an officially supported way to create single-page React applications.

**Create the app**

```sh
# with typescript
npx create-react-app my-app --template typescript

# without typescript
npx create-react-app my-app
```

This will create a project using the base template ([cra-template])(https://www.npmjs.com/package/cra-template)).

You can now optionally start a new app from a template by appending --template [template-name] to the creation command. Templates are always named in the format cra-template-\[template-name\], however you only need to provide the \[template-name\] to the command.

For example, you can start a new TypeScript-based app using the base TypeScript template ([cra-template-typescript](https://www.npmjs.com/package/cra-template-typescript)):

```sh
# format: npx create-react-app my-app --template [template-name]
npx create-react-app my-app --template typescript
```

See the list of available templates by searching for "[cra-template-*](https://www.npmjs.com/search?q=cra-template-*)" on npm.

**Run it**

```sh
cd my-app
npm start
```

It automatically opens your browser to http://localhost:3000/.

The `--open` (or just `-o`) option automatically opens your browser to http://localhost:4200/.

### From scratch

TODO

## Concepts

TODO

**Reducer**

TODO

## Testing

### Using create-react-app

Create React App uses [Jest](https://jestjs.io/) as its test runner. Note that **Jest is a Node-based runner. This means that the tests always run in a Node environment and not in a real browser**. This lets us enable fast iteration speed and prevent flakiness.

**While Jest provides browser globals such as 'window' thanks to jsdom, they are only approximations of the real browser behavior**. Jest is intended to be used for unit tests of your logic and your components rather than the DOM quirks.

When you run `npm test`, Jest will launch in watch mode. Every time you save a file, it will re-run the tests.

For non-CI environments, you can pass the --watchAll=false flag to disable test-watching.

#### Using the React Testing Library

If you want to use the React Testing Library with a create-react-app app, then you have to install it manually. It's recommended to use with jest-dom for improved assertions:

```sh
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

### From scratch

TODO

### The library react-testing-library

The [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) is a library for testing React components in a way that resembles the way the components are used by end users. It is well suited for unit, integration, and end-to-end testing of React components and applications. It works more directly with DOM nodes, and therefore it's recommended to use with [jest-dom](https://testing-library.com/docs/ecosystem-jest-dom/) (a companion library for the Testing Library that provides custom DOM element matchers for Jest) for improved assertions.

See [react-testing-library intro](https://testing-library.com/docs/react-testing-library/intro) and [https://github.com/testing-library/react-testing-library](https://github.com/testing-library/react-testing-library) for details.


### Filename conventions with Jest

Jest will look for test files with any of the following popular naming conventions:

- Files with .js suffix in \_\_tests\_\_ folders.
- Files with .test.js suffix.
- Files with .spec.js suffix.
The .test.js / .spec.js files (or the \_\_tests\_\_ folders) can be located at any depth under the /src top level folder.

**Is recommended to put the test files (or \_\_tests\_\_ folders) next to the code they are testing so that relative imports appear shorter. For example, if App.test.js and App.js are in the same folder, the test only needs to import App from './App' instead of a long relative path**. Collocation also helps find tests more quickly in larger projects.

### Focusing and excluding tests with Jest

You can replace it() with xit() to temporarily exclude a test from being executed.

Similarly, fit() lets you focus on a specific test without running any other tests.

### Version Control integration with Jest

By default, when you run `npm test`, Jest will only run the tests related to files changed since the last commit. This is an optimization designed to make your tests run fast regardless of how many tests you have. However it assumes that you don’t often commit the code that doesn’t pass the tests.

**Jest will always explicitly mention that it only ran tests related to the files changed since the last commit. You can also press `a` in the watch mode to force Jest to run all tests**.

### Building and running Jest on a CI server

Jest will always run all tests on a CI server or if the project is not inside a Git repository.

By default `npm test` runs the watcher with interactive CLI. However, you can force it to run tests once and finish the process by setting an environment variable called 'CI'.

When creating a build of your application with `npm run build` linter warnings are not checked by default. Like `npm test`, you can force the build to perform a linter warning check by setting the environment variable 'CI'. If any warnings are encountered then the build fails.

Note that **popular CI servers already set the environment variable CI by default but you can do this yourself too**.

#### Running manually

**Windows**

CMD:

```cmd
set CI=true&&npm test

set CI=true&&npm run build
```

Powershell:

```cmd
($env:CI = "true") -and (npm test)

($env:CI = "true") -and (npm run build)
```

**Linux, macOS (Bash)**

```sh
CI=true npm test

CI=true npm run build
```

### Snapshot Testing with Jest

Snapshot testing is a feature of Jest that automatically generates text snapshots of your components and saves them on the disk so if the UI output changes, you get notified without manually writing any assertions on the component output.

Note that jsdom is not needed for snapshot testing. This means that you could safely set --env=node, and your tests will run faster.

See [https://jestjs.io/blog/2016/07/27/jest-14.html](https://jestjs.io/blog/2016/07/27/jest-14.html) for details.

### Coverage Reporting with Jest

Jest has an integrated coverage reporter. Run `npm test -- --coverage` (note extra -- in the middle) to include a coverage report in the output.