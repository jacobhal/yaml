# YAML

## YAML Basics

This document covers the schema of an Azure Pipelines YAML file. To learn the basics of YAML, see [Learn YAML in Y Minutes](https://learnxinyminutes.com/docs/yaml/). Azure Pipelines doesn't support all YAML features. Unsupported features include anchors, complex keys, and sets. Also, unlike standard YAML, Azure Pipelines depends on seeing stage, job, task, or a task shortcut like script as the first key in a mapping.

## Azure DevOps Pipelines

To check out how to use YAML with Azure DevOps you can also look at Microsoft's docs: https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema

YAML is often used to define a full build pipeline. This section will cover how to use YAML with Azure DevOps Pipelines.

A pipeline is one or more stages that describe a CI/CD process. Stages are the major divisions in a pipeline. The stages "Build this app," "Run these tests," and "Deploy to preproduction" are good examples.

A stage is one or more jobs, which are units of work assignable to the same machine. You can arrange both stages and jobs into dependency graphs. Examples include "Run this stage before that one" and "This job depends on the output of that job."

A job is a linear series of steps. Steps can be tasks, scripts, or references to external templates.

This hierarchy is reflected in the structure of a YAML file like:

* Pipeline
  * Stage A
    * Job 1
      * Step 1.1
      * Step 1.2
      * ...
    * Job 2
      * Step 2.1
      * Step 2.2
      * ...
  * Stage B
    * ...

Simple pipelines don't require all of these levels. For example, in a single-job build you can omit the containers for stages and jobs because there are only steps. And because many options shown in this article aren't required and have good defaults, your YAML definitions are unlikely to include all of them.

### User-defined variables

When you define a variable, you can use different syntaxes (macro, template expression, or runtime) and what syntax you use will determine where in the pipeline your variable will render.

In YAML pipelines, you can set variables at the root, stage, and job level. You can also specify variables outside of a YAML pipeline in the UI. When you set a variable in the UI, that variable can be encrypted and set as secret. Secret variables are not automatically decrypted in YAML pipelines and need to be passed to your YAML file with env: or a variable at the root level.

User-defined variables can be set as read-only.

You can use a variable group to make variables available across multiple pipelines.

You can use templates to define variables that are used in multiple pipelines in one file.

### System variables

In addition to user-defined variables, Azure Pipelines has system variables with predefined values. If you are using YAML or classic build pipelines, see [predefined variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml) for a comprehensive list of system variables. If you are using classic release pipelines, see [release variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/variables?view=azure-devops&tabs=batch).

System variables are set with their current value when you run the pipeline. Some variables are set automatically. As a pipeline author or end user, you change the value of a system variable before the pipeline is run.

System variables are read-only.

### Environment variables

Environment variables are specific to the operating system you are using. They are injected into a pipeline in platform-specific ways. The format corresponds to how environment variables get formatted for your specific scripting platform.

On UNIX systems (macOS and Linux), environment variables have the format $NAME. On Windows, the format is %NAME% for batch and $env:NAME in PowerShell.

System and user-defined variables also get injected as environment variables for your platform. When variables are turned into environment variables, variable names become uppercase, and periods turn into underscores. For example, the variable name any.variable becomes the variable name $ANY_VARIABLE.

### Variable Example

Azure Pipelines supports three different ways to reference variables: macro, template expression, and runtime expression. Each syntax can be used for a different purpose and has some limitations.

In a pipeline, template expression variables (${{ variables.var }}) get processed at compile time, before runtime starts. Macro syntax variables ($(var)) get processed during runtime before a task runs. Runtime expressions ($[variables.var]) also get processed during runtime but were designed for use with conditions and expressions. When you use a runtime expression, it must take up the entire right side of a definition.

In this example, you can see that the template expression still has the initial value of the variable after the variable is updated. The value of the macro syntax variable updates. The template expression value does not change because all template expression variables get processed at compile time before tasks run. In contrast, macro syntax variables are evaluated before each task runs.

```YAML
variables:
- name: one
  value: initialValue 

steps:
  - script: |
      echo ${{ variables.one }} # outputs initialValue
      echo $(one)
    displayName: First variable pass
  - bash: echo '##vso[task.setvariable variable=one]secondValue'
    displayName: Set new variable value
  - script: |
      echo ${{ variables.one }} # outputs initialValue
      echo $(one) # outputs secondValue
    displayName: Second variable pass
```

### Which variable syntax to use?
https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch

### Pipelines

### Stage
A stage is a collection of related jobs. By default, stages run sequentially. Each stage starts only after the preceding stage is complete unless otherwise specified via the dependsOn property.

### Job
A job is a collection of steps run by an agent or on a server. Jobs can run conditionally and might depend on earlier jobs.

### Container reference
A container is supported by jobs.

### Deployment job
A deployment job is a special type of job. It's a collection of steps to run sequentially against the environment. In YAML pipelines, we recommend that you put your deployment steps in a deployment job.
