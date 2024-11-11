# Github Actions Tutorial

This tutorial will demonstrate how to create a pipeline using Github Actions and set up some external tools that will make the pipeline useful for continuous intigration.

In this tutorial, you learn how to:

*   Create a Github Actions workflow file.
*   Run your pipeline in the cloud.
*   Setup automated builds and testing.
*   Add useful external tools:
    *   Sonar static analysis
    *   Doxygen
    *   Database migration using entity framework


You will need a dotnet project to follow this tutorial.

## 1. Github Actions

Setup (or assume) a github rep?
Public repo, so you don't incur costs

### Workflow File

In the root of your project (where your .git folder is) create a directory '.github/workflows'. Any .yml file in this folder will be interpreted as a pipeline in Github Actions. You can now create a file named 'workflow.yml', which will store all the instructions for your pipeline.

### Syntax

#### On

This is how you declare what events will cause the pipeline to run.

#### Jobs

A collection of actions that represent one item in you pipeline.

#### Runs On

Declaration of which operating system will run your job.

#### Steps

List of actions contained in a job.

#### Uses

Call an existing action.

#### Run

Run a command.

Tip: You can run a sequence of commands using pipes.

### A Basic Workflow

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
All syntax is available in the [Github Actions Documentation](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions)


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

It might be useful to automatically test your code in pull requests, so that only valid changes will be merged into your project. To accomplish this you can add another step to run dotnet test.

``` yml
    - name: test
      run: dotnet test <path to solution>
```


## 3. External Tools


### Sonar


### Doxygen


### Database Migration