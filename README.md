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

To set up your first GitHub Actions workflow manually, create a `.github` directory in the root folder of your project (note the leading dot) and then a `workflows` directory inside it. Then, create a file named `workflow.yml` which will store all the instructions for your pipeline. Any `.yml` or `.yaml` file in this folder will be interpreted as a workflow in Github Actions. Your file structure should look like this:

![File structure for workflow](images/file-structure.png)

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

You should use secrets for anything that could be exploited to gain unauthorised access, e.g. connection strings, API keys, passwords, access tokens etc.

## 2. Building and Testing .Net

Now that you have an understanding of the syntax used in workflow files, you should be able to setup a build step using the standard dotnet build command.

It's important to remember that all commands will be executed from the root of your project, so you may need to supply a path to your .csproj file.


``` yml
    - name: build
      run: dotnet build <path to .csproj file>
```

If you try to run this, it may not work because the runner is missing some important resources:
*   The Dotnet framework
*   Your repository code
*   The dependencies your program relies on


The first step of your pipeline will probably be checking out the code from your repo using the following action:
``` yml
    - uses: actions/checkout@v4 
```
This is an example of ... stored action thingy.


Now you can setup dotnet.

``` yml
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 8.0
```

Then you can restore the workloads your project needs to build (like MAUI):

``` yml
    - name: Restore workloads
      run: dotnet workload restore <Path to .csproj>
```

Side note:
    You might notice that you are reusing certain values throughout your pipeline, such as the path to your project file. To reduce reuse, you can set up a variable in github to make future modification more efficient.

    ...

Here is an example workflow file that automatically builds a MAUI app.

``` yml
name: Build MAUI App 

on:
  [pull_request]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 8.0

    - name: Restore workloads
      run: dotnet workload restore <path to .csproj>

    - name: build
      run: dotnet build <path to .csproj>
```

> When building a MAUI app, you might need to supply a --framework condition like: --framework net8.0

It might be useful to automatically test your code in pull requests, so that only valid changes will be merged into your project. To accomplish this you can add another step to run dotnet test.

``` yml
    - name: test
      run: dotnet test <path to solution>
```


## 3. External Tools


### Sonar
Make an account in [sonarcloud](https://www.sonarsource.com/products/sonarcloud/signup/)

![Fig. 1. Sonar Cloud Signup Page](images/sonar-signup.png)

Make a new organisation, you can import one form Github or make one manually in sonar cloud.

Setup a project in sonarcloud, making note of the token.

Make a variable in Github that stores the token provided by sonar cloud.



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

