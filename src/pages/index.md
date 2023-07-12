---
layout: base
title: Conventional Scripts
query: '{
    pages {
        frontmatter {
            title
        }
        relativePath
    }
}'
---

# Conventional Scripts

## Introduction

The **Conventional Scripts** specification is a convention built on top of [npm scripts](https://docs.npmjs.com/cli/v9/using-npm/scripts), aiming to streamline the naming of `scripts` within the `package.json` file for npm packages. It provides a consistent way for developers to set up their packages, execute common development tasks, and harmonize the development workflow. Inspired by [Conventional Commits](https://www.conventionalcommits.org/), this specification promotes a standardized approach to script naming.

## Applications & Libraries

This specification distinguishes between applications and libraries. An **application** refers to a CLI, website, or web application that is intended to be executed.

On the other hand, a **library** refers to a collection of reusable code and functionalities that are packaged together and made available for developers to use in their applications.

While an application can depend on libraries, it is not considered a library itself. In contrast to libraries, applications can be deployed and started. Consequently, they typically include a `deploy` and `start` script.

## Script Naming Convention

The Conventional Scripts specification defines a consistent structure for script names: `category[:type:configuration]`

Each part of the structure serves a specific purpose:

1. **Category:** Groups scripts of a common type and is mandatory. It identifies the general purpose or action performed by the script.
2. **Type:** Optional and used when multiple scripts of the same category are needed. It helps differentiate similar scripts within a category.
3. **Configuration:** Optional and used to create scripts that pass optional parameters or options to existing types.

## Categories

A category can serve the purpose of executing a single tool, or when utilizing types, it can act as a parent script to execute all types within a shared category.

The Conventional Scripts specification defines the following categories:

1. **build:** Used to automate the process of building or preparing an application to be started or deployed.
1. **check:** Used to run code quality tools, such as linters or spellcheckers, on the package's source code.
1. **clean:** Used to remove generated or temporary files and directories from the package.
1. **deploy:** Used to execute the necessary steps required to deploy an application.
1. **dev:** Used to automate the development environment setup or run the application with debugging capabilities (e.g., `node --inspect`).
1. **docs:** Used to generate or build documentation for the package.
1. **fix:** Used to automatically fix or correct code issues.
1. **log:** Used to generate or update a changelog file for the package.
1. **[start](https://docs.npmjs.com/cli/v9/using-npm/scripts#npm-start):** Used to start an application.
1. **[test](https://docs.npmjs.com/cli/v9/using-npm/scripts#npm-test):** Used to automate the execution of tests for the project.

**Note:** The scripts mentioned above with hyperlinks are npm built-in scripts. For more information on these scripts, refer to the npm documentation.

**Example:**

Below, you can find a configuration using Conventional Scripts categories without types:

```json
{
  "scripts": {
    "build": "stasis-cli build -c stasis.config.json src",
    "check": "eslint --ext .js .",
    "clean": "rimraf dist",
    "deploy": "gh-pages -d dist",
    "dev": "node --inspect -r ts-node/register src/start.ts",
    "docs": "typedoc --plugin typedoc-plugin-markdown --plugin typedoc-github-wiki-theme",
    "fix": "npm run check -- --fix",
    "log": "generate-changelog -M",
    "start": "stasis-cli serve -c stasis.config.json src",
    "test": "jest src --passWithNoTests"
  }
}
```

## Types

Depending on your requirements, there may be a need for multiple tools within a specific category. For instance, when performing code testing, you may have various levels of tests, such as type testing, unit testing, and end-to-end testing. To accommodate this scenario, Conventional Scripts introduces the concept of types.

**Types** allow for differentiation of scripts within the Conventional Scripts categories, enabling you to specify different tools or approaches for distinct testing levels.

While the Conventional Scripts specification does not prescribe specific type names, it promotes the practice of naming scripts based on the actions or concerns they address. This approach allows for better clarity and flexibility when switching between tools. Below is an example demonstrating this convention.

**Good Example 😃:**

The code snippet provided uses names that effectively describe the actions being performed, such as executing end-to-end tests, conducting static type checking, and performing unit testing. This naming approach enables developers to easily switch between tools without the need to modify the script names. For instance, it allows for seamless transitions from using [Jest](https://jestjs.io/) to [Vitest](https://vitest.dev/) while maintaining consistent script names:

```json
{
  "scripts": {
    "test": "npm run test:type && npm run test:unit && npm run test:e2e",
    "test:e2e": "playwright test",
    "test:type": "tsc --noEmit",
    "test:unit": "jest ./src --passWithNoTests"
  }
}
```

**Bad Example 🥲:**

In the given code, the scripts are named after specific tools, which tightly couples them to the technology being used. This naming approach leads to inflexibility when attempting to switch or replace tools:

```json
{
  "scripts": {
    "test": "npm run test:type && npm run test:unit && npm run test:e2e",
    "test:playwright": "playwright test",
    "test:tsc": "tsc --noEmit",
    "test:jest": "jest ./src --passWithNoTests"
  }
}
```

**Another Good Example 😃:**

The following code demonstrates a symbiotic relationship between the "check" and "format" categories. The "check" types encompass various tests related to code formatting, static analysis, and spelling.

Building upon the "check" types, the "fix" types leverage the results of the checks to automatically correct any identified issues, whenever possible:

```json
{
  "scripts": {
    "check": "run-p check:*",
    "check:format": "prettier --ignore-path .gitignore .",
    "check:lint": "eslint --ext .js .",
    "check:typo": "cspell -c .cspell.json \"*.md\"",
    "fix": "run-p fix:*",
    "fix:format": "npm run check:format -- --write",
    "fix:lint": "npm run check:lint -- --fix"
  }
}
```

**Note:** Josh Goldberg has written an excellent article that effectively explains the [distinction between a formatter and a linter](https://blog.joshuakgoldberg.com/the-blurry-line-between-formatting-and-style/).

## Configuration

Following Conventional Scripts, additional options can be added to types. The naming principle still holds: it is advisable to name configurations based on their purpose rather than using names that are specific to the tools being executed.

Here's an example:

```json
{
  "scripts": {
    "test": "npm run test:unit",
    "test:unit": "jest ./src --passWithNoTests",
    "test:unit:coverage": "npm run test:unit --coverage"
  }
}
```

In the example above, the `"test:unit:coverage"` script includes a configuration that enables code coverage reports.

## Best Practices

### Shorthand Helpers

To execute all types of a certain category, popular packages like [Prettier](https://prettier.io/) use shorthand helpers like [npm-run-all](https://www.npmjs.com/package/npm-run-all) ([source](https://github.com/prettier/prettier/blob/2.8.8/package.json#L155)). With helpers like these, the maintenance becomes significantly easier, as wildcards can be employed. This eliminates the need to update the **category** every time new **types** are added to the scripts section.

**Example:**

```json
{
  "scripts": {
    "test": "run-p lint:*",
    "test:e2e": "playwright test",
    "test:type": "tsc --noEmit",
    "test:unit": "jest ./src --passWithNoTests"
  }
}
```

### Placeholder Scripts

Even if you do not have a specific need for a category, we strongly recommend defining it in your scripts (using `exit 0`) to prevent encountering a "Missing script" error when your code is executed by a developer or CI tool that relies on Conventional Scripts. By including the category, you ensure that all required script references are available and avoid potential issues during execution.

**Example:**

```json
{
  "scripts": {
    "build": "exit 0",
    "check": "exit 0",
    "clean": "exit 0",
    "deploy": "exit 0",
    "dev": "exit 0",
    "docs": "exit 0",
    "fix": "exit 0",
    "log": "exit 0",
    "start": "node src/server.js",
    "test": "exit 0"
  }
}
```

Without placeholder scripts, developers will need to execute your scripts with additional flags like `npm run deploy --if-present` to avoid encountering errors.

## Feedback

We welcome feedback on the proposed script names and appreciate it if you provide it through our [GitHub repository](https://github.com/bennycode/conventionalscripts.org). We are open to receiving pull requests for fixing typos and improving descriptions. Your contributions and suggestions are valuable to us, and we encourage your active participation in enhancing the specification. 📝
