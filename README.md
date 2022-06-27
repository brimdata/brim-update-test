# Zui Insiders

A stand-alone app for early adopters to try out the latest features and fixes coming to Zui. 

This repository contains scripts responsible for building & publishing a new version each weeknight or manually.

### Features

Zui Insiders has the following features:

* Its own icon
* Its own data directory
* Runs side-by-side with Zui stable
* Builds nightly off `brimdata/zui#main`
* Subscribes to this repository's releases for automatic updates
* May occasionally break

### How It Works

Everything happens in the Github Actions workflow named `release.yml`. It runs the following steps:

1. Checkout  `brimdata/zui#main` 
2. Inject the `package.json` file with properties for Zui Insiders
3. Build the app
4. Publish a new release with the build artifacts

### Injecting package.json

We use [Electron Builder](https://www.electron.build/) to build the app. When building, it references package.json to set things like name, version, and repository. One of the scripts in this repo is called "inject".

````
yarn inject <path_to_app_dir>
````

It will set the app's package.json to the correct values for the next Insider's release.

Example:

```
brimdata/zui-insiders % yarn inject ../zui
‣ Injecting app's package.json with: {
  name: 'zui-insiders',
  productName: 'Zui - Insiders',
  repository: 'https://github.com/brimdata/zui-insiders',
  description: 'Zui for early adoptors with frequent updates.',
  version: '0.30.1-2'
}
```

In this example, I have the zui-insiders repo as a sibling of the zui repo on my file system.

```
brimdata/zui
brimdata/zui-insiders
```

When I ran yarn inject and passed the path the app directory, it created this diff in the zui repo.

```diff
--- a/package.json
+++ b/package.json
@@ -1,11 +1,11 @@
 {
-  "name": "zui",
+  "name": "zui-insiders",
-  "productName": "Zui",
+  "productName": "Zui - Insiders",
-  "description": "Zui Desktop App",
+  "description": "Zui for early adoptors with frequent updates.",
-  "repository": "https://github.com/brimdata/zui",
+  "repository": "https://github.com/brimdata/zui-insiders",
-  "version": "0.30.0",
+  "version": "0.30.1-0",
   "main": "dist/js/electron/main.js",
   ...
```

Now the app will be built with desired properties. 

We also run electron-builder with a special configuration file that overrides the icons and the release strategy. This file lives in the main `zui`repo.

```
yarn electron-builder -c electron-builder-insiders.json
```

And that's how it works.

### Versioning

How do we determine what the next version of Insiders should be?

We use the latest Insiders release tag version and the latest Zui version to determine what the next Insider's version should be. 

Running `yarn latest` in this repo will make an HTTP request to Github's API and print the version tag of the latest release.

```
brimdata/zui-insiders % yarn latest
0.30.1-4
```

The algorithm for choosing the version of each release is:

```
if stableVersion > lastInsidersVersion
	use the stable version
else
	increment the lastInsidersVersion by one "prerelease"
```

Incrementing by a "prerelease" means first bumping the patch number, then adding a numerical suffix that increments by one each time.

```js
semver.inc(version, "prerelease")
/* 
   version     returns
   0.30.0      0.30.1-0
   0.30.1-0    0.30.1-1
   0.30.1-1    0.30.1-2
   and so on ...
*/
```

> It's important to know that the semver specification says  `0.1.0` is higher than `0.1.0-0`. That's why the patch version gets bumped initially.



This release process is very much based on [VSCode Insiders](https://code.visualstudio.com/insiders/).
