

# Setup dev environment

To run `<project name>` you will need a few things on your machine. Follow the steps below.

## Installations

*   Install [Node JS](https://nodejs.org/en/download/)
    
*   Install an IDE (preferably [VS Code](https://code.visualstudio.com/))
    
*   Install git (if you don't already have it on your machine).
    
*   Install [Yarn](https://classic.yarnpkg.com/en/docs/install/#mac-stable) package manager globally on your machine. For Mac users, it is recommended to use [Homebrew](https://brew.sh): `brew install yarn`. If you wish to install Yarn using NPM, read ahead first about configuring your NPM global packages directory.
    

## Clone and configure the repo on your machine

*   Go ahead and install the latest Swimm app from [The download page](https://swimm.io/download/).
    
*   Choose a local "workspace" directory for Git repositories. For example, `$HOME/dev`.
    
*   On [GitHub](https://github.com/swimmio), click your user profile icon on the top right corner, choose "Settings" -> "Developer settings" -> "Personal access tokens". Click "Generate new token", and check "repo" and "user". When cloning Swimm repositories via HTTP (`git clone https://github.com/swimm/something` as opposed to `git clone git@github.com:swimm/something`), your username is your Swimm email address, and your password is the newly generated token.
    
*   Clone the main Swimm repository by running (in your workspace directory): `git clone https://github.com/swimm/swimm.git`
    
*   Within the repository directory, run `yarn install` to install the project's dependencies.
    
*   On non-Windows machines, configure your NPM global packages directory so that installing packages globally will not require root permissions:
    
    *   `mkdir ~/.npm-global`
        
    *   `npm config set prefix '~/.npm-global'`
        
    *   Edit your `~/.zshrc` or `~/.bashrc` file, depending on which shell you use (`echo $SHELL` to find out), and add this at the end of the file: `export PATH=$PATH:$HOME/.npm-global/bin`. For this change to take effect, run `exec $SHELL`, or open a new terminal window.
        
*   Install some useful NPM packages globally by running: `npm i -g cross-env firebase-tools concurrently ncu` .
    

## OSX / Linux additional steps

*   Create a new file `~/swimm-dev-env.sh`, and insert the following content:
    
    *   `export SWIMM_PROJECT_DIR="path-to-swimm-dir"`, where _path-to-swimm-dir_ is the path to the main Swimm repository, for example `export SWIMM_PROJECT_DIR="$HOME/dev/swimm"`. Use `$HOME` rather than `~`.
        
    *   `alias swimmdev="node <path-to-swimm-dir>/dist/cli/development/swimm-cli-bundled.js"` This alias runs the CLI from within your development repository, without deploying it as a local package. Note that this alias will work only after a successful cli TS compiling was done.
        
*   At the bottom of your `~/.zshrc` or `~/.bashrc` file, add `. $HOME/swimm-dev-env.sh`. This will execute your environment file, and retain the defined environment variables and aliases in your terminal session.
    
*   Setup `swimm-app-dev`:
    
    *   `sudo ln -sf $SWIMM_PROJECT_DIR/devops/dev-setup/swimm-app-dev.sh /usr/local/bin/swimm-app-dev`
        
    *   `chmod +x /usr/local/bin/swimm-app-dev`
        
    *   `chmod +x $SWIMM_PROJECT_DIR/devops/dev-setup/swimm-app-dev.sh`
        
    *   Tell git to honor the file mode and not remove the execution permissions:
        
        *   `git config --global core.fileMode false`
            

## Linux (Ubuntu)

### electron builder prerequisite

> Error! Cannot execute command (...) "need executable 'ar' to convert dir to deb"(...)

*   For electron builder to run, the package `binutils` needs to be installed. Although it should be included when installing electron on the machine/VM - it sometimes fails
    
*   To avoid building issues, please run `sudo apt-get install binutils` to install this dependency before trying to build the app
    
*   further information can be found in these links:
    
    *   [https://github.com/electron-userland/electron-builder/issues/3992](https://github.com/electron-userland/electron-builder/issues/3992)
        
    *   [https://github.com/electron-userland/electron-build-service/commit/9bac60e6c3db1d013cd60f737f2004793489e3d0](https://github.com/electron-userland/electron-build-service/commit/9bac60e6c3db1d013cd60f737f2004793489e3d0)
        

## Windows additional steps

TBD

## Configure Git

Run `git config --global user.name "Don Joe"`, use your own full name with proper capitalization.

Run `git config --global user.email yourusername@swimm.io`, use your own Swimm email address.

## Run the tests 🧪

To run all the tests, run:

```
$ yarn test
```

To run subsets of the tests - you can use `yarn test:<name>` - look at `package.json` for additional info. For example:

```
$ yarn test:cli
$ yarn test:adapters
```

### E2E

you can run the e2e tests by using: `yarn test:e2e`

read all about it in [E2E Testing](e2e-testing.SOLx67J3ZHuGvxD98rj9.sw.md)

## Run a local Swim development server

**WebApp:**

[Swimm in the WEB](swimm-in-the-web.FEnqEBAHprn2iRKzjf1S.sw.md)

**Local App:**

Run `yarn dev`. This will run a local server and open Electron for you. Fill in the details:

*   Full name: your full name, or the name of your pet or a celebrity, or alternatively a celebrity pet, if you know one.
    
*   Email address: your actual Swimm email address, but add a plus-suffix to your username, so for example if your username is `shimshon` you can choose for example `shimshon+dev@swimm.io`. You will receive emails sent to this address to your regular inbox, but you can define a rule for emails where the recipient address is `shimshon+dev`.
    
*   Workspace name: just a name of your choosing, for your development workspace within your local Swimm deployment.
    

Note that this information will be retained, so if you take your server down and then launch it again, you will not have to fill in these details again.

## Scripts worth mentioning ⚡️✨

Serve your code with electron's hot reloader. The code is not minified and the env is development.

```
$ yarn dev
```

Bundle the app (with production env) without signing and without creating an installer.

```
$ yarn build
```

Pack for Production. This will generate installers and sign+notarize our app.

```
$ yarn pack
```

Build for dev purposes - this will build the app with the staging environment. Should be used for testing and while developing.

```
$ yarn build:dev
```

## Debugging Electron

[Debugging Electron](app://open_new_swimm_window/#/repos/veezvxCuzpPrRLLXWD2E/file?filePath=docs/debugging-electron.md)

## **IDE Plugins dev environment:**

1.  Make sure you have installed on your machine:
    
    1.  **Intellij - [https://intellij-support.jetbrains.com/hc/en-us](https://intellij-support.jetbrains.com/hc/en-us)**
        
        1.  and Yes. The free version is totally fine
            
    2.  VSCode - [https://code.visualstudio.com/](https://code.visualstudio.com/)
        
2.  Clone our repos:
    
    1.  [https://github.com/swimmio/swimm-IntelliJ-plugin](https://github.com/swimmio/swimm-IntelliJ-plugin) - our intelliJ plugin
        
    2.  Clone [https://github.com/swimmio/swimm-vscode](https://github.com/swimmio/swimm-vscode) - our vscode plugin
        
3.  Make sure you can run and debug the plugins
    
    1.  IntelliJ - follow the instructions in `https://app.swimm.io/workspaces/aRvMqc0yWAVcJlLN944D/repos/RZB4LncehkScUjRv6Dnj/playlists/vwgus/steps/6?branch=main`
        
    2.  VSCode - follow the instructions in **TBD** - (talk to @Divo if the doc doesn't exist)
        

# Congrats

You now have your dev environment ready 🎉

<br/>

This file was generated by Swimm. [Click here to view it in the app](https://app.swimm.io/repos/veezvxCuzpPrRLLXWD2E/docs/ceA0GzroQRhQmjVJ3nwK).