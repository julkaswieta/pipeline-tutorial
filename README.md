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

```
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


## 2. Building and Testing

Now that you have an understanding of the syntax used in workflow files, you should be able to setup a build step using the standard dotnet build command.

It's important to remember that all commands will be executed from the root of your project, so you may need to supply a path to your .csproj file.


```
    - name: build
      run: dotnet build <path to .csproj file>
```

If you try to run a this, it may not work because the runner is missing some important resources:
*   The dependencies your program relies on
*   Your repository code


The first step of your pipeline will probably be checking out the code from your repo using the following action:
``` - uses: actions/checkout@v4 ```

Then you can restore the workloads and dependencies your project needs to build:

```
    - name: Restore workloads
      run: dotnet workload restore <Path to .csproj>

    - name: Restore dependencies
      run: dotnet restore <Path to .csproj>
```

## 3. External Tools


### Sonar


### Doxygen


### Database Migration