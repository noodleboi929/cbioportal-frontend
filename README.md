# cbioportal-frontend
This repo contains the frontend code for cBioPortal which uses React, MobX and TypeScript. Read more about the architecture of cBioPortal [here](https://docs.cbioportal.org/2.1-deployment/architecture-overview).

## Branch Information
| | main branch | upcoming release branch | later release candidate branch |
| --- | --- | --- | --- |
| Branch name | [`master`](https://github.com/cBioPortal/cbioportal-frontend/tree/master) |  --|  [`rc`](https://github.com/cBioPortal/cbioportal-frontend/tree/rc) |
| Description | All bug fixes and features not requiring database migrations go here. This code is either already in production or will be released this week | Next release that requires database migrations. Manual product review often takes place for this branch before release | Later releases with features that require database migrations. This is useful to allow merging in new features without affecting the upcoming release. Could be seen as a development branch, but note that only high quality pull requests are merged. That is the feature should be pretty much ready for release after merge. |
| Test Status | [CircleCI master workflow](https://circleci.com/gh/cBioPortal/workflows/cbioportal-frontend/tree/master) | -- | [CircleCI rc workflow](https://circleci.com/gh/cBioPortal/workflows/cbioportal-frontend/tree/rc) |
| Live instance frontend | https://frontend.cbioportal.org / https://master--cbioportalfrontend.netlify.app/ | -- | https://rc--cbioportalfrontend.netlify.app |
| Live instance backend | https://www.cbioportal.org / https://master.cbioportal.org | -- | https://rc.cbioportal.org |

Note: you can check the frontend version of the live instance by checking `window.FRONTEND_COMMIT` in the console.

## Run

Make sure you have installed the node version and yarn version specified in
[package.json](https://github.com/cBioPortal/cbioportal-frontend/blob/master/package.json).

> **Tip:**  We recommend that you use [nvm:  Node Version Manager](https://github.com/nvm-sh/nvm) and [yvm:  Yarn Version Manager](https://yvm.js.org/docs/overview) to switch between versions more easily.

> **Windows Tip:** If you are developing on Windows, we recommend that you use [Ubuntu / Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

Remove old compiled `node_modules` if it exists

```
rm -rf node_modules
```

To install all app and dev dependencies
```
yarn install --frozen-lockfile
```

To build DLLs in common-dist folder (must be done prior to start of dev server)
```
yarn run buildDLL:dev
```

To start the dev server with hot reload enabled
```
# set the environment variables you want based on what branch you're branching
# from
export BRANCH_ENV=master # or rc if branching from rc
# export any custom external API URLs by editing env/custom.sh
yarn run start
```

Example pages:
 - http://localhost:3000/
 - http://localhost:3000/patient?studyId=lgg_ucsf_2014&caseId=P04

To run unit/integration tests
```
yarn run test
```

> **Windows Tip:** There is a known solved hiccup running the tests on Ubuntu via Windows Subsystem for Linux (WSL): [#7096](https://github.com/cBioPortal/cbioportal/issues/7096)

To run unit/integration tests in watch mode
```
yarn run test:watch
```

To run unit/integration tests in watch mode (where specName is a fragment of the name of the spec file (before `.spec.`))
```
yarn run test:watch -- --grep=#specName#
```

## Formatting Code with PrettierJS
When you make a git commit, PrettierJS will automatically run *in write mode* on all the files you changed, and make 
formatting adjustments to those entire files as necessary, before passing them through to the commit (i.e. this is a 
"pre-commit git hook"). No action from you is necessary for this. You may observe that your changes don't look exactly 
the same as you wrote them due to formatting adjustments.

When you make a pull request, CircleCI will run PrettierJS *in check mode* on all of the files that have changed between 
your pull request and the base branch of your pull request. If all of the files are formatted correctly, then the
CircleCI `prettier` job will pass and you'll see a green check on Github. But if, for whatever reason, this check *fails*, 
you must run the following command in your cbioportal home directory:
```$bash
yarn run prettierFixLocal
```
This will make PrettierJS run through the same files that CircleCI checks (i.e. all files changed since the base branch)
but *in write mode* and thus adjust those files to have correct formatting. When you make this update, the CircleCI 
`prettier` job should pass. To check if it will pass, you can also run the same command that CircleCI will run:
```$bash
yarn run prettierCheckCircleCI
```


## Changing the URL of API
If the version of the desired API URL is the same as the one used to generate
the typescipt client, one can change the `API_ROOT` variable for development in
[my-index.ejs](my-index.ejs). If the version is different, make sure the API
endpoint works with the checked in client by changing the API URL in
[package.json](package.json) and running:
```
# set the environment variables you want based on what branch you're branching
# from
export BRANCH_ENV=master # or rc if branching from rc
# export any custom external API URLs by editing env/custom.sh
yarn run updateAPI
yarn run test
```

## Check in cBioPortal context
Go to https://cbioportal.org (`master` branch) or https://rc.cbioportal.org/ (`rc` branch)

In your browser console set:
```
localStorage.setItem("localdev",true)
```
This will use whatever you are running on `localhost:3000` to serve the JS (i.e. you need to have the frontend repo running on port 3000). To unset do:
```
localStorage.setItem("localdev",false)
```
or clear entire local storage
```
localStorage.clear()
```
You can also use a netlify deployed cbioportal-frontend pull request for serving the JS:
1. Create the following bookmarklet: 
```
javascript:(function()%7Bvar pr %3D prompt("Please enter PR%23")%3Bif (pr %26%26 Number(pr)) %7B localStorage.netlify %3D %60deploy-preview-%24%7Bpr%7D--cbioportalfrontend%60%3Bwindow.location.reload()%3B %7D%7D)()
```
2. Navigate to the cBioPortal installation that you want to test.
3. Click the bookmarklet and enter your pull request number.

## Run e2e-tests

End-to-end tests can be run against public cbioportal instances or against a local dockerized backend. These two e2e-tests types are referred to as `remote` and `local` types of e2e-tests.

## Run of `remote e2e-tests`

Follow instructions to boot up frontend dev server. This is the frontend that will be under test in the e2e tests (running against production backend/api)

```
cd end-to-end-test

// install deps
yarn 

```

```
yarn run e2e:remote --grep=some.spec* 
```

### Mount of frontend onto HTTPS backend
A custom frontend can be tested against any backend in the web browser using a local node server (command `yarn run start`) and the `localdev` flag passed to th e browser (see section 'Check in cBioPortal context'). For remote backends that communicate over a HTTP over SSL (https) connection (e.g., cbioportal.org or rc.cbioportal.org), the frontend has to be served over SSL as well. In this case run `yarn run startSSL` in stead of `yarn run start`.

## Run of `localdb` e2e-tests
To enable e2e-tests on for features that depend on data that are not included in studies served by the public cBioPortal instance, cbioportal-frontend provides the `e2e local database` (refered to as _e2e-localdb_ or _local e2e_ in this text) facility that allows developers to load custom studies in any backend version used for e2e-tests. CircleCI runs the `e2e-localdb` tests as a separate job.

Files for the local database e2e-tests are located in the `./end-to-end-test/local` directory of cbioportal-frontend. The directory structure of `./end-to-end-test/local` is comparable to that of the `./end-to-end-test/remote` directory used for e2e-tests against remote public cBioPortal instances.

### Running `localdb` e2e-tests for development

1. You need to have Docker installed and running.

2. You need to have the [jq](https://stedolan.github.io/jq/) package installed on your system. E.g. using brew:
   ```brew install jq```

In a terminal, start the frontend dev server

```
export BRANCH_ENV=custom
yarn install --frozen-lockfile // only necessary first time
yarn buildDLL:dev // only necessary first tiem
yarn start
```

3. Install dev dependencies:
```bash
cd end-to-end-test
yarn
```
5. In a second terminal at project root, spinup the backend (api) instance:

```
// if you are running for first time, you will need to build the docker containers.
// Answer yes when it prompts you to do so. This will take at least 20 minutes depending
// on your system speed.
// Once you have done this, you can answer no on subsequent attempts

yarn run e2e:spinup
```

6. When backend instance is operational, you can run tests. Upon executing
the command below, a browser should open and you should see your tests execute.

```
//grep accepts fragments of file name, 
//but you MUST using trailing *
//you need only match the file name, not path

yarn run e2e:local --grep=some.spec*   

```



### Running e2e-localdb tests _CircleCI_ or _CircleCI+PR_ context
E2e-tests on _CircleCI_ and _CircleCI+PR_ context are triggered via _hooks_ configured on GitHub. Configuration of hooks falls beyond the scope of this manual.

#### cBioPortal-backend version
E2e-testing against a local database removes dependence on data provided by public cbioportal instances for testing. This makes it possible to test features for data types that are not provided by public cbioportal instances or test features that depend on a backend feature not yet integrated in  public cbioportal instances. E2e-localdb tests make use of the `BACKEND` environmental variable to test against a specific backend version. Depending on the running context (see section above) setting the `BACKEND` environmental variable is required or optional (see table below).

Requirement for setting the BACKEND variable depends on the context of the job:

| **context**             | **BACKEND var** | **comments** |
|------------------------ | ----------------- | ------------ |
| _Local_                   | mandatory        | |
| _CircleCI_                | mandatory for feature branches  | not for `master` or `rc` builds |
| _CircleCI+PR_   | optional        | 1. When specified, GitHub PR must be of 'draft' state.<br>2. When not-specified, backend branch will mirror frontend branch (rc frontend vs. rc backend) |

The value of the `BACKEND` variable must be formatted as `<BACKEND_GITHUB_USER>:<BACKEND_BRANCH>`. For example, when the /env/custom.sh file contains `export BACKEND=thehyve:uwe7872A` this is interpreted as a requirement for the commit `uwe7872A` of the _github.com/thehyve/cbioportal_ repository.

#### BACKEND environmental variable in _CircleCI+PR_ context
Using the `BACKEND` variable e2e-localdb tests can be conducted against any backend version. This poses a risk when testing in _CircleCI+PR_ context, i.e., tests show up as succesful but should have failed against the backend version that compatible with the target cbioportal-frontend branch. To guard against this and prevent merging of incompatible branches into cbioportal-frontend the e2e-localdb tests enforce the use of _draft_ pull requests (see [here](https://help.github.com/en/articles/creating-a-pull-request) for more info). When a cBioPortal backend version is specified (i.e., may require a not yet merged backend branch) and the branch is part of a pull request, the pull request must be in state _draft_. Only when the `BACKEND` variable is not defined a (non-_draft_) e2e-localdb tests will be conducted on branches that are part of pull requests. Needles to say, pull request should for this and other reasons only be merged when the e2e-localdb tests succeed!

When the `BACKEND` variable is not set, the backend version will be set to the target branch of the pull request, i.e. a pull request to 'rc' branch will be tested against the 'rc' branch of the backend.

#### Writing e2e tests
Some random remarks on e2e-test development
- Screenshot tests and DOM-based tests are contained in files that end with *.screenshot.spec.js or *.spec.js, respectively.
- Screenshot tests should only be used to test components that cannot be accessed via the DOM.
- Screenshots should cover as little of the page possible to test behavior. Larger screenshots will make it more likely the screenshot will need to be updated when an unrelated feature is modified. 
- For DOM selection webdriverio selectors are used. Although overlapping with jQuery selectors and both using the '$' notation these methods are not equivalent. See [this link](https://blog.kevinlamping.com/selecting-elements-in-webdriverio/) for more information on webdriverio selectors.
- At the moment of this writing webdriverio v4 is used. Selectors for this version are not fully compatible with webdriverio v5. For instance, selecting of a element with id _test_ `$('id=test')` does not work; this should be `$([id=test])`. I was not able to find documentation of v4 selectors.
- e2e tests use the node.js _assert_ library for assertions. It has an API that is different API from _chai_ assertion library used in unit tests of cbioportal-frontend! See the [assert documentation](https://nodejs.org/api/assert.html) for information on _assert_ API.
- Screenshots for failing tests are placed in the `screenshots/diff` and `screenshots/error` folders. These are a valuable asset to debug tests on when developing in _Local_ context.
- A great tool for test development is the ability of webdriverio to pause execution with `browser.debug()`. When placing this command in the test code and using the `run_local_screenshot_test.sh` facility, a prompt becomes available on the command line that allows testing of DOM selectors in the webbrowser. In addition, the browser window is available on screen; opening of DevTools allows to explore the DOM and observe the effects of webdriverio commands on the command line.
- Although webdriverio takes asynchronous behavor of webbrosers into account it does not defend against asynchronous behavior of specific web components (e.g., database access). Not taking this asynchronicity into account will result in `flaky` tests. Typically, flaky test run well on the local system used for development (that has plenty of free resources at moment of test), but fail often on a CI system. Often this is the result of longer times needed page/component update causing tests to fail because the test evaluates a condition before it is loaded. In webdriverio the `waitForExist()`, `waitForVisible()` and `waitFor()` method should be used to pause test execution until the page has been updated. Sometimes it is needed to wait for the appearance of a DOM element which presence is tested.
```javascript
browser.waitForExist('id=button');
assert($('id=button'));
```
- Reference screenshosts that are created on host system directly (not in dockerized process) differ from screenshots produced by the dockerized setup (e.g., on CircleCI) and cannot be used as references

#### Create new e2e-test
Making e2e-tests follows the current procedure for the e2e-tests:
1. Create junit test file and place in the `./end-to-end-test/local/specs` or `./end-to-end-test/remote/specs` directory.
2. [Optional] Add a folder with an uncompressed custom study in the `./end-to-end-test/local/studies` directory.

#### Random notes
* Study_es_0 is imported by default.
* Gene panel and gene set matrix data of custom studies must comply with gene panel/sets imported as part of study_es_0.
* Imports of custom seed data for gene panels and gene sets are not implemented at the moment of this writing.
* In order to minimize time of local database e2e-tests the size of custom studies should be kept as small as possible.
* When developing in _Local_ context port 8081 can be used to access the cbioportal instance ('http://localhost:8081/cbioportal').

#### Debugging help
Here are some errors that have been encountered and are hard to debug.

##### "boundingRects.reduce is not a function"
This error occurs when an e2e test tries to take a screenshot of an element that doesn't exist.

##### "There are some read requests waitng on finished stream"
This error occurs in CircleCI when the reference screenshot file is somehow corrupted. It can be fixed by deleting and updating the reference screenshot.


## Workspaces

We are utilizing `yarn workspaces` to maintain multiple packages in a single repo (monorepo). The monorepo approach is an
 efficient way of working on libraries in the same project as the application that is their primary consumer. 

The `cbioportal-frontend` is the main web application workspace. It is used to build and deploy the cBioPortal frontend webapp. 
Workspaces under `packages` directory are separate modules (npm packages) designed to be imported by `cbioportal-frontend` workspace as well as by external projects.
 __Please note:__ `config` and `typings` directories under the `packages` directory are NOT workspaces or packages. They are intended to share common settings among all packages under the `packages` directory.

### Adding a new workspace

To add a new workspace, create a new directory under `packages` and add a `package.json` file. (See `cbioportal-frontend-commons` workspace for example configuration).

The recommended way to add a new dependency to an existing workspace is to run `yarn workspace <workspace name> add <package name>` instead of just `yarn add <package name>`. For example, run `yarn workspace cbioportal-frontend add lodash` instead of `yarn add lodash`. 
Similarly, to remove a package, run `yarn workspace <workspace name> remove <package name>`.

### Tips for dependency management

Please abide by the following rules for importing dependencies in a monorepo:   

1. If you are working on `cbioportal-frontend` repository, import modules from packages using the package's alias:

```
// CORRECT, uses alias:
import {something} from 'cbioportal-frontend-commons'

// INCORRECT, uses relative paths:
`import {something} from ../../packages/cbioportal-frontend-commons/src/lib/someLib`
```

2. When working on a package, never import custom code from outside that package unless you really intend for that package to be a dependency.  For example, commons packages should not import from the main cbioportal project.  

3. Avoid circular dependencies at all costs. For example, while it is okay to import a module from `cbioportal-frontend-commons` in `react-mutation-mapper`, there should not be any imports from `react-mutation-mapper` in `cbioportal-frontend-commons`. If you happen to need some component from from `react-mutation-mapper` in `cbioportal-frontend-commons`, consider moving that component into `cbioportal-frontend-commons` package.

### Updating existing packages

Remember that the packages are used by other projects and compatibility needs to be carefully managed. 

When you update code under packages a new version of changed packages automatically published once the code is merged to master. However, in a rare case when you would like to set a custom package version, you can run
 
```
yarn run updatePackageVersion
```

Alternatively you can manually set a custom version. When updating manually you should update the version number in the corresponding `package.json` as well as the dependencies of other packages depending on the package you update. For example if you update the `cbioportal-frontend-commons` version from `0.1.1` to `0.1.2-beta.0`, corresponding `cbioportal-frontend-commons` dependency in the `package.json` for `react-mutation-mapper` and `cbioportal-frontend` should also be updated to the new version.

Note that when setting a custom version if you want the next published package version to be, for example, `1.0.6`, then you should set the new version to `1.0.6-beta.1` or a similar prerelease version. If you set the custom version to `1.0.6`, the next published version will be `1.0.7` not `1.0.6`. This is because the auto publish script runs in any case to detect changes in all packages including custom versioned packages.

#### Update API clients

```
yarn run updateAPI
```

## Components

Components under `packages` should only depend on either external node modules or workspaces under `packages`.
Please make sure to not introduce any dependencies from `cbioportal-frontend` workspace when updating or adding new files under `packages`.

### cbioportal-frontend-commons

[cbioportal-frontend-commons](https://www.npmjs.com/package/cbioportal-frontend-commons/) is a separate public npm library which contains basic utility functions and components.
 
### react-mutation-mapper

[react-mutation-mapper](https://www.npmjs.com/package/react-mutation-mapper/) is a separate public npm library that contains the Mutation Mapper and related components.
