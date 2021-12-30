
We use [Cypress](https://www.cypress.io/) for our E2E tests. Cypress is a purely JavaScript-based front end testing tool built for the modern web. It aims to address the pain points developers or QA engineers face while testing an application. Cypress is a more developer-friendly tool that uses a unique DOM manipulation technique and operates directly in the browser.

How Does It Work
----------------

Cypress runs on Chrome in Incognito mode (to clear local app storage in the browser between each test runs). We have a special [test organization](https://github.com/swimm-test) with several repos that we can use freely for these tests. Do not use repos from `swimmio` organization, to avoid mixing up our codebase with test data.

<br/>

## Configure Automation User and Password

We use [dotenv](https://github.com/motdotla/dotenv#readme) to keep the testing user. Dotenv reads dynamically a `.env` file and reads from it the testing credentials: `AUTOMATION_USER`[<sup id="1CwlcU">↓</sup>](#f-1CwlcU), `AUTOMATION_PASS`[<sup id="Z1N1OnT">↓</sup>](#f-Z1N1OnT) and `AUTOMATION_GITHUB_TOKEN`[<sup id="Z2nnadE">↓</sup>](#f-Z2nnadE) . To configure :

1.  Create a file named `.env` in the root folder of Swimm **(this file is ignored by git and should never be pushed to the repo as it contains auth tokens)**.
    
2.  The `.env` file should look like:
    
    ```
    AUTOMATION_USER="<enter the testing user here>"
    AUTOMATION_PASS="<enter the testing password here>"
    AUTOMATION_GITHUB_TOKEN="<enter token here>"
    AUTOMATION_TOKEN_DB_NAME=swimm_state_dev
    ```
    
3.  Talk to @Dalko to get the user and password
    

## Running E2E Tests Locally

<br/>

Make sure you local web app is running (`yarn web`) and simply run `yarn` `test:e2e`[<sup id="SEBiw">↓</sup>](#f-SEBiw) . Cypress will run on the web app that is running on `localhost:5000`. When Cypress UI opens, click `Run X integration specs`
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 package.json
```json
⬜ 200        "test:unit": "yarn test:release",
⬜ 201        "test:security": "concurrently --raw -k -s first \"wait-on http://localhost:4000 && cross-env TEST_MODE=SECURITY jest --config jest.config.js firestore_rules.spec.js\" \"yarn run firebase:emulators:firestore\"",
⬜ 202        "test:push": "swimm verify && yarn cli:bundle:test && jest --config jest.config.js",
🟩 203        "test:e2e": "cypress open",
⬜ 204        "test:e2e:ci": "cypress run",
⬜ 205        "lint": "eslint --no-error-on-unmatched-pattern --ext .js,.vue,.ts .",
⬜ 206        "lint:fix": "eslint --fix --no-error-on-unmatched-pattern --ext .js,.vue,.ts .",
```

<br/>

## Writing Tests

There are many Cypress commands, you can follow their [docs](https://docs.cypress.io/api/table-of-contents) to see what each command does. Here are some basic and common commands:

`cy.get('.submenu'); // select any element using CSS selectors`

`cy.get('div').contains('This is some element'); // select any div with a certain text`

`cy.get('input').type('This is an input text'); // type inside an input`

`cy.get('input').focus(); // focus on an element`

`cy.focused(); // select the currently focused element`

`cy.get('.element').click(); // click on an element`

`cy.log('Going to click on a button'); // log some text during the test`

Notice that Cypress automatically waits for elements to appear on the screen so no need to add code that waits for elements to appear.

## Assertions

Each selector or action that doesn't appear on the screen will fail the test, but you can also verify that some things behave the way we want, with `.should`. For example:

`cy.get('.element').should('be.visibile')`

`cy.get('button').should('have.class', 'primary')`

`cy.get('.link').should('have.text', 'Link')`

`cy.get('.submenu').should('have.length', 4)`

`cy.get('.element').should('not.exist')`

## Elements Selection

Although we can easily select elements that we want to test using class names or tag names, or even by order (e.g. `:nth-of-type(3)`). Doing so can make the tests easily break if someone changes the design of a certain component. A better approach would be to use unique data attributes, in our case we use `data-testid` .

<br/>

For example, in this component we have 3 Add buttons which are quite similar. Selecting the correct Add button in tests can be a bit tricky. In addition we don't have anything unique about each instance of the component. A bad approach would be to select the first/second/third element. A better approach would be to use `testid`[<sup id="20QLJ">↓</sup>](#f-20QLJ) data attribute and giving each Add element a unique functional identifier.
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 src/app/common/components/organisms/RepoWorkspace.vue
```vue
⬜ 102                <Icon name="playlist" class="header-icon" />
⬜ 103                <h4 class="header-title system-body">Playlists ({{ Object.keys(playlistsFilteredByPath).length }})</h4>
⬜ 104                <div v-if="canModifyResources && canCreatePlaylist" @click="goToPage(`${getRepoPath()}/playlists/new`)">
🟩 105                  <Action secondary no-padding size="small" data-testid="add-playlist-button">
⬜ 106                    <Icon name="add" class="bold" />
⬜ 107                  </Action>
⬜ 108                </div>
```

<br/>

Now it's really easy to select this element by using `getByTestId`[<sup id="ZP9ydn">↓</sup>](#f-ZP9ydn) command. The test won't break due to class naming, change of order, or any other renaming, but only due to functionality - which is exactly what we want.
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 e2e/specs/playlist-flow.e2e.js
```javascript
⬜ 10       });
⬜ 11     
⬜ 12       it('should create a new playlist with an exercise, video and a doc', () => {
🟩 13         cy.getByTestId('add-playlist-button').click();
⬜ 14         cy.get('.edit-playlist-form input[type="text"]').type(playlistName);
⬜ 15     
⬜ 16         cy.get('[contenteditable="true"]').type('This is a playlist created by Cypress');
```

<br/>

## Custom Commands

You can add your own custom commands for repetitive actions during tests.

<br/>

For example, here we added the `cy.` `login`[<sup id="1LUFif">↓</sup>](#f-1LUFif) custom command:
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 e2e/support/commands.js
```javascript
🟩 23     Cypress.Commands.add('login', (username = Cypress.env('AUTOMATION_USER')) => {
🟩 24       if (Cypress.env('AUTOMATION_GITHUB_TOKEN')) {
🟩 25         setGithubToken(Cypress.env('AUTOMATION_GITHUB_TOKEN'));
🟩 26       }
🟩 27       cy.get('input[type="email"]').should('be.empty').type(username);
🟩 28       cy.get('input[type="password"]').should('be.empty').type(Cypress.env('AUTOMATION_PASS'));
🟩 29       cy.get('.login-button').click();
🟩 30     });
```

<br/>

some of the commands we have in the `📄 e2e/support/commands.js` file are:

*   login
    
*   logout
    
*   go to repo
    
*   go to branch
    
*   go to workspace
    

## Test Examples

Let's take a look at a simple test:

<br/>

In this test we are in a flow where a playlist was created and we want to view the playlist to make sure it was created correctly. Notice the assertions (i.e. `should`[<sup id="ZAI7Ol">↓</sup>](#f-ZAI7Ol) that make sure the element we want to test matches what we expect.
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 e2e/specs/playlist-flow.e2e.js
```javascript
⬜ 41         cy.get('.card').contains(playlistName).should('be.visible');
⬜ 42       });
⬜ 43     
🟩 44       it('should view the created playlist', () => {
🟩 45         cy.get('.card').contains(playlistName).click();
🟩 46         cy.get('h1').should('have.text', playlistName);
🟩 47     
🟩 48         cy.log('Step #1');
🟩 49         cy.get('button').contains('Go to Step #1').click();
🟩 50         cy.get('h1').should('have.text', 'Test Doc 1');
🟩 51         cy.get('.icon-check').should('be.visible');
🟩 52     
🟩 53         cy.log('Step #2');
🟩 54         cy.get('button').contains('Go to Step #2').click();
🟩 55         cy.get('h1').should('have.text', 'Exercise 1');
🟩 56         cy.get('.icon-check').should('be.visible');
🟩 57     
🟩 58         cy.log('Step #3');
🟩 59         cy.contains('The Trashmen - Surfin Bird').click();
🟩 60         cy.get('.icon-check').should('be.visible');
🟩 61     
🟩 62         cy.log('Finish playlist');
🟩 63         cy.get('button').contains('Finish Playlist').click();
🟩 64         cy.get('div').contains('This is the end of the playlist').should('be.visible');
🟩 65       });
⬜ 66     
⬜ 67       it('should add a doc to the playlist', () => {
⬜ 68         cy.get('.crumb-name').contains('Test Workspace').click();
```

<br/>

Here's a more complex test:

<br/>

In this test we use `type`[<sup id="1N9cuY">↓</sup>](#f-1N9cuY) to mimic keyboard clicks (in this case, arrow keys).
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 e2e/specs/doc-flow.e2e.js
```javascript
⬜ 245            .should('have.text', 'we are adding smart text in a text block  EXTRA   and add new token  EXTRA  ');
⬜ 246        });
⬜ 247    
🟩 248        it('traverse the edit page', () => {
🟩 249          cy.get('input.headline').should('have.value', testTitle).should('have.focus');
🟩 250          cy.focused().type('{downarrow}');
🟩 251          cy.get('.ProseMirror').eq(0).should('have.focus');
🟩 252          cy.focused().type('{downarrow}{downarrow}');
🟩 253          cy.focused().type('{downarrow}');
🟩 254          cy.get('.ProseMirror').eq(1).should('have.focus');
🟩 255          cy.focused().type('{downarrow}{downarrow}');
🟩 256          cy.focused().type('{downarrow}');
🟩 257          cy.get('.ProseMirror').eq(2).should('have.focus');
🟩 258          cy.focused().type('{downarrow}{downarrow}');
🟩 259          cy.focused().type('{downarrow}');
🟩 260          cy.get('.comment-editor > .editor > .editor__content > .ProseMirror').eq(0).should('have.focus');
🟩 261          cy.focused().type('{downarrow}{downarrow}');
🟩 262          cy.focused().type('{downarrow}');
🟩 263          cy.get('.comment-editor > .editor > .editor__content > .ProseMirror').eq(1).should('have.focus');
🟩 264          cy.focused().type('{downarrow}{downarrow}');
🟩 265          cy.focused().type('{downarrow}');
🟩 266          cy.get('.comment-editor > .editor > .editor__content > .ProseMirror').eq(2).should('have.focus');
🟩 267          cy.focused().type('{downarrow}{downarrow}');
🟩 268          cy.focused().type('{downarrow}');
🟩 269          cy.get('.ProseMirror').eq(6).should('have.focus');
🟩 270          cy.focused().type('{downarrow}{downarrow}');
🟩 271          cy.focused().type('{downarrow}');
🟩 272          cy.get('.ProseMirror').eq(7).should('have.focus');
🟩 273        });
⬜ 274    
⬜ 275        it('add to the document', () => {
⬜ 276          cy.focused().type('New line');
```

<br/>

## Test Suites

Each chunk of tests is called a Test Suite. We now have several tests suites that each of them is responsible to a certain set of features in the app. For example, `📄 e2e/specs/autosync-flow.e2e.js` is responsible for for all the autosync functionality.

Each test suite should start "clean", meaning it should start from a certain baseline, be it the workspace page, repo page, or anything else. Consider the fact that in the CI all tests suites will run one after the other, so by assuming the starting point of the suite - it can easily break the entire suite.

It's a good practice to split tests into suites, due to the fact that they have the ability to run in parallel. So more suites = more paralyzation possible.

Also, each test should follow the Arrange, Act, Assert (AAA) guidelines of writing tests. Each test should be incapsulate as much as possible and should not relay on previous tests. Each test should not fail the tests that come after it because of incorrect arrangement.

## Debugging

When running tests locally, you can see the test running in real time, and when hovering over each step you can see a screenshot of the state of app during the test. This is actually called "Time Travel".

You can also click on any request to the server and Cypress will log the request details into the console.

Also, when you click on any step, Cypress will highlight the element that was selected in that step.

### CI

<br/>

Currently automation tests run in the CI after every publish to staging (i.e. a merge of a PR and deploy to staging).
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 .github/workflows/test_e2e.yml
```yaml
⬜ 3      on:
⬜ 4        # Run after our staging publish workflow
⬜ 5        workflow_run:
🟩 6          workflows: [ "Publish Swimm Web Staging" ]
⬜ 7          types:
⬜ 8            - completed
⬜ 9        # Run after a review was submitted (we'll check it was "approved" later)
```

<br/>

You can also run locally before merge by running `yarn` `test:e2e:ci`[<sup id="Z12WCiV">↓</sup>](#f-Z12WCiV)
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 package.json
```json
⬜ 201        "test:security": "concurrently --raw -k -s first \"wait-on http://localhost:4000 && cross-env TEST_MODE=SECURITY jest --config jest.config.js firestore_rules.spec.js\" \"yarn run firebase:emulators:firestore\"",
⬜ 202        "test:push": "swimm verify && yarn cli:bundle:test && jest --config jest.config.js",
⬜ 203        "test:e2e": "cypress open",
🟩 204        "test:e2e:ci": "cypress run",
⬜ 205        "lint": "eslint --no-error-on-unmatched-pattern --ext .js,.vue,.ts .",
⬜ 206        "lint:fix": "eslint --fix --no-error-on-unmatched-pattern --ext .js,.vue,.ts .",
⬜ 207        "prepare": "husky install"
```

<br/>

### Add a New Suite to the CI

<br/>

When testing are running back to back (vs in parallel) it will be running in the order state in the `testFiles`[<sup id="1cMpX0">↓</sup>](#f-1cMpX0) array
<!-- NOTE-swimm-snippet: the lines below link your snippet to Swimm -->
### 📄 cypress.json
```json
⬜ 2        "projectId": "swimm",
⬜ 3        "chromeWebSecurity": false,
⬜ 4        "pluginsFile": "e2e/plugins/index.js",
🟩 5        "testFiles": [
⬜ 6          "**/basic-flow.e2e.js",
⬜ 7          "**/backwards-compatibility-flow.e2e.js",
⬜ 8          "**/doc-flow.e2e.js",
```

<br/>

## Q&A

**Q: Why are some requests always pending?**

A: Since we use Firestore and it uses Websockets, some requests to the server may always appear as pending. This is not an issue and doesn't interrupt the tests.

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimmio-content/o/repositories%2FveezvxCuzpPrRLLXWD2E%2F4c97da85-e94c-46b6-9202-6435b39b964a.png?alt=media&token=ef6865e4-e448-4f71-b627-aa0948197b27" style="width:'50%'"/></div>

<br/>

**Q: My test fails due to timeout. What should I do?**

A: Every command has a timeout of 10 seconds, which should be more than enough for every action in the app. In some places where make continuous operations, we show a loader that indicates that this operation might take a few seconds. When a loader appears in such operation we use `cy.waitForLoader` command that expects a loader, and immediately after it no longer appears the tests will continue to run. Every other case is considered a performance issue and should be resolved regardless the automation tests.

<br/>

**Q: Why should I avoid using `cy.visit`[<sup id="fDlyI">↓</sup>](#f-fDlyI) ?**

A: It should be used once when opening the app, because it uses the browser's navigation bar, meaning it will refresh the page and the app, and will load all the needed data from scratch - which will slow the test dramatically. A better approach would be to mimic user behavior and navigate inside the app using the breadcrumbs, or any other button that takes from one page to another. This will not require any unnecessary fetching and reloading of the app's data.

<br/>

<!-- THIS IS AN AUTOGENERATED SECTION. DO NOT EDIT THIS SECTION DIRECTLY -->
### Swimm Note

<span id="f-Z2nnadE">AUTOMATION_GITHUB_TOKEN</span>[^](#Z2nnadE) - "e2e/support/commands.js" L24
```javascript
  if (Cypress.env('AUTOMATION_GITHUB_TOKEN')) {
```

<span id="f-Z1N1OnT">AUTOMATION_PASS</span>[^](#Z1N1OnT) - "e2e/support/commands.js" L28
```javascript
  cy.get('input[type="password"]').should('be.empty').type(Cypress.env('AUTOMATION_PASS'));
```

<span id="f-1CwlcU">AUTOMATION_USER</span>[^](#1CwlcU) - "e2e/support/commands.js" L23
```javascript
Cypress.Commands.add('login', (username = Cypress.env('AUTOMATION_USER')) => {
```

<span id="f-fDlyI">cy.visit</span>[^](#fDlyI) - "e2e/support/commands.js" L33
```javascript
  cy.visit('/');
```

<span id="f-ZP9ydn">getByTestId</span>[^](#ZP9ydn) - "e2e/specs/playlist-flow.e2e.js" L13
```javascript
    cy.getByTestId('add-playlist-button').click();
```

<span id="f-1LUFif">login</span>[^](#1LUFif) - "e2e/support/commands.js" L23
```javascript
Cypress.Commands.add('login', (username = Cypress.env('AUTOMATION_USER')) => {
```

<span id="f-ZAI7Ol">should</span>[^](#ZAI7Ol) - "e2e/specs/playlist-flow.e2e.js" L41
```javascript
    cy.get('.card').contains(playlistName).should('be.visible');
```

<span id="f-SEBiw">test:e2e</span>[^](#SEBiw) - "package.json" L203
```json
    "test:e2e": "cypress open",
```

<span id="f-Z12WCiV">test:e2e:ci</span>[^](#Z12WCiV) - "package.json" L204
```json
    "test:e2e:ci": "cypress run",
```

<span id="f-1cMpX0">testFiles</span>[^](#1cMpX0) - "cypress.json" L5
```json
  "testFiles": [
```

<span id="f-20QLJ">testid</span>[^](#20QLJ) - "src/app/common/components/organisms/RepoWorkspace.vue" L105
```vue
              <Action secondary no-padding size="small" data-testid="add-playlist-button">
```

<span id="f-1N9cuY">type</span>[^](#1N9cuY) - "e2e/specs/doc-flow.e2e.js" L250
```javascript
      cy.focused().type('{downarrow}');
```

<br/>

This file was generated by Swimm. [Click here to view it in the app](https://app.swimm.io/repos/veezvxCuzpPrRLLXWD2E/docs/SOLx67J3ZHuGvxD98rj9).