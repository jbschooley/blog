---
title: "Moving On from AWS Amplify"
author: Jacob Schooley
#excerpt:
tags:
  - AWS
  - infrastructure as code
#image:
#  path: /assets/images/2023-04-06-hacking-my-cars-climate-controls-part-2/device-wide.jpg
#  width: 1251
#  height: 727
---

Amplify is one of Amazon Web Services' coolest offerings. It’s a great way to get started building and running software in the cloud, even if all you know is how to build a frontend and what a database does. A few commands can get you a fully built back end, complete with authentication, a database, and a GraphQL API for reading and writing to your database. You even get automatically generated frontend code for handling authentication and communicating with your API. Amplify was my first foray into cloud development and infrastructure as code, and for getting a simple web application online quickly with high availability and scalability, it’s a dream come true.

This simplicity comes with a major pitfall—rigidity. Amplify is powerful, but it can’t do everything. It tries to abstract as much as it can to make it easier to build something as fast as possible, but limits how much you can change what it generates. There are options for overriding many of its predefined structures, but not all.

Imagine you want to deploy a simple full stack application. The first version of this application only consists of simple CRUD operations and basic authentication. You deploy it using Amplify and it works well enough.

The application requirements change. Authentication gets more complicated with different user types. Business logic is introduced that requires custom GraphQL queries and data types. This _can_ be done with Amplify, but requires overriding the automatically generated code, and there are limits to what you can override, how it can be organized, and how it's deployed.

Your application continues to grow. Your custom resolver file grows to contain hundreds of resolvers and functions, without much organization because there’s only so much you can do to organize resources inside a single CloudFormation stack. You have dozens of Lambdas, with shared resources being copied between them manually and dependencies being built into each one separately because the Amplify CLI is finicky with detecting layer updates. Your data access patterns adapt to changing business requirements and NoSQL is no longer the best fit for your data. You want to build a separate admin-only API that uses the same database and might share some code with the user-facing API, but can't because Amplify only supports creating one GraphQL API.

It’s time to move on from Amplify. But how? You can't just build a new backend from scratch while keeping your old API key, and you can't export user accounts from Cognito, so you're stuck with Amplify forever, right?

Wrong. There is a way out.

### How Amplify Works

Amplify, at its core, is a wrapper around CloudFormation. CloudFormation is a low-level tool for deploying AWS resources. You create a JSON file (or set of JSON files) describing the resources you want to deploy and their configurations and CloudFormation deploys them to your AWS account. Unfortunately, it's not very user-friendly. It's hard to read and write, and it's easy to make mistakes. Amplify abstracts away the complexity of CloudFormation and provides a simple CLI for deploying resources, then generates a CloudFormation template based on your configuration and deploys it to your AWS account.

Think of Amplify as a what-you-see-is-what-you-get (WYSIWYG) editor for your infrastructure, compared to CloudFormation's raw HTML/CSS/JS. It's great for getting started, but once you want to do something more complex, you need to move on to a more powerful tool. While you _could_ switch to managing the CloudFormation templates directly, there are better options. (Imagine trying to edit a React app by editing the HTML directly. It's possible, but why would you?)

### Moving On

Amazon's Cloud Development Kit (CDK) is another really cool infrastructure as code tool. It allows you to deploy AWS resources by writing code in your favorite language, whether that be TypeScript, Python, or Go, and also deploys by generating CloudFormation templates. It's a lot more powerful and flexible than Amplify, while still abstracting the frustrating parts of CloudFormation away.

This makes it a great tool for migrating from Amplify. Because Amplify generates CloudFormation templates, you can write CDK to "take over" your Amplify resources. You can do this gradually, ensuring a seamless transition once your CDK app is developed enough that Amplify is no longer necessary.

### The Migration Process

The migration process is fairly simple. You'll need to do the following:
* Set up a CDK app
* Import your generated Amplify resources into your CDK app
* Verify that your CDK app generates the same CloudFormation templates as Amplify and make any necessary changes so the templates match
* Rebuild each Amplify stack in your CDK app, one at a time, using the same names as your Amplify resources, and continually verify that the generated templates match
* Once all your resources have been migrated, deploy your CDK app and verify that everything works as expected

For this example, I'll be migrating my BYU Ride Board application from Amplify to CDK. This application is a simple full stack application that allows students to post and search for rides to and from BYU. It uses a GraphQL API for data access and authentication, and DynamoDB for storing data. It also uses Cognito for authentication and authorization.

## Getting started
Begin by deploying a new branch of your amplify app to use for rebuilding in CDK. you may choose to deploy this branch in a separate AWS account to ensure you don’t accidentally break or destroy a resource on one of your existing branches.

### Setting up your CDK app
Install the CDK tool kit. Before deploying a CDK app, you will need to bootstrap the account you wish to deploy in

Create a new directory for your project and navigate to it in a terminal. Create a new CDK app in your language of choice. I prefer Typescript because most CDK documentation is written for Typescript.
cdk init app --language=typescript

This will create all the files necessary for a basic CDK application:
* `bin/` - contains the entry point for your CDK app.
* `lib/` - contains the stacks that make up your CDK app. The root stack is defined in lib/your-app-name-stack.ts. Other stacks can be defined in separate files and imported into the root stack.
* `test/` - contains tests for your CDK app. These are written using Jest. We can ignore this for now.
* `cdk.json` - contains configuration for your CDK app. This will rarely need to be changed.
* `package.json` - used by npm to manage dependencies and scripts for your CDK app.

There are others, but these are the most important.

If you run `cdk diff` now, you should see that if you were to deploy now, a single stack would be created with no resources. You'll also see the `cdk.out` directory, which contains the CloudFormation templates that would be deployed if you were to run `cdk deploy` (which you should not do yet).

### Basic configuration

We need to make some changes to the default configuration to make the migration easier.

#### `cdk.json`

After "app", add these two properties:
```json
"versionReporting": false,
"pathMetadata": false,
```

This will prevent CDK from adding extra version and path metadata to your CloudFormation templates. The goal of this migration is to make the templates generated by CDK as similar as possible to the ones generated by Amplify, so we don't want any extra metadata. This will make it easier to compare the two and ensure we're not missing anything. If you run `cdk diff` again, you'll see that far less data is included in the diff.

#### Line endings

You'll want to make sure your line endings are set to LF. By default, Git will convert line endings to CRLF on Windows, which will cause problems when comparing the CloudFormation templates generated by Amplify and CDK. You can set this globally by running `git config --global core.autocrlf input` in a terminal.

You should also set it for your project. Add these two files to the root of your project:

* `.gitattributes`
```
* text=auto eol=lf
```

* `.editorconfig`
```
[*]
end_of_line = lf
```

This will tell Git to convert all files to LF when committing or pulling, and tell your editor to use LF for new files. You can convert existing files to LF by running `git rm --cached -r . && git reset --hard` in a terminal after committing the `.gitattributes` file, or in IntelliJ by selecting the root directory and running `File > File Properties > Line Separators > LF`.

#### Git

When you initialize a CDK app, it also initializes the directory as a git repository, so you can start working with git immediately using the CLI or your favorite git client (I use SmartGit).

With Amplify, if you are using git to manage branches, you need to checkout the Amplify environment after switching branches. If you don't, you might accidentally deploy to the wrong environment. Because CDK is much more customizable, we can just use git to manage branches and environments.

First, rename your `master` branch to the name of the environment you'll be using for the migration. For this example, I'll be using `dev`. If you already have a `dev` branch where active development is taking place, you may want to use a different name.

#### Config files

Next, add a `config` directory to the root of your project. Create two files, `env.json` and `branches.json`. These files will be used to store environment and branch configuration, respectively.

`env.json`

```json
{
  "account": "594264796139",
  "region": "us-west-2",
  "appName": "rideboard" // This should match the name of your Amplify app
}
```

`branches.json` should contain an empty object for each branch you'll be using. This will be used to store branch-specific configuration, similar to how Amplify stores branch-specific configuration in `amplify/team-provider-info.json`.

```json
{
  "dev": {},
  "prod": {}
}
```

One of my pain points with Amplify is that `team-provider-info.json` does not remain consistent between branches, and sometimes git merging will cause issues. To avoid this, you can setup `config` as a git submodule. This will allow you to keep the configuration files in a separate repository, and you can use a git client to manage them separately from your CDK app. This is not necessary, but I recommend it.

#### Install dependencies

```
npm i lodash @types/lodash
```

---------------

* add AmplifyStack
* setup bin/your-app-name.ts
* add amplify templates from amplify/#current-cloud-backend/awscloudformation
* add cfn import to amplify-stack.ts
* rebuild resources one at a time, verifying that the generated templates match
  * remove resource with `this.removeCfnResource('resourceId')`
  * L1 (Cfn) resources use given id
  * L2 resources add a hash to the id, so use `this.overrideId(resource, 'id')` to set the id
  * remove outputs with `this.removeCfnOutput('outputId')`
  * can ignore resources that will remain stable (auth, amplify hosting) but need to rebuild nested stacks or links will be messed up when switching branches
  * sometimes things like description won't match what's in cfn. Amplify will occasionally update the description in the templates it generates, but cfn sometimes doesn't update the description when the resource is updated. However, there is a bug which causes a description provided in CDK to be prepended to the template's description. The only way around this is to change the description in the cfn template, then set it in CDK so it will be included once the cfn template is no longer imported. Depending on when you created each branch, this will result in inconsistencies in diffs, so pay attention to that and make sure only the description is different. After all branches are moved over, adding a tag to the root stack will force cfn to update or remove descriptions on all nested stacks.
  * taking over a stack will cause an update to TemplateURL in the parent stack, which will show in the diff. This is expected and can be ignored.
* use feature flags to test deployment step by step
* cannot set parameters for root stack, but existing parameters won't change, just plan on eventually removing them, and if you need to create a new stack with cdk you'll have to specify them in the cdk command
