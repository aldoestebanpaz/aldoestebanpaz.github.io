
# Angular usage notes

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
nvm use lts/*
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

## Install / Update the Angular CLI

NOTE: If the command throws a 'not found' message, then close and reopen the terminal and try again.

```sh
npm install -g @angular/cli
# or update with:
#   npm update -g @angular/cli
```

Ref: https://angular.io/guide/setup-local


## Create and run an initial app

**Create the app**

```sh
ng new my-app
```

**Run it**

```sh
cd my-app
ng serve --open
```

The `--open` (or just `-o`) option automatically opens your browser to http://localhost:4200/.

## Concepts

**Directives**

TODO

**Components**

A Component is a class that contains application data and logic, and is associated with an HTML template that defines a view to be displayed.

**Templates**

A .html file that combines ordinary HTML with 'Angular directives' and 'binding markup' that allow Angular to modify the HTML before rendering it for display.

The metadata for a component class associates it with a template that defines a view.

**Interpolation and template expressions**

Interpolation allows you to incorporate calculated strings into the text between HTML element tags and within attribute assignments. Template expressions are what you use to calculate those strings.

Interpolation refers to embedding expressions into marked up text. By default, interpolation uses as its delimiter the double curly braces, {{ and }}.

**Data binding**

TODO

**Pipes**

Pipes are simple functions to use in template expressions throughout the application to accept an input value and return a transformed value. For example, you could use a pipe to show a date as "April 15, 1988" rather than the raw string format.

**Services**

Services are reusable code (typically classes) hat is shared by the Components. Services provide specific functionality not directly related to views. Service providers can be injected into components as dependencies.

The metadata for a service class provides the information Angular needs to make it available to components through dependency injection (DI).

**Modules**

An Angular module:

1- group components (but also services, directives, pipes etc ...)
2- define their dependencies
3- define their visibility.

An Angular module is simply defined with a class (usually empty) and the NgModule decorator.

An application always has at least a 'root module' (conventionally named AppModule) that enables bootstrapping, and typically has many more feature modules.

NgModule differ from and complement JavaScript (ES2015) modules. An NgModule declares a compilation context for a set of components that is dedicated to an application domain, a workflow, or a closely related set of capabilities. An NgModule can associate its components with related code, such as services, to form functional units.

Like JavaScript modules, NgModules can import functionality from other NgModules, and allow their own functionality to be exported and used by other NgModules. For example, to use the router service in your app, you import the Router NgModule.

**Decorators**

Modules, components and services are classes that use decorators. These decorators mark their type and provide metadata that tells Angular how to use them.

**Router**

An NgModule that provides a service that lets you define a navigation path among the different application states and view hierarchies in your application.

It is modeled on the familiar browser navigation conventions:
- Enter a URL in the address bar and the browser navigates to a corresponding page.
- Click links on the page and the browser navigates to a new page.
- Click the browser's back and forward buttons and the browser navigates backward and forward through the history of pages you've seen.

The router maps URL-like paths to views instead of pages. When a user performs an action, such as clicking a link, that would load a new page in the browser, the router intercepts the browser's behavior, and shows or hides view hierarchies.

## Template-driven vs Reactive forms

TODO
















npx @angular/cli help
npx @angular/cli new <project_name>
# ? Would you like to add Angular routing? Yes
# ? Which stylesheet format would you like to use? SCSS   [ https://sass-lang.com/documentation/syntax#scss ]



