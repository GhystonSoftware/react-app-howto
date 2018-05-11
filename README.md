# How to set up a new React app #


* [Install NPM and Yarn](#install-npm-and-yarn)
    * [Why use Yarn?](#why-use-yarn)
    * [Why use a Windows package manager](#why-use-a-windows-package-manager)
* [Scaffold the app](#scaffold-the-app)
    * [Should you ever not use create-react-app?](#should-you-ever-not-use-create-react-app)
	* [Start the app](#start-the-app)
	* [Add routing](#add-routing)
	* [Add styling](#add-styling)
	* [Add state management](#add-state-management)
	* [Testing](#testing)
	* [Tweak the tslint config](#tweak-the-tslint-config)
	* [Hook it up to an API](#hook-it-up-to-an-api)
* [Serve with MVC](#serve-with-mvc)
	* [.NET Core](#net-core)
	* [Maven and Spring Boot](#maven-and-spring-boot)
	* [Django](#django)
* [Configure Jenkins](#configure-jenkins)
* [Eject from create-react-app](#eject-from-create-react-app)

This guide was written in March 2018 - given the pace at which the React world moves it's likely that bits of this are out of date, or new best practices have emerged. If you can, remember to check up-to-date release notes or blogs or chat to someone who keeps up to speed!

## Install NPM and Yarn ##

Install Node / NPM. If you don't already have it installed you may want to use a package manager such as Chocolatey, Scoop or Node Version Manager - install that first and then use it to install/upgrade Node. This can help keep on top of new versions of Node.

We normally use Yarn to manage packages and run scripts instead of NPM.

### Why use Yarn?

Yarn introduces `yarn.lock` files which specify exactly which versions of all your dependencies (and dependencies of your dependencies!) you use. You should check these in, and then Jenkins etc. can rebuild your app with exactly the same dependencies - you'll never fall prey again to a dependency's dependency updating and breaking your app.

Yarn is also much faster than NPM at downloading and installing packages. It edits the same `package.json` file as NPM, though you shouldn't mix & match the two because the `yarn.lock` file will get out of sync.

### Why use a Windows package manager?

#### Pros

Instead of scouring the internet for a download link, carefully selecting the correct installer, and then clicking through a million installation steps, Windows package managers like [Chocolatey](https://chocolatey.org/) or [Scoop](http://scoop.sh/) allow you to instantly install, upgrade, or downgrade Windows programs with a single command. For example, installing Node Version Manager using Chocolatey is as easy as typing `choco install nvm` from the command line, updating is as easy as `choco upgrade nvm`, and downgrading to an older version is as easy as `choco upgrade --version 1.1.1`.

#### Cons

Some may say: "Why would I need a package manager to install a package manager (to install a package manager)?"

#### Node Version Manager

Allows you to have multiple versions of Node installed (useful if you're working on support / multiple projects, for example), and allows you to easily upgrade and downgrade between versions. it's much lighter-weight than a full-blown package manager like Chocolatey or Scoop.

#### Chocolatey vs. Scoop ##

Scoop's take on the difference can be found here: https://github.com/lukesampson/scoop/wiki/Chocolatey-Comparison

The main headline is that Scoop is used primarily for installing open-source command-line developer tools (e.g. Flyway), while Chocolatey can be used to install a much larger and more diverse range of programs (e.g. Google Chrome).

## Scaffold the app ##

If you're planning to build your React app on top of .NET Core, skip to the .NET Core section below.

Normally, your first point of call should be the create-react-app CLI. We use it with an alternative set of scripts for Typescript.
````
npm install -g create-react-app

create-react-app my-app --scripts-version=react-scripts-ts
````

This bootstraps a fully-fledged but opinionated starter react app for you. All the configuration is hidden away in the react-scripts (or in our case, react-scripts-ts) package so you don't need to worry about the webpack config etc. - but you can't edit it either.

### Should you ever not use create-react-app?

Create-react-app is the standard way to create React apps these days and is being pushed heavily by both Microsoft and Facebook. We generally think that you should use this unless you have a really good reason not to.

#### Pros

* It's the industry standard and recommended by React (Facebook), Microsoft etc.
* You'll definitely start out with something that works, so you can develop straight away (you can always tweak the config later)
* All the edge cases are (probably) covered
* It encourages best practice
* You can always eject (see below) if you need to tweak the config (this exposes all the underlying configuration and removes all dependencies on create-react-app / react-scripts-ts)
* Even if you eject immediately, it still saves you a lot of time setting the things up that you don't want to change

#### Cons

* It's extremely opinionated about how you develop - while you can easily edit the tsconfig, tslint and package.json files it ships with, everything else is hidden under the hood unless you eject
* If you're using LESS or SASS and don't want to eject, you have to compile these as a separate step - you can't have them compiled by Webpack
* While you can eject, the config produced by this is complex and split across many files worth of configuration and scripts so this can be quite daunting - plus it creates a lot of clutter in your project for use cases you may not need.
* We've had typings issues in the past where the libraries which shipped with create-react-app were slightly behind their corresponding typings - there's no way to adjust these dependencies short of figuring out which version create-react-app imports and manually specifying the corresponding typings version.

#### Alternatives

If you still want to use Webpack, you can follow the TypeScript documentation to set up a basic TypeScript, React and Webpack project.

A good reason not to use create-react-app might be if you don't want to use Webpack as your bundler.

### Start the app

Run `yarn start` (or `npm start`) and the app will start up on [http://localhost:3000](http://localhost:3000) This should display a basic app with a rotating logo! (If it doesn't, good luck, you're on your own now.) This watches your changes (most of the time) and refreshes the page when it needs to. It notably also won't display anything if you have compilation errors anywhere in your project (including in your unit tests and other files not actually needed to run the app - so keep those tests up to date!)

````
yarn start
````

#### Troubleshooting

If you're getting warnings about a missing base URL / root URL when starting, you may need to fix the baseUrl in tsconfig.json:

````
{
  "compilerOptions": {
    "baseUrl": ".", // add this line
    ...
  }
}
````
### Add routing

The react-scripts README is full of useful instructions on how to add various standard components to your new React app.

Follow its instructions for adding React Router, which is basically just

````
yarn add react-router-dom
````

**Watch out** - React Router has upgraded to v4 recently! This introduces a new style of nested routing which we haven't really taken full advantage of yet on any projects. See [this blog](https://css-tricks.com/react-router-4/) for an explanation.

### Add styling

You have three options for styling: styled-components, LESS, or SASS.

#### Why use styled-components?

Styled-components allows you to have all the code for your component in a single file, by having little snippets of styling in tagged template literals in your TSX files. It also conveniently allows you to create e.g. divs with just a little bit of styling as reusable components, without having to create a whole component class with its own separate CSS/LESS file just for that tiny bit of styling.
````
import * as colours from '../styling/colours';
import * as spacing from '../styling/spacing';
import { tinyMaxWidth } from '../styling/screenWidths';
 
// Sometimes styled-components is useful for naming wrapping divs!
const AccordionContainer = styled.div`
`;
 
// You can use LESS syntax to nest your styling
const AccordionTitle = styled.div`
  h2 {
    margin-bottom: 0.5rem;
 
    @media (max-width: ${tinyMaxWidth}) {
      margin-bottom: 0;
    }
`
 
// You can use TypeScript variables
const AccordionBody = styled.div`
  border-top: 1px solid ${colours.paleGrey};
  padding: ${spacing.panelPadding};
`;
 
// When everything is named, the component reads much more smoothly
export const DashboardAccordion = (props: {}) => (
  <AccordionContainer>
    <AccordionTitle>Title</AccordionTitle>
    <AccordionBody>
      // ...
    </AccordionBody>
  </AccordionContainer
}
````

#### Why use LESS instead of styled-components?

Using LESS can be better if you also have an MVC portion of the site. If you use styled-components, then to share styling between the two (e.g. variables, mixins, ...), you'd have to duplicate these as styled-components expects TypeScript variables, not LESS variables.

You can still follow the ethos of styled-components of styling component by component if you have a LESS file for each component which sits next to its TSX file. However these LESS files can now import variables and mixins from a central variables file which is also shared with your MVC site.

Having separate LESS files can also be tidier if you have a lot of detailed styling for each component - this can make the component TSX files very large and mean you have to scroll a long way before you get to the actual functionality. Some developers may prefer to split the styling out into a separate file instead. (Of course, you could also doing this by having separate files for your styled-components, even if they're only used for one "proper" component.)

#### Why use SASS?

Generally we don't find that SASS has any advantages over LESS. The only reason to use SASS is if the customer is already using it, or you want to use variables / mixins from a library using it.

#### Using styled-components and LESS together


If you really want to use styled-components in your React app but also need LESS for e.g. an MVC landing page, there are two libraries js-to- styles-var-loader and less-vars-to-js which will translate variables between the two. I've had a little play around with these and the code you
have to write to link this up seems pretty ugly though, so do this at your own risk!

#### styled-components

````
yarn add styled-components
````

#### LESS

Here's how we set this up recently:
````
yarn add npm-run-all less-watch-compiler
````
And then in package.json:
````
"scripts": {
    "start": "npm-run-all -p start-css start-js",
    "build": "npm-run-all build-css build-js",
    "start-js": "react-scripts-ts start",
    "build-js": "react-scripts-ts build",
    "start-css": "less-watch-compiler src src",
    "build-css": "less-watch-compiler --run-once src src",
    // test etc.
}
````

**Watch out** - You need import the CSS files into your TSX - not the LESS files!

In App.tsx:
````
import * as React from 'react';
import './App.css';
 
export function App({history}: Props) {
  return (
    <div className="App">
      <div className="App-content">
        // ...
      </div>
    </div>
  );
}
````

#### SASS

Follow the instructions on the react-scripts README.

**Watch out** - remember to import the CSS files into your TSX, not the SCSS files! As you'll have noticed in the setup, the webpack config from create-react-app doesn't compile SASS, you have to do this separately. Also make sure that you run the SASS build step before you try to compile the app for anything, including running tests, otherwise the CSS files you're trying to import may not exist.

````
import * as React from 'react';
import './Navbar.css';
 
export const Navbar = (props: Props) => (
  <nav className="navbar">
    // ...
  </nav>
);
````

### Add state management

We normally use Redux for state management, but sometimes there are cases when you can get away with just React's setState(), or that plus some higher-order components to manage state like authentication tokens which are used by all components.

#### Pros and Cons of Redux

##### Pros

* All your state in one place, and can only be updated through one route
* Easy to manage large / complex state
* No need to pass huge amounts of props to child components
* [Time travel](https://github.com/gaearon/redux-devtools#features) debugging - undo and redo in the browser dev tools as well as seeing everything that ever happened

##### Cons

* Can generate a lot of boilerplate (see "generic actions and reducers" below for ways of reducing this)
* Sometimes feels like using a sledgehammer to crack a nut

#### Partial Redux

Not everything has to live in Redux! For example, you might choose to handle API calls outside of Redux. If you do this, you won't be able to use Redux's time travel functionality, but you might save yourself a lot of boilerplate.

#### Vanilla React

Jira Test Plan Management uses vanilla React setState, with all state controlled by the parent App.tsx. It's a good example of an app small enough to use vanilla setState, but because the state all has to be saved in one big bang, it's also a good example of how the state management
by the parent component can get out of hand as state gets bigger.

#### Helpers

Dean has written a helpful little library for creating merge reducers: https://github.com/Dean177/create-reducer-extra

#### Redux thunk

Sometimes you don't just want to dispatch plain strings as actions - you may want to dispatch actions which kick off a sequence of other actions, e.g. when making an API request. [Redux Thunk](https://github.com/reduxjs/redux-thunk) allows you to dispatch arbitrary functions as actions, which get called with helpful parameters when dispatched. 

#### Redux

````
yarn add redux react-redux @types/react-redux
````

Simplest version:
````
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import { appReducer } from './reducers/appReducer';
 
const store = createStore(appReducer);
 
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root') as HTMLElement
);
````

#### Rematch

A library called [Rematch](https://github.com/rematch/rematch) promises Redux without the boilerplate. At the time of writing it's still too immature to be used with TypeScript - the typings don't really work and you spend a lot of time casting to 'any'. Keep an eye on it though!

#### Higher-order components

If you're not using Redux, you might still have a few bits of state which are awkward to pass from parent to child / fetch directly from the API, or are needed by most components. A good example is authentication - details of the current user like their user ID or auth token for the API. You can encapsulate this by wrapping all the components which need this in a higher-order component which provides this information as props.

### Testing

#### Add Enzyme

Follow the instructions on the react-scripts README (scroll down a bit).

A lot of projects use a little helper extension called toRenderWithoutError. Dean has recently published it as an [open-source NPM package](https://github.com/Dean177/jest-to-render-without-error)!

#### Set up IntelliJ to run tests

If you're using IntelliJ or Rider, it's sometimes quite useful to be able to run the tests inside the IDE. Not only does this run all the test once only (unlike the watcher task 'yarn test', which by default runs only the tests affected by files changed since your last commit, and re-runs when you change things), but you get a nice visual output of the tests that makes it easy to re-run individual tests. (This is also possible with the commands provided inside 'yarn test', by the way.)

Create a new Jest run configuration as follows. If you're using any environment variables in .env.test (see the react-scripts documentation for .env
), add them as well.

![IntelliJ screenshot](https://i.imgur.com/mdUhI89.png)


Then you get a nice test window, like so!

![IntelliJ screenshot](https://i.imgur.com/53DTy9U.png)

### Tweak the tslint config

We like to make one change to the tsconfig.json - it seems overkill to fail to compile just because you have unused local variables. Some projects also remove the suppressImplicitAnyIndexErrors option.

````
{
    "compilerOptions": {
    ...
    "suppressImplicitAnyIndexErrors": true, // remove this line if it doesn't affect your project, and add it back in if necessary
    "noUnusedLocals": true // remove this line, and optionally add it to the tslint file instead
    ...
}
````

We often like to make a few small changes to the tslint config provided by create-react-app - especially because if you have linting errors, `yarn build` will throw an error!
````
{
    "defaultSeverity": "warning", // add this if you want your app to still run with 'yarn start' if it has linting errors (at CI time, these warnings will still be treated as errors)
    "rules: {
        "align": [
            true,
            "parameters",
            // remove "arguments" - some people find these tend to line up weirdly rather than helpfully
            "statements"
        ],
        "eofline": true, // instead of false - some people like a blank line at the end of files
        "max-line-length": [ false ], // instead of [ true, 120 ] - it's often useful not to have your app fail to build just because you went one or two characters over in a comment
        "no-empty": false, // instead of true - empty functions are useful when testing
        "no-non-null-assertion": true, // definitely add this - handling the null case is safer than asserting not null
        "no-unused-variable": true, // add this in tslint instead of having it in tsconfig, or just remove it from the tsconfig - this may be useful at build time but you often want unused variables while developing
        "semicolon": [false, "always"], // instead of [true, "always"] - this one is a matter of style! Dean prefers no semicolons at all but backend developers often like the semicolons.
    }
}
````

#### Set up IntelliJ for TSLint

Finally, if you're using IntelliJ or Rider, you also need to tweak the settings so that when you hit Ctrl+Alt+L, it actually formats your code according to the tslint file.

Download example Project.xml (paste this into / merge this with .idea/codeStyles)

Then, open Settings > Languages and Frameworks > TypeScript > TSLint and tick enable. Set the package directory to be <projectDirectory>/node_modules/tslint.
![IntelliJ screenshot](https://i.imgur.com/sSqUQQv.png)

### Hook it up to an API

#### Use fetch

Dean has written an [open-source higher-order-component](https://github.com/Dean177/higher-order-async) for async actions.

#### Add an API proxy

If your API sits under a different port or even under a different URL to your app (typically if you're not using the same thing to serve both your app and your API), Node.js can proxy requests to the API for you so that you don't get CORS errors.

See the instructions on the react-scripts README.

## Serve with MVC

### .NET Core

Follow [these instructions](https://docs.microsoft.com/en-us/aspnet/core/spa/react?tabs=visual-studio) from Microsoft. This replaces the 'create-react-app' step above. Then go back to the top of the page to add routing, state management etc.

## Configure Jenkins

The easiest way to do this is to write a Jenkinsfile in the root of your repository.

## Eject from create-react-app

If you find that you need to modify the config and scripts generated by create-react-app after, all, you may want to eject. This removes all dependencies on react-scripts-ts from your project and exposes all the underlying config and scripts for you to edit. Note that this adds a lot of files to your project! The react-scripts README explains what ejecting is in more detail.


**Watch out** - once you've ejected there's no going back except by reverting in Git. So make sure you commit before ejecting and have a clean working tree.
````
yarn eject
````
The office is divided over whether you should eject or not. The most common reason to want to eject is to modify the webpack config, e.g. adding a new loader. In the specific cases of SASS and LESS loaders, there are workarounds described above, but for all other loaders you'll need to eject so that you can modify the webpack config.

