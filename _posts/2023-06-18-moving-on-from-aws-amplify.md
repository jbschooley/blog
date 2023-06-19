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


Getting started
Begin by deploying a new branch of your amplify app to use for rebuilding in CDK. you may choose to deploy this branch in a separate AWS account to ensure you don’t accidentally break or destroy a resource on one of your existing branches.

Setting up CD K
Install the CDK tool kit. Before deploying a CD K app, you will need to bootstrap the account you wish to deploy in

Create a new CDK app in your language of choice (I prefer Typescript)

