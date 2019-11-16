# 05 JQuery

So far we have made good progress on our journey... but we are lacking one of the
basic pillars of web development, consuming third party libraries.

In this demo we will install a legacy library (jquery) via npm, define it as global, and use it. Finally we will end up creating a separate bundle for libraries.

We will start from sample _00 intro/04 output_.

Summary steps:
  - Prerequisites:
    - nodejs packages must be installed
    - (Optional) starting point _00 intro/04 output_
  - Install jQuery packages via npm
  - Setup global variables

# Steps to build it

## Prerequisites

Prerequisites, you will need to have nodejs installed (at least v 8.9.2) on your computer. If you want to follow this guide steps you will need to take as starting point sample _00 intro/04 output_.

## Steps

- `npm install` to install previous sample packages:

```bash
npm install
```

- Let's start by downloading the jquery library via npm. In this case we will execute the following command on the command prompt: ```npm install jquery --save```.
>Note down: this time we are not adding the `-d` suffix to the parameter, this time the jquery package is a dependency of the web app not of the build process.

```
npm install jquery --save
```

It automatically adds that entry to our _package.json_.

_package.json_

```diff
  ...
+  "dependencies": {    
+    "jquery": "^3.3.1"    
  ...
+  }
```

- Since this is a legacy library it expects to have a global variable available.
Instead of assigning this manually let's define it in the `webpack.config.js` file.

- First we will require an import "webpack" at the top of the file:

```diff
const HtmlWebpackPlugin = require('html-webpack-plugin');
+ const webpack = require('webpack');

module.exports = {
```

- Then we will use a plugin from webpack to define jQuery and $ as global variables.

```diff
  plugins: [
    //Generate index.html in /dist => https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html', //Name of file in ./dist/
      template: 'index.html', //Name of template in ./src
      hash:true,
    }),
+   new webpack.ProvidePlugin({
+     $: "jquery",
+     jQuery: "jquery"
+   }),
  ],
```

- Now it's ready to be used. Just to test it, let's change the background color of the page body to blue. Let's change the background of the body element using jquery:

_./students.js_

```diff
import { getAvg } from './averageService';

+ $('body').css('background-color', 'lightSkyBlue');

const scores = [90, 75, 60, 99, 94, 30];
const averageScore = getAvg(scores);

const messageToDisplay = `average score ${averageScore}`;

document.write(messageToDisplay);

```
## Running and Testing it
- Now we can just execute the app (```npm start```) and check how the background of the page has changed from white to blue.

```bash
npm start
```

> Now that we have the sample running, try to update _student.js_ and change the color from _lightSkyBlue_ to _red_ and save the file you will notice that automatically the browser gets refreshed.
