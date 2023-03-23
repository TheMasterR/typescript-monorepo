# Getting Started with npm Workspaces

The newest major release of  [npm](https://npmjs.com/), launched in October 2020, came out with a very anticipated feature (at least for me). The 7th version of the package manager introduced  **Workspaces**.

The name may sound familiar. Other package managers such as  [Yarn](https://classic.yarnpkg.com/en/)  and  [pnmp](https://pnpm.io/)  already ship with Workspaces for quite a while now. In fact, npm is not trying to reinvent the wheel. You can find similarities between all three Workspace implementations.

But what are Workspaces for? Workspaces help us managing repositories with multiple packages - more than one  `package.json`  file. In projects like this, you usually have a complex dependency tree, with many packages depending on each other. This special type of repository is known as a  **monorepo**.

In this post, you will see how npm Workspaces work, how to get started, and a comparison with other Workspace implementations. I’ve also provided a  [repository](https://github.com/ruanmartinelli/npm-workspaces-demo)  on GitHub with some sample code from the examples.

## What changes with Workspaces?

Now, when you run  `npm install`  in a multi-package repository,  [npm’s dependency tree manager](https://github.com/npm/arborist)  is smart enough to scan your folders looking for all dependencies to install.

Dependencies are hoisted, meaning they get installed in the root  `node_modules`  folder. This is done for performance reasons: if a dependency is shared by multiple packages, it gets saved only once in the root.

Your packages (the ones you created) also get symlinked in the root  `node_modules`  folder. If two of your packages depend on each other, they get the reference from there.

For example, if you have this structure:

```
.
├── package.json
└── packages
    ├── package-a
    │   └── package.json # Dependencies: `lodash`
    └── package-b
        └── package.json # Dependencies: `lodash`, `package-a`
```

After running  `npm install`, your  `node_modules`  folder will look like this:

```
.
├── node_modules
│   ├── lodash # `lodash` is installed in the root `node_modules/`
│   ├── package-a # package-a is symlinked
│   └── package-b # package-b is symlinked
├── package.json
└── packages
    └── # ...

```

If you have packages using the same dependency but on different versions, npm will create a  `node_modules`  folder inside of one of the packages. For example, suppose  `package-a`  uses  `lodash@4`  while  `package-b`  is still on  `lodash@3`:

```
.
├── package.json
└── packages
    ├── package-a
    │   └── package.json # Dependencies: `lodash@4`
    └── package-b
        └── package.json # Dependencies: `lodash@3`
```

When running  `npm install`, you will get the following result:

```
.
├── node_modules
│   ├── lodash # This will be `lodash` version 4.x
│   └── # ...
├── package.json
└── packages
    ├── package-a
    │   └── package.json
    └── package-b
        │   node_modules # `node_modules` is created inside the package
        │      └── lodash # This is `lodash` version 3.x
        └── package.json
```

These are just a few examples to illustrate how the new dependency resolution works on Workspaces, there’s certainly a lot more happening under the hood.

## Getting started

### Setting up

You can try out Workspaces today by updating your npm to version 7. To update, run this command on your terminal:

```
npm install -g npm@7
```

If you install Node.js 15 today, it should already come with npm 7.

You can use a similar structure as the ones showed before as a starting point for playing around with workspaces. For example:

```
.
├── package.json
└── packages
    ├── package-a
    │   └── package.json
    └── package-b
        └── package.json
```

Notice you still need a  `package.json`  file in the root of your repository, even if it has no dependencies. In fact, it’s on that file that you define the location where your packages live.

Edit the root  `package.json`  to reference your packages:

```
// ./package.json
{
  // ...
  "workspaces": ["./packages/*"]
}
```

Now, when you run  `npm install`  in the root of your repository, npm will be smart enough to install  `package-a`  and  `package-b`’s dependencies.

I made a simple GitHub repository with this example in case you wanna check it in more detail. Here’s the link:  [npm-workspaces-demo](https://github.com/ruanmartinelli/npm-workspaces-demo).

### Using Workspaces

The npm CLI also introduced the  `--workspace`  and  `--workspaces`  flags. These flags can be added to many of the existing npm commands to run them in your sub-packages, instead of your root package.

For example, imagine  `package-a`  and  `package-b`  both have a script named “test” defined in their  `package.json`  files:

```
// packages/package-a/package.json
{
  "name": "package-a",
  "scripts": {
    "test": "echo 'all package-a tests passed!'"
  }
  // ...
}

// packages/package-b/package.json
{
  "name": "package-b",
  "scripts": {
    "test": "echo 'all package-b tests passed!'"
  }
  // ...
}
```

You can run all the “test” scripts at once by adding the  `--workspaces`  (plural) to your  `npm run`  command:

```
# Run "test" script on all packages
npm run test --workspaces

# Tip - this also works:
npm run test  -ws
```

That will give you the following output:

```
> package-a@1.0.0 test
> echo 'all package-a tests passed!'

all package-a tests passed!

> package-b@1.0.0 test
> echo 'all package-b tests passed!'

all package-b tests passed!
```

To run a command for a specific package, add the  `--workspace`  (singular) flag:

```
# Runs "test" only on package-a
npm run test --workspace package-a

# Tip - this also works:
npm run test -w package-a
```

The  `install`  command also accepts the  `--workspace`  flag:

> Important! This was only introduced on npm@7.14.0

```
# Install `lodash` on `package-a`
npm install lodash --workspace package-a

# Install `tap` on `package-b` as a dev dependency
npm install tap --workspace package-b --save-dev

# Install `package-a` on `package-b`
npm install package-a --workspace package-b

# Install `eslint` in all packages
npm install eslint --workspaces
```

## npm Workspaces vs. Yarn Workspaces

Yarn is the second biggest package manager for JavaScript, so it might be fair to make a comparison.

Yarn Workspaces is around for much longer. It was launched somewhere around 2017. The  `yarn workspaces`  interface includes some extra tooling that npm is still catching up on. For example, an equivalent of  `yarn workspace <workspace> add <dependency>`  (adding a dependency to a workspace)  is still in the works  **Update: this feature was added on v7.14.0!**.

You can also see some minor differences in the CLI (e.g.  `npm --workspaces`  vs.  `yarn workspaces`), but the overall concepts remain the same. Yarn is constantly cited as prior art in the  [RFCs](https://github.com/npm/rfcs). I would be surprised to see big disparities between both CLIs.

Despite the differences, npm Workspaces is iterating fast. Since I first wrote this post in November 2020, many features have been released and RFCs have been accepted. It shouldn’t take long for npm to reach feature parity with Yarn and other package managers.

## Using npm Workspaces today

_“I’m starting a new monorepo project today, can I use npm Workspaces?”_

You can, but keep reading.

If you are building a small monorepo project and need npm to just resolve your dependencies in a smart way, npm Workspaces alone should do the trick. You will get a similar experience as if you were using Yarn Workspaces.

But if you’re working on a bigger project, with multiple packages and a complex dependency tree, you might want to combine npm with a tool like  [Lerna](https://github.com/lerna/lerna).

Lerna has many  [useful commands](https://lerna.js.org/#commands)  to manage monorepos. Yarn has  [stated before](https://classic.yarnpkg.com/en/docs/workspaces/#toc-how-does-it-compare-to-lerna)  that the goal of Yarn Workspaces is to provide low-level primitives for tools such as Lerna to use, not to compete with them. While npm hasn’t given a similar statement, I suspect they will follow the same approach as Yarn.
