# Part 1: Main Tools

These are the main tools we'll be using for Angular development. Note that these could be swapped out to a certain extent based on need and personal preference but this should demonstrate how to properly configure things to make your life easier.

## Git
We'll be using git for this demonstration but this could be adapted to other source control systems like SVN or VSTS. More specifically, we'll be using GitBash for this demonstration but you can use a different commandline tool if you'd prefer. GitBash provides more functionality as well as better command history and the more universal *nix tooling (like 'ls' instead of 'dir').

1. Install git 
    + [Git Website](https://git-scm.com/download/win)
        + __Make sure to install GitBash with it__
2. Configure git - This is where we start the true environment configuration. Note that we'll be setting global configuration values but by omitting the "--global" we would be able to configure these on a per-project/workspace basis.
    - For now we'll just set a couple configuration values and later on we'll learn how to customize it further once all of our main tools are in place.
        - The basic command for setting a git configuration variable is structured as follows:
            ```
            git config [--global] section.variable <value>
            ```
        - Let's set our name that will appear on our repository actions. In GitBash, execute the following command:

            ```
            git config --global user.name "Sean Merritt"
            ```
            - If you'd rather this not be global, make sure you've navigated to your workspace and then execute the same command sans "--global"
        - We'll do the same for our email:

            ```
            git config --global user.email "sean.merritt@us.sogeti.com"
            ```
    

## VSCode
VSCode will be the editor of choice for this demonstration but you could also use something else if you wanted. Coming from .NET development, you might want to use Visual Studio or, if you'd rather not use Microsoft products, Sublime Text or Notepadd++ are great.

1. Install VSCode

    + Download from [VSCode Site](https://code.visualstudio.com)
    + Run the installer
    
2. Verify VSCode was added to the path.
    + Execute from command the following code
        ```
            code --help
        ```
    + You should see output from VSCode's help
        - __Add the path to where you installed VSCode if you don't see the help output.__
    
    + (Optional) Modify file exclude list to keep VSCode's explorer clean by hiding files you don't care about.
    
        1. File>Preferences>Settings (or ctrl-comma)
        2. Add the following code to the User Settings tab. If there are already settings there, simply add to them.
        
            ```
                "files.exclude": {
                    "**/.git": true,
                    "**/.svn": true,
                    "**/.hg": true,
                    "**/CVS": true,
                    "**/.DS_Store": true,
                    
                    // Angular
                    "**/*.js.map": true,
                    "node_modules": true,
                    // The following will only hide a js file if there is a 
                    // corresponding ts file. This will exclude transpiled files.
                    "**/*.js": { "when": "$(basename).ts"}
                }
            ```
## Node
Self-explanatory, we are building on top of Node so you'll need to install Node. This can be accomplished by downloading the appropriate installer for your Operating System. Note that it's possible to download just the binaries if you don't have administrative rights which the installer needs.

1. Install NodeJs
    + Download LTS from [NodeJs Website](https://nodejs.org/en/). You can either use the installer or download the binaries.
        + (installer) Run it as admin
        + (binary) Extract files to install location
            + Manually add the path to Node to your environmental variables.    
                ``` 
                rundll32 sysdm.cpl,EditEnvironmentVariables 
                ```
2. Verify Node was added to the path.
    + Execute from command the following code
        ```
            node -v
        ```
    + You should see NodeJs version information.
        - __Add the path to where you installed NodeJs if you don't see the help output.__

## Yarn
Node comes packaged with the NPM tool but there are a couple downsides to it at this point in time.

1. In order to ensure that all developers are *actually* using the same versions of packages, additional steps are required with npm. Specifically, you must shrinkwrap your packages. Yarn provides this built-in with the use of a lock-file. With Yarn, every package and version is "locked" automatically meaning anyone else using your codebase will automatically be running the same as you. In other words, this gets ride of the "works on my computer" issues related to packaging.

2. Npm is serial by nature (again, at this point in time) so dependency downloads only happen one at a time. This not only takes longer but can end in a situation where you have multiple of the same package downloaded. Yarn is more intelligent with package download utilizing parallelism and smart dependency retrieval. This results in faster package management while also preventing unnecessary downloads.

With that quick compare/contrast, let's install Yarn.
1. Install
    + Download Yarn from the [Yarn website](https://yarnpkg.com/en/docs/install)
        + Note that like Node, you can either use the installer or download the binaries. Whichever version you end up going with make sure yarn is accessible from the command prompt by ensuring you add Yarn to your path much like you did with Node.    
2. Verify installation    
    + Verify with "yarn --version"
3. (optional) Disable SSL if you are behind a proxy (such as at work)
    + You can change the configuration by running the following command. To reverse this in the future, simply replace "false" with "true"
        ```
        yarn config set "strict-ssl" false -g
        ```
# Part 2: Git Configuration

Let's go ahead and configure git properly now that our main tools are installed. By default, git will use vim and vimdiff for most operations. Since we no longer live in the 70's, we're going to change this behavior so that git will use the tools we want. We'll start by telling git which editor we want for configuration changes and then go in and update the configuration.

1. Set VSCode as the configuration editor:

    + Execute the following command in GitBash. This will tell git to use VSCode as the default configuration editor.
        
        ```
        git config --global core.editor "code --wait"
        ```
        + Note that the --wait will tell GitBash to wait for an exit code from VSCode before continuing. This is useful if you want to make sure the editor finishes before git accepts a new command.

    + Open the global config in our editor of choice
        ```
        git config --global -e
        ``` 
        + You'll notice that our email and name are already in the configuration along with the default editor. When we ran the commands above, git modified this file to reflect those values.
    + Add configurations for default merge and diff tools
        ```
            [user]
                email = sean.merritt@us.sogeti.com
                name = Sean Merritt
            [core]
                editor = code --wait
            [merge]
                tool = vsdiffmerge
            [mergetool "vscode"]
                cmd = code --wait $MERGED
            [diff]
                tool = vsdiffmerge
            [difftool "default-difftool"]
                cmd = code --wait --diff $LOCAL $REMOTE
            [difftool]
                prompt = true
            [difftool "vsdiffmerge"]
                cmd = \"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\community\\Common7\\IDE\\CommonExtensions\\Microsoft\\TeamFoundation\\Team Explorer\\vsdiffmerge.exe\" \"$LOCAL\" \"$REMOTE\" //t
                keepBackup = false
            [mergetool]
                prompt = true
            [mergetool "vsdiffmerge"]
                cmd = \"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\community\\Common7\\IDE\\CommonExtensions\\Microsoft\\TeamFoundation\\Team Explorer\\vsdiffmerge.exe\" \"$REMOTE\" \"$LOCAL\" \"$BASE\" \"$MERGED\" //m
                keepBackup = false
                trustExitCode = true
        ```

        + Here we are setting both the diff and merge tools to use Visual Studio's diff/merge tools. You could set these to other tools if you prefer including things like tortoisesvn's tool (as an example). Really, set them to whichever tools you want. I prefer VS's.



# Part 4: Main Tools Review
Above we have downloaded, installed, and configured the main tools we're going to be using. We have our source control, ide, package manager, and the server platform we'll be building on top of. Now we're ready to start creating projects and configuring them in such a way that our lives are much easier. Let's get started!

# Part 5: Global Package installations
The first thing we'll want to grab is Angular CLI. This will be responsible for generating boiler-plate code as we're adding new applications, modules, components, and so on. 

1. Install Angular CLI
    + To install Angular CLI, run the following from GitBash.
        ```
        npm install -g @angular/cli
        ```
    + Note that this will install the Angular CLI globally. This may be fine but you can also install it locally by running the command above without "-g" while located in the local directory you want it for.

2. Verify install by running the following command from GitBash.
    ```
        ng
    ```

# Part 6: Application Creation
Now we're ready to start creating our own applications. The first step is to create an Angular application through the Angular CLI.

1. Create a new application named MyApp
    + Defaults with css
        ```
        ng new MyApp
        ```
    + Defaults with scss
        ```
        ng new MyApp --style=scss
        ```
    + If you plan to work on an existing project, you can set the styles through the CLI as well with:
        ```
        ng set defaults.styleExt scss
        ```
        + Note that you'll need to manually update any existing styles.
2. Note that the CLI went ahead and created/initialized a git repository for our application.
3. Update the .gitignore file to exclude unecessary items from making it into git.
    + You can download a pre-built git ignore file that includes most of what you need excluded for a project. Find the templates at [this git repository](https://github.com/github/gitignore).
        + I'd recommend the Angular template and possibly the Visual Studio template if you plan on using VS for more than merges/diffs.

# Part 7: Linting and Formatting
One common issue with development is conforming to a standard. There are many tools out there that help with this such as ReSharper but those are costly and we're working on open source tools here. Angular CLI applications come with built-in linting through TSLint. This handles some of what we need but it is mostly about code implementation than formatting (like standard accessability modifiers).

For example, how many code reviews have you had where someone says to remove a blank line or add an extra space somewhere? How long was the debate about that rule in the first place? Let's get rid of these problems.

## Prettier
Prettier is a package that provides formatting with easy customization. Let's get it going with our project.
1. Install Prettier
    - In GitBash, execute the following command to install Prettier.
        ```
        yarn add -D prettier
        ```
2. Create Yarn command scripts
    + Add the following script to packages.json
    ```
    "format": "prettier --write \"src/**/app/*.ts\""
    ```
    + Test script by entering the following command. You should see formatting status information.
        ```
        yarn format
        ```
    
3. Create run control script to handle Prettier configurations
    + Create run control .prettierrc. This will start VSCode and create the file in memory.
        ```
        code .prettierrc
        ```
    + Add formatting overrides to the run control file. Below is an example.
        ```
        {
            parser: "typescript",
            "overrides": [
                {
                    "files": "*.ts",
                    "options": {
                        semi: true,
                        singleQuote: true,
                        bracketSpacing: true,
                    }
                }
            ]
        }
        ```
    + Save the run control file
4. The Prettier and TSLint rules have some overlap since TSLint isn't strictly code enforcement and contains formatting rules. To make this easy on ourselves, we're going to use a new package that will help us identify any conflicting rules.
    + Install tslint-config-prettier with:
        ```
        yarn add -D tslint-config-prettier
        ```
    + Open tslint.json so we can link it's configuration with Prettier's.
        ```
        code tslint.json
        ```
    + Paste the following code in the file replacing any properties that share the same name.
        ```
        "extends": [
            "tslint-config-prettier"
        ],
        "rulesDirectory": [
            "node_modules/codelyzer"
        ],
        ```
    - Find conflicts
        ```
        "yarn tslint-config-prettier-check ./tslint.json"
        ```
    - Now comment or remove any conflicts in tslint.json that appeared in the output of the check above.

Now we can run ```yarn format``` whenever we want to format our entire codebase. Note that you can customize what files this command touches by altering the match expression specified when we created the format script. 

## Husky
At this point we are able to format code whenever we want with overrides if we need any custom rules. Pretty nifty, but we can take it a step further. As it stands, it would be easy for a developer to forget to format things before checking in. Let's fix that by making the code *always* format and lint before any code is staged.
1. Install Husky
    + Our formatting robot will run on Husky so let's install it
        ```
        yarn add -D lint-staged husky
        ```
    + Add the following scripts to packages.json at the end of the json object.
        ```
        code packages.json --wait
        ```
    + Paste the following into the end of the json object.
        ```
        "lint-staged": {
            "{src,e2e}/**/*.{js,jsx,ts,tsx}": [
                "yarn lint",
                "git add"
            ],
            "{src,e2e}/**/*.{js,jsx,ts,tsx,css,scss,md}": [
                "prettier --write",
                "git add"
            ]
        }
        ```        
    + Create a new Yarn command for the pre-stage action:
        ```
        "precommit": "lint-staged",
        ```
    + Save the file and exit Code

With everything in place above, every time we execute a ```git commit``` the formatting and linting rules will be applied to the files before they are commited. This means there will never be a time where someone is lashing out in a code review because of an extra space or line.

# Part 7: Unit Testing
Angular CLI comes with some built-in testing libraries but in order to have live unit testing that are many times faster, we'll be using a different package. Enter Jest:

## Jest
Jest is a very fast testing framework and spy that allows us to implement unit testing, have them running live, run them very fast, and provide additional benefits such as code coverage reporting. Let's start.
1. Install Jest
    + Run the following command to install Jest with the default Angular testing capabilities:
        ```
        yarn add -D jest jest-preset-angular
        ```
    + We now need to create the configuration file. Create the file.
        ```
        code jest.config.js --wait
        ```
    + Paste the following into the configuration
        ```
        module.exports = {
            preset: 'jest-preset-angular',
            setupTestFrameworkScriptFile: '<rootDir>/src/setupJest.ts'
        }
        ```
    + Save and close code
    + Create and populate the initialization file for Jest
        ```
        echo -e "import 'jest-preset-angular';\nimport 'jest-zone-patch';" > ./src/setupJest.ts
        ```
    - Add new yarn scripts
        + Open packages.json
            ```
            code packages.json --wait
            ```
        + Add the following scripts.
            ```
            "test": "jest --watch"
            "test:ci": "jest --runInBand"
            "test:coverage": "jest --coverage"
            ```
2. Test
    - Let's setup our first tests for the main application component.
        + Open ./src/app/app.component.spec.ts
            ```
            code ./src/app/app.component.spec.ts --wait
            ```
        + Overwrite the contents with the following:
            ```
            import { TestBed, async } from '@angular/core/testing';
            import { AppComponent } from './app.component';

            describe('AppComponent', () => {
            beforeEach(
                async(() => {
                TestBed.configureTestingModule({
                    declarations: [AppComponent]
                }).compileComponents();
                })
            );
            it(
                'should create the app',
                async(() => {
                const fixture = TestBed.createComponent(AppComponent);
                const app = fixture.debugElement.componentInstance;
                expect(app).toBeTruthy();
                })
            );
            it(
                `should have as title 'app'`,
                async(() => {
                const fixture = TestBed.createComponent(AppComponent);
                const app = fixture.debugElement.componentInstance;
                expect(app.title).toEqual('app');
                })
            );
            it(
                'should render title in a h1 tag',
                async(() => {
                const fixture = TestBed.createComponent(AppComponent);
                fixture.detectChanges();
                const compiled = fixture.debugElement.nativeElement;
                expect(compiled.querySelector('h1').textContent).toContain(
                    'Welcome to app!'
                );
                })
            );
            });

            ```
    - Try out the different test scripts you added above.
        ```
        yarn test
        yarn test:ci
        yarn test:coverage
        ```
# Part 8: End-To-End Testing
The (arguably) most importent tests are e2e tests as they make sure all components in your codebase are communicating with each other correctly. The Angular CLI came with Protractor and Selenium for this e2e testing. These tools are older, slower, and you need multiple packages to actually e2e testing. Instead we're going to use TestCafe which is an all-in-one e2e testing package.
1. Install TestCafe
    + Install TestCafe with the following command
        ```
        yarn add -D TestCafe testcafe-live
        ```
    + Now we can add our scripts to packages.json so we can easily run the e2e tests.
        + Open packages.json
            ```
            code packages.json --wait
            ```
        + Add the following scripts:
            ```
            "e2e": "ng serve & testcafe-live chrome e2e/**/*.e2e-spec.ts",
            "e2e:ci": "testcafe all e2e/**/*.e2e-spec.ts --app 'yarn start'
            ```
            + (optional) You can add a start delay if you find the tests are executing before the testing framework can initialize with the following modifier:
                ```
                --app-init-delay 5000"
                ```
2. Test  e2e testing
    + Open up the app.e2e-spec.ts file in the e2e directory.
        ```
        code ./e2e/app.e2e-spec.ts
        ```
    + Replace the contents with the following:
        ```
        import { AppPage } from './app.po';

        describe('angular-development App', () => {
        let page: AppPage;

        beforeEach(() => {
            page = new AppPage();
        });

        it('should display welcome message', () => {
            page.navigateTo();
            expect(page.getParagraphText()).toEqual('Welcome to app!');
        });
        });

        ```
    + Save and exit code
    + Add the protractor configuration for the test
        + Create e2e/app.po.ts
            ```
            code ./e2e/app.po.ts
            ```
        + Paste the following
            ```
            import { browser, by, element } from 'protractor';

            export class AppPage {
            navigateTo() {
                return browser.get('/');
            }

            getParagraphText() {
                return element(by.css('app-root h1')).getText();
            }
            }

            ```
    + Try out the different e2e testing scripts you created above. Note that depending on which command you're using, you may need to start your application first.
    
# Part 9: Review
The foundation for our development environment is now complete. We have automation tools to generate new code (Angular CLI), code structure and format standardization (Prettier and TSLint), automated source control hooks to ensure consistency (Husky), and of course testing tools to cover unit and end-to-end (integration) tests.

This was a very long document but keep in mind that every single thing in here can be scripted. This means you can create a single script (if you were so inclined) to set all of this up and every single new project, computer, or developer can simply execute the script and they will have the exact same tools, configurations, and packages. Say goodbye to "works on my machine" issues between team members!

# Part 10: Next Time....
Next time we'll cover some of the packages and libraries we can use that will help us architect, maintain, and create great code. These include (although not necessarily limited to): StoryBook and Rudux.
