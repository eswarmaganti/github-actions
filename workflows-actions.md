# Workflows and Actions

## Workflows
high level overview of GitHub Actions workflows, including triggers, syntax, and advanced features

### About Workflows
A workflow is a configurable automation process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.

Workflows are defined in the `.github/workflows` directory in a repository. A repository can have multiple workflows, each of which can perform a different set of tasks such as:
- Building and testing pull requests
- Deploying your application every time a release is created
- Adding a label whenever a new issue is opened

### Workflow basics
A workflow must contain the following basic components
1. One or more events that will trigger the workflow.
2. One or more jobs, each of which will execute on a runner machine and run a series of one or more steps.
3. Each step can either run a script that you define or run an action, which is reusable extension that can simplify your workflow.

### Workflow triggers
Workflow triggers are events that cause a workflow to run. These events can be:
- Events that occur in your machine repository
- Events that occur outside of GitHub and trigger a `repository_dispatch` event on GitHub
- Scheduled times
- Manual

For example, you can configure your workflow to tun when a push is made to the default branch of your repository, when a release is created, or when an issue is opened.

Workflow triggers are defined with the `on` key. 
The following steps occur to trigger a workflow run:
1. An event occurs on your repository. The event has an associated commit SHA and Git ref.
2. GitHub searches the `.github/workflows` directory in the root of your repository for workflow files that are present in the associated commit SHA or Git ref of the event.
3. A workflow run is triggered for any workflows that have `on:` values that match the triggering event. Some events also require the workflow file to be present on the default branch of the repository in order to run.

Each workflow run will use the version of the workflow that is present in the associated commit SHA or Git ref of the event. When a workflow runs, GitHub sets the `GITHUB_SHA` (commit SHA) and `GITHUB_REF` (Git ref) environment variables in the runner environment.

---

## Variables
Learn about variables in GitHub Actions workflows

### About
Variables provide a way to store and reuse non-sensitive configuration information. You can store any configuration data such as compiler flags, usernames, or server names as variables. Variables are interpolated on the runner machine that runs your workflow. Commands that run in actions or workflow steps can create, read, and modify variables.

You can set your own custom variables or use the default environment variables that GitHub sets automatically.

You can set the custom variables in two ways
- To define an environment variable for use in a single workflow, you can use the `env` key in the workflow file. 
- To define a configuration variable across multiple workflows, you can define it at the Organization, repository, or environment level.
  - When creating a variable in an organization, you can use a policy to limit access by repository. For example, you can grant access to all repositories, or limit access to only private repositories or a specified list of repositories.

> By Default, variables render unmasked in your build outputs. If you need greater security for sensitive information, such as passwords, use secrets instead.

---

## Contexts
Learn about contexts in GitHub Actions

### About contexts
Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps. Each context is an object that contains properties, which can be strings or other objects.

Contexts, objects, and properties will vary significantly under different workflow run conditions. For example, the `matrix` context is only populated for jobs in a matrix.

You can access the context using the expression syntax `${{ <context> }}`

### Determining when to use contexts
GitHub Actions includes a collection of variables called *contexts* and a similar collection of variables called *default* variables. These variables are intended for use at different points in the workflow:

- **Default environment variables**: The environment variables exist only on the runner that is executing your job. 
- **Contexts**: You can use most contexts at any point in your workflow, including when *default variables* would be unavailable.
  - For example, you can use contexts with expressions to perform initial processing before the job is routed to a runner for execution; this allows you to use a context with the conditional `if` keyword to determine whether a step should run. Once the job is running, you can also retrieve context variables from the runner that is executing the job, such as `runner.os`

```yaml
name: CI
on: push
jobs:
  prod-check:
    if: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - name: deployment
        run: echo "Deploying the application to production server on branch $GITHUB_REF"
```

--- 

## Expressions
You can evaluate expressions in workflows and actions.

### About expressions
You can use expressions to programmatically set environment variables in workflow files and access contexts. An expression can be any combination of literal values, references to a context, or functions. You can combine literals, context references, and functions using operators.

Expressions are commonly used with the conditional `if` keyword in a workflow file to determine whether a step should run. When an `if` conditional is `true`, the step will run.

You need to use specific syntax to tell GitHUb to evaluate an expression rather than treat it as a string. `${{ <expression> }}`

---

## Reusing workflow configurations
Learn how to avoid duplication when creation a workflow.

### Reusable workflows
Rather than copying and pasting from one workflow to another, you can make workflows reusable. You and anyone access to the reusable workflow can then call the reusable workflow from another workflow.

Reusing workflows avoids duplication. This makes workflows easier to maintain and allows you to create new workflows more quickly by building on the work of others, just as you do with actions. Workflow reuse also promotes the best practice by helping you to use workflows that are well-designed, have already been tested, and have been proven to be effective.

A workflow that uses another workflow is referred to as a "caller" workflow. The reusable workflow is a "called" workflow. One caller workflow can use multiple called workflows. Each called workflow is referred in a single line.
The result is that the caller workflow file may contain just a few lines of YAML, but may perform a large number of tasks when it's run. When you reuse a workflow, the entire called workflow is used, just as it was part of the caller workflow.

If you reuse a workflow from a different repository, any actions in the called workflow run as if they were part of the caller workflow. For example, **if the called workflow uses `actions/checkout`, the action checks out the contents of the repository that hosts the caller workflow, not the called workflow.**

### Reusable workflows vs composite actions
Reusable workflows and composite actions both help you avoid duplicating workflow content. Whereas reusable workflows allow you to reuse entire workflow, which multiple jobs and steps, composite actions combine multiple steps that you can then run within a single step, just like any other action.

Lets compare some aspects of each solution:
- **Workflow Jobs**: Composite actions contains series of steps that are run as s single step within the caller workflow. Unlike reusable workflows, they cannot contain jobs.
- **Logging**: When a composite action runs, the log will show just the step in the caller workflow that ran the composite action, not the individual steps within the composite action. With reusable workflows, every job and step is logged separately.
- **Specifying Runners**: Reusable workflows contain one or more jobs, As with all workflow jobs, the jobs in a reusable workflow specify the type of machine on which the job will run. Therefore, if the steps be run on a type of machine that might be different from the machine chosen for the calling workflow job, then you should use a reusable workflow, not a composite action. 
- **Passing output to steps**: A composite action is run as a step within the workflow job, and you can have multiple steps before or after that step that runs the composite action. *Reusable workflows are called directly within a job, and not from within a job step.* You can't add steps to a job after calling a reusable workflow, so tou can't use `GITHUB_ENV` to pass values to subsequent job steps in the caller workflow.

### Workflow templates
Workflow templates allow everyone in your organization who has permission to create workflows to do so more quickly and easily. When people create a new workflow, they can choose a workflow template and some or all of the work of writing the workflow will be done for them. Within a workflow template, you can also reference reusable workflows to make it easy for people to benefit from reusing centrally managed workflow code.

If you use a commit SHA when referencing the reusable workflow, you can ensure that everyone who reuses that workflow will always be using the same YAML code. However, if you reference a reusable workflow by a tag or branch, be sure that you can trust that version of the workflow.

GitHub offers workflow templates for a variety of languages and tooling. When you set up workflows in your repository, GitHub analyzes the code in your repository and recommends workflows based on the language and framework in your repository. For example, ig you use Node.js GitHub will suggest a workflow templatefile that installs your Node.js packages and runs your tests. 

### YAML anchors and aliases
You can use YAML anchors and aliases to reduce repetition in your workflows. An anchor (marked with `&`) identifies a piece of content that you want to reuse, while an alias (marked with `*`) repeats that content in another location. Think of an anchor as creating a named template and an alias using that template. This is particularly useful when you have jobs or steps that share common configuration.

---

## Custom Actions
Actions are individual tasks that you can combine to create jobs and customize your workflow. You can create your own actions, or use and customize actions shared by the GitHub community.

### About custom actions
You can create actions by writing custom code that interacts with your repository in a way you'd like, including integrating with GitHub APIs and any publicly available third-party API. For example, an action can publish npm module, send SMS alerts when urgent issues are created, ot deploy production-ready code.

You can write your own actions to use in your workflow or share the actions you build with the GitHub community. To share actions you've built with everyone, your repository must be public.

Actions can run directly on a machine or in a Docker container. You can define an action's inputs, outputs, and environment variables.

### Type of actions
> You can build Docker container, JavaScript, and composite actions. Actions require a metadata file to define the inputs, outputs and runs configuration for your action. Action metadata files uses YAML syntax, and the metadata filename must be either `action.yaml` or `action.yml`

#### Docker Container actions
Docker containers package the environment with the GitHub Actions code. This creates a more consistent and reliable unit of work because the consumer of the action does not need to worry about the tools or dependencies.

A Docker container allows you to use specific versions of an operating system, dependencies, tools, and code. FOr actions that must run in a specific environment configuration, Docker is an ideal option because you can customize the operating system and tools. Because of the latency to build and retrieve container, Docker container actions are slower than JavaScript actions.

Docker container actions can only execute on runners with a Linux operating system. Self-hosted runners must use a Linux operating system and have Docker installed to tun Docker container actions.

#### JavaScript actions
JavaScript actions can run directly on a runner machine, and separate the action code from the environment used to run the code. Using a JavaScript action simplifies the action code and executes faster than a Docker container action. 

To ensure your JavaScript actions are compatible with all GitHub-hosted runners (Ubuntu, Windows and macOS), the packaged JavaScript code you write should be pure JavaScript and not rely on other binaries. JavaScript actions run directly on the runner and use binaries that already exist in the runner image.

#### Composite actions
A composite action allows you to combine multiple workflow steps within one action. For example, you can use this feature to bundle together multiple run commands into an action, and then have a workflow that executes the bundled commands as a single step using that action.

---

## Concurrency
By default, GitHub Actions allows multiple jobs with the same workflow, multiple workflow runs within the same repository, and multiple workflow runs across a repository owner's account to run concurrently. This means that multiple instances of the same workflow or job can run at the same time, performing the same steps.

GitHub Actions also allows you to disable concurrent execution. This can be useful for controlling your account's organization resources in situation where running multiple workflows or jobs at the same tome could cause conflicts or consume more Actions minutes and storage than expected. For example, you might want to prevent multiple deployments from running at the same time or cancel linters checking outdated commits.

When you limit concurrency, by default only one run can be pending in a concurrency group -- any additional pending runs cancel the previous one. If you need runs to execute sequentially without being canceled, you can opt in to queue, which allows multiple runs to wait in line and execute in order.

To start controlling concurrency in your own workflows with the `concurrency` keyword.

---

## Workflow Artifacts
Learn about storing and sharing data as artifacts of GitHub Actions workflows.

### About workflow artifacts
An artifact is a file or collection of files produced during a workflow run. Artifact allow you to persist data after a job has completed, and share that data with another job in the same workflow. For example, you can use artifacts to save your build and test output after a workflow run has ended.

GitHub provides two actions that you can use to upload and download build artifacts, `upload-artifact` and `download-artifact`.

Common artifacts include:
- Log files and core dumps
- Tet results, failures, and screenshots
- Binary or compressed files
- Stress test performance output and code coverage results.

### Artifacts vs dependency caching
Artifacts and caching are similar because they provide the ability to store files on GitHUb, but each feature offers different use cases and cannot be used interchangeably.
- Use caching when you want to reuse files that don't change often between jobs or workflow runs, such as build dependencies from a package management system.
- Use artifacts when you want to save files produced by a job to view after a workflow run has ended, such as built binaries or build logs.

### Generating artifact attestation for builds
Artifact attestations enable you to create unfalsifiable provenance and integrity guarantees for the software you build. In turn, people who consume your software can verify where and how your software was built.

When you generate artifact attestations with your software, you create cryptographically signed claims that establish your build's provenance and include the following information.
- A link to the workflow associated with the artifact
- The repository, organization, environment, commit SHA, and triggering event for the artifact
- Other information from the OIDC token used to establish provenance.

You can also generate artifact attestations that include an associated software bill of materials (SBOM). Associating your builds with a list of the opensource dependencies used in them provides transparency and enables consumers to comply with data protection standards.

---

## Dependency caching
Learn about dependency caching for workflow speed and efficiency.

### About workflow dependency caching
Workflow runs often reuse the same outputs or download dependencies from one run to another. For example, package dependency management tools such as Maven, Gradle, npm and Yarm keep a local cache of downloaded dependencies.

Jobs on GitHUb-hosted runners start in a clean runer image and must download dependencies each time, causing increased network utilization, longer runtime, and increased cost. To help speed up the time it takes to recreate files like dependencies, GitHub can cache files you frequently use in workflows.
