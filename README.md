# Implementing a continuous integration pipeline with GitHub Actions 

This tutorial will demonstrate how you can create a continuous integration pipeline using Github Actions and set up some external tools that will be used in the pipeline. 

In this tutorial, you will learn how to:
*   Create a Github Actions workflow file.
*   Run your pipeline in the cloud.
*   Setup automated builds and testing.
*   Add useful external tools:
    *   Sonar static analysis
    *   Doxygen
    *   Database migration using entity framework

## Before you start
Before you start working through this tutorial, it might be useful to take a look at the notes about Continuous Integration/Deployment, and DevOps in general. This will explain the concepts we use in this tutorial. You can find these here: [DevOps notes](https://edinburgh-napier.github.io/remote_test/notes/unit7_devops/).

To work through the steps outlined in this document, you need to have a dotnet project. This pipeline should work with any dotnet project, but for the purpose of this module, make sure you have completed the first tutorial on setting up a MAUI project, which you can find here: [Getting started with MAUI](https://edinburgh-napier.github.io/remote_test/tutorials/csharp/maui/maui.html).

## 1. GitHub Actions
GitHub Actions is a powerful CI/CD solution integrated into GitHub repositories. It allows developers to create workflows that run when certain events occur within a repository. A workflow is GitHub's name for a pipeline. For example, you can have a workflow that runs anytime new code is pushed to the master branch and checks whether the code builds and runs correctly. 

GitHub Actions can be used for free by anyone as long as the project is open-source. That means that the repository that hosts your code must be public. Otherwise, every workflow run incurs additional costs.

You can check that your repository for this project is public by going to its main page in GitHub. If it is, a "Public" badge will be displayed next to the name of the repository.

![Checking whether a repository is public](images/public-repo.png)

### Workflow File
Creating a standard workflow in GitHub Actions is easy. GitHub provides a wide array of templates for creating a new workflow. It can even suggest the most suitable template based on the languages you use in your project.  In every GitHub repository, there is an **Actions** tab where you can setup a new workflow or monitor your current ones. 

![Actions tab in GitHub](images/actions-tab.png)

In this tutorial, you will write your own workflow file from scratch to get an in-depth understanding of what each line does. If you decide you want to know a bit more about Actions, GitHub has a very comprehensive documentation, which you can find here: [GitHub Actions documentation](https://docs.github.com/en/actions). 

Every workflow file is in the YAML format, which is commonly used for configuration and uses minimal syntax. YAML files can have either the .yaml or .yml extension. Both are accepted by GitHub Actions - the choice is down to personal preference. 

### Syntax
Even though most CI/CD runners use YAML files for their steps definitions, each of them requires a different syntax. GitHub Actions is event-oriented, which means that it first defines the event that will trigger the following jobs. The table below outlines different keywords used in GitHub's workflow files that will be used in this tutorial. A full list is available in the documentation: [GitHub Actions Syntax Docs](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions).

| Keyword   | Description   |
| ---       | ---           |
| **name**  | Sets the name of the workflow, which is used by GitHub Actions to identify specific workflows. |
| **on**        | Declares what events will cause the pipeline defined in the file to run. There can be one or more of these. Examples of events include push or pull request. |
| **jobs**      | A collection of high-level stages in the pipeline. Each job has a set of steps to execute. For example, we can have a build job and a deploy job.  |
| **runs-on**   | Specifies which operating system will run your job.    |
| **steps**     | List of steps contained in a job. Each step starts with `- name: <name of step>`. Each step has its own commands to run or configurations to set up. |
| **uses**      | Specifies an existing action to be used within the step. They can come from GitHub's marketplace or a repository.    |
| **run**       | Runs a command or script. Tip: You can run a sequence of commands using pipes.    |
| **with** | Typically used in conjunction with **uses**, it provides input parameters for an action to configure how it runs. |
| **if** | Adds a conditional statement to a job or a step. |
| **env** | Defines environment variables for a job or a step. This can be used to pass parameters to a command, script or an action. | 
| **permissions** | Specifies what resources the job can access within the repository. |
| **needs** | Specifies job dependencies. Basically means that the job after the keyword needs to run first in order for this job to run successfully. Essentially defines the order of execution.  |

> Note: When you specify multiple events after the `on` keyword, all jobs contained in the same workflow file will run whenever any of those events occur. If you need different workflows to run on occurrence of different events, you will need to include multiple workflow files.

Here is an template for a very basic workflow file:

``` yml
name: <Name of your pipeline> 

# What events cause the pipeline to run
on:
  push:
    branches:
      - master # Will only run when a change is made to master (including a merge)

jobs:
  <Name of job>:

    # Which OS the runner wil use
    runs-on: windows-latest

    # A list of steps to run
    steps:
      - name: <Name of step>
        run: echo "Some command" 
```

### Secrets and environment variables
You might sometimes need to use values that are sensitive to configure your CI/CD pipelines successfully. Since they are sensitive, it is a very bad idea to commit them to a repository, especially if it's public. Locally, you can manually create configuration files and fill the sensitive information yourself or if you are working in a company, this might be sent to you over secure channels. 

CI/CD pipelines run automatically in the cloud so they don't have access to those local files. GitHub provides us with **secrets**. They are encrypted values that are used to securely store sensitive information and are not visible in the repository, but Actions can still access them. 

You should use secrets for anything that could be exploited to gain unauthorised access, e.g. connection strings, API keys, passwords, access tokens etc. We'll flag it up when you need to do this for this project.

## 2. Basic workflow setup
Now that you have an understanding of the syntax used in workflow files, you can move on to implementing your own workflow. 

> It might be a good idea to create a new branch before you start making any changes, e.g. `feature/workflow`. 

### Setting up your workflow file
To set up your first GitHub Actions workflow manually, create a `.github` directory in the root folder of your project (note the leading dot) and then a `workflows` directory inside it. Then, create a file named `workflow.yml` which will store all the instructions for your pipeline. Any `.yml` or `.yaml` file in this folder will be interpreted as a workflow in Github Actions. Your file structure should look like this:

![File structure for workflow](images/file-structure.png)

The first step in creating pipelines is deciding what events will trigger the runs. It could run anytime something is pushed to any of the branches, or only to some selected branches. Another option is to run the workflow on pull requests, which we will use as an example in this tutorial. A full breakdown can be found in the documentation: [Triggering a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow). 

Start your file by putting the following code in the `workflow.yml` file:
```yml
name: Build & Test # the name can be anything else you want

on: 
  [pull_request]
```

The next step will be deciding what actions get executed in the jobs. 

### Setting up the environment
You can now move on to defining what jobs and actions will be executed in the workflow. You also need to decide what operating system will be used for the runner. For this project, we will use the lastest version of Windows. Add this code to your workflow file:

```yml
jobs:
  build: # this is just a name of the job, it can be changed to something else if you wish
    
    runs-on: windows-latest
```

Now you can start defining the order of the steps in the job. Usually, the starting point is ensuring that the workflow can access the source code. In GitHub Actions, you can use a standard action that checks out the code from the repository. You can do this by using this code:

```yml
jobs:
  build:

    runs-on: windows-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4
```

The next step is ensuring that the environment is set up properly and all tools needed to build the code are installed. In .NET projects, this involves setting up the .NET SDK, restoring workloads and dependencies. 

To set up the SDK, you can use a ready-made action and provide it with the version of .NET that you require:

``` yml
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 8.0
```

Workloads in .NET projects are various additional tools, libraries or features that are not included by default in the SDK, for example MAUI. They are defined in the `.csproj` file so the command that does it needs a correct path to that file. 

> It's important to remember that all commands in the workflow will be executed from the root of your project, so you must supply a relative path to your .csproj file, e.g. `./Project/Project.csproj`. Make sure the paths are relative to the root of the project, not the workflows folder. 

```yml
    - name: Restore workloads
      run: dotnet workload restore <Path to .csproj>
```

Apart from restoring workloads, you must also restore dependencies to ensure the project works properly. This can be done by adding this step:

```yml
    - name: Restore dependencies
      run: dotnet restore <Path to .csproj>
```

### Building the project
Now that you've got the source code checked out and the environment set up, you can move on to building the project. This is done by running `dotnet build` like so:

``` yml
    - name: Build project
      run: dotnet build <path to .csproj file>
```
> Note: When building a MAUI app, you might need to specify what framework to use for the build. You can do this by appending the `--framework` argument to the command above like so: `--framework net8.0`

### Optional: Adding environment variables to GitHub
You might notice that you are reusing certain values throughout your pipeline, such as the path to your project file. To encourage reuse, you can set up a variable in GitHub to make future modification more efficient.

To add a variable, navigate to the Settings tab in the repository 

![Settings tab in GitHub](images/settings-tab.png)

In the menu on the left, navigate to `Secrets and variables` > `Actions`

![Actions settings](images/actions-menu.png)

Switch to the `Variables` tab and select `New repository variable`.

![Variables tab](images/variables.png)

This will take you to a page where you give a name to your variable, e.g. `CSPROJ_PATH`, and the value, which is the path to your `.csproj` file in this example.

![Adding a variable](images/add-variable.png)

After you add the variable in GitHub, you can use it in your workflow file like this:

```yml
- name: Build
  run: dotnet build ${{ vars.CSPROJ_PATH }}
```

### Testing the code
Testing is a critical element of any CI/CD pipeline. Automatically running a suite of tests speeds up development significantly, while ensuring that the application remains fully functional and the recent changes have not introduced any bugs (or at least the bugs that are covered by tests).

In GitHub Actions, you can run tests as part of the workflow. Usually, you'll want to have a separate step for this after the build step. To test a .NET project, you can use this command, which will run all tests included in the entire solution. 

> Note: in all previous steps, you needed a path to the `.csproj` file, but in this case it's the `.sln` file because tests will usually be defined in a separate project that makes use of the base project. The solution file brings them both together so they can communicate. 

``` yml
    - name: test
      run: dotnet test <path to .sln>
```

At this point, you can commit your changes and push. Then, when you open a pull request to merge your changes to another branch, you will trigger your workflow run. Go ahead and do this now to check your workflow runs properly. You can do this from GitHub. 

## 3. Extending your pipeline with external tools
You now know how to create a basic pipeline that builds and tests your code. You can expand the pipeline by adding external tools for static code analysis, database migrations, documentation generation or many other things that will add value to the automatic runs.

### Static code analysis wit SonarQube Cloud
Static code analysis can tell us a lot about the quality of the code written. SonarQube Cloud is a cloud-based version of the SonarQube code analyser. You can find information about the local usage of SonarQube in this tutorial: [SonarQube](https://edinburgh-napier.github.io/remote_test/tutorials/tools/sonarqube/).

#### Setting up a SonarCloud account, organization and project
To use the cloud version of Sonar, you have to create an account on their website. You can find it here: [SonarQube Cloud signup](https://www.sonarsource.com/products/sonarcloud/signup/). Since we are using GitHub to store the repository, you should sign up using GitHub. If you do that, you will be asked to authorize SonarCloud to gain access to certain details of your GitHub account, which you should accept by pressing the `Authorize SonarCloud` button. 

![Fig. 1. Sonar Cloud Signup Page](images/sonar-signup.png)

Once your account is ready, you will be taken to the getting started screen, where you can import your organization from GitHub that owns your repository. You should have that set up if you worked through the first tutorial. If not, you can create an organization manually in Sonar by clicking on the underlined `create a  project manually`. 

> Setting up a project manually in SonarQube is not recommended as it leads to missing features like analysis feedback in the Pull Request. We recommend you go back and setup an organization in GitHub to host your code. If you still decide against it, you can follow the on-screen instructions to create a manual organization. 

We're going to assume you have an organization in GitHub so select `Import an organization`.

![Import organization in Sonar](images/import-org-sonar.png)

On the next page, select which orgaization you want to use. Then, you will be asked for permissions to ask Sonar on your organization. Since we only need the analysis on one repository, choose the `Only select repositories` option and select the correct repository, then proceed by pressing `Install`.

![alt text](images/select-repo-sonar.png)

On the next page, you will be asked for the name and the key of the project - keep them as is. You also need to choose the payment plan so make sure to select the free one to not incur any costs. Proceed by pressing `Create organization`.

Now that your Sonar organization is set up, you have to select the repositories that will be included in the Sonar project you are creating. 

![Choosing the project](images/choose-project-sonar.png)

Finally, you need to choose what Sonar considers new code in the repository. You have two choices which are explained in the screenshot below. You should proceed with the `Previous version` one. Then, you can finally create the project.

![Selecting what is new code](images/new-code.png)

#### Adding SonarCloud to your workflow



Add all this stuff to your workflow file.

``` yml
    #Setup a Java JDK
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: 17
        distribution: 'zulu'

    #Cache Sonar Dependencies
    - name: Cache SonarCloud packages
      uses: actions/cache@v4
      with:
        path: ~/sonar/cache
        key: ${{ runner.os }}-sonar
        restore-keys: ${{ runner.os }}-sonar

    - name: Cache SonarCloud scanner
      id: cache-sonar-scanner
      uses: actions/cache@v4
      with:
        path: ./.sonar/scanner
        key: ${{ runner.os }}-sonar-scanner
        restore-keys: ${{ runner.os }}-sonar-scanner

    #Install the SonarCloud Scanner
    - name: Install SonarCloud scanner
      if: steps.cache-sonar-scanner.outputs.cache-hit != 'true'
      run: |
        mkdir -p ./.sonar/scanner
        cd ./Pipeline
        dotnet tool update dotnet-sonarscanner --tool-path ../.sonar/scanner

    - name: Start Sonar Analysis
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: |
        ./.sonar/scanner/dotnet-sonarscanner begin /k:<key>" /o:"<organisation>" /d:sonar.token="${{ secrets.SONAR_TOKEN }}" /d:sonar.host.url="https://sonarcloud.io" /d:sonar.scanner.scanAll=false /d:sonar.cs.vscoveragexml.reportsPaths=coverage.xml

    #Build and test steps here
    #
    #
    #

    - name: End Sonar Analysis
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: ./.sonar/scanner/dotnet-sonarscanner end /d:sonar.token="${{ secrets.SONAR_TOKEN }}"

```

### Doxygen

Doxygen is a tool that generates a web-based representation of your project's documentation. It takes the comments from your code and produces HTML files with all of the details.


![Fig. 2. Example Doxygen Output](images/Doxygen_Example.png)

It might be best to create a new workflow file that runs only on pushes to master, so that your documentation site is not updated with anything outside of your main branch.

``` yml
name: Documentation 

on:
  push:
    branches:
      - master

jobs:
  generate:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Install Doxygen
      run: sudo apt-get install doxygen -y

    - name: Generate Doxygen Documentation
      run: doxygen <path to Doxygen file>
```

This job generates the documentation but it would be better if we could access it from the web. To do this, you can use Github pages by uploading the generated artifacts and then deploying to Github pages.

First you must add a step to the 'generate' job which uploads the artifacts to Github pages:

> You will need write permissions for the job to access Github pages.


``` yml
    permissions:
      pages: write
      id-token: write

    - name: Upload static files as artifact
      uses: actions/upload-pages-artifact@v3 
      with:
        path: html
```


You should add a new job called 'deploy' after the generation job with the following contents.

``` yml
  deploy:
    needs: generate
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```


### Database Migration

After updating the datamodels in your application, you can use entity framework to migrate those changes into your database.


``` yml
- name: Migrate Database
  run: |
      dotnet ef migrations add ${{ github.run_number }}
      dotnet ef database update
```

