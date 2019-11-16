# 02 CSS Modules

In this demo we are going to isolate different scss files using same css class names.
We will learn how to configure it and how to deal with external css class provided by third libraries like Bootstrap.

We will start from sample _03 environments/01 linting_.

Summary steps:

- Update `webpack.config.js` with CSS Modules config.
- Add scss file with averageComponent styles.
- Create other component and scss file with same class name.
- Create selector using custom class and Bootstrap class.

# Steps to build it

## Prerequisites

Prerequisites, you will need to have nodejs installed in your computer. If you want to follow this step guides you will need to take as starting point sample _01 linting_.

## steps

- `npm install` to install previous sample packages:

```
npm install
```

- We can start creating `AverageComponent` styles file (SASS file):

_./src/averageComponentStyles.scss_

```css
$background: teal;

.result-background {
  background-color: $background;
}
```

- Let's go to use this style in `AverageComponent`:

_./src/averageComponent.tsx_

```diff
import React from 'react';
import { getAvg } from './averageService';

export const AverageComponent: React.FunctionComponent = () => {
  const [average, setAverage] = React.useState<number>(0);

  React.useEffect(() => {
    const scores: number[] = [90, 75, 60, 99, 94, 30];
    setAverage(getAvg(scores));
  }, []);

  return (
    <div>
-     <span>Students average: {average}</span>
+     <span className="result-background">Students average: {average}</span>
    </div>
  );
};

```

- Finally we need to update `webpack.config` to load `.scss` file, as we usually load it in other samples:

_./webpack.config.js_

```diff
...
  entry: {
    app: ['regenerator-runtime/runtime', './students.tsx'],
-   appStyles: ['./mystyles.scss'],
+   appStyles: ['./mystyles.scss', './averageComponentStyles.scss'],
    ...
  },
  ...
};

```

- Run example:

```bash
npm start
```

- If we run the the sample we can check that the style is being applied to the component (background color), but we are using global css names.

- Now we are going to create `totalScoreComponent` creating the component [./src/totalScoreComponent.tsx](./src/totalScoreComponent.tsx) and its style [/src/totalScoreComponentStyles.scss](/src/totalScoreComponentStyles.scss) :

_./src/averageService.ts_

```diff
- function getTotalScore(scores: number[]): number {
+ export function getTotalScore(scores: number[]): number {
  return scores.reduce((score, count) => score + count);
}

export function getAvg(scores: number[]): number {
  return getTotalScore(scores) / scores.length;
}

```

_./src/totalScoreComponent.tsx_

```javascript
import React from "react";
import { getTotalScore } from "./averageService";

export const TotalScoreComponent: React.FunctionComponent = () => {
  const [totalScore, setTotalScore] = React.useState < number > 0;

  React.useEffect(() => {
    const scores: number[] = [10, 20, 30, 40, 50];
    setTotalScore(getTotalScore(scores));
  }, []);

  return (
    <div>
      <span className="result-background">
        Students total score: {totalScore}
      </span>
    </div>
  );
};
```

_./src/totalScoreComponentStyles.scss_

```css
$background: indianred;

.result-background {
  background-color: $background;
}
```

> NOTE: we declare same class name for TotalScoreComponent

- Using `TotalScoreComponent`:

_./src/students.tsx_

```diff
import React from 'react';
import ReactDOM from 'react-dom';
import { AverageComponent } from './averageComponent';
+ import { TotalScoreComponent } from './totalScoreComponent';
const logoImg = require('./content/logo_1.png');

$('body').css('background-color', 'lightSkyBlue');

const img = document.createElement('img');
img.src = logoImg;

document.getElementById('imgContainer').appendChild(img);

ReactDOM.render(
  <div>
    <h1>Hello from React DOM</h1>
    <AverageComponent />
+   <TotalScoreComponent />
  </div>,
  document.getElementById('root')
);

```

- And update `webpack.config`:

_./webpack.config.js_

```diff
...
  entry: {
    ...
    appStyles: [
      './mystyles.scss',
      './averageComponentStyles.scss',
+     './totalScoreComponentStyles.scss',
    ],
    ...
  },
  ...
};

```

- Now if we run the project, we will find that both css names overlaps.

```
npm start
```

- As result, averageComponentStyles are overridden by totalScoreComponentStyles. How to solve this? _CSS Modules to the rescue!_

- CSS Modules scope is to isolate different scss files using same css class names. We are going to replace `adding styles via webpack entry point` by `import styles where we need it`. Let's configure it:

_./webpack.config.js_

```diff
...
  entry: {
    ...
    appStyles: [
      './mystyles.scss',
-     './averageComponentStyles.scss',
-     './totalScoreComponentStyles.scss',
    ],
    ...
  },
  module: {
    rules: [
      ...
      {
        test: /\.scss$/,
        exclude: /node_modules/,
        use: [
          MiniCssExtractPlugin.loader,
-         'css-loader',
+         {
+           loader: 'css-loader',
+           options: {
+             modules: {
+               localIdentName: '[name]__[local]__[hash:base64:5]',
+             },
+           },
+         },
          {
            loader: 'sass-loader',
            options: {
              implementation: require('sass'),
            },
          },
        ],
      },
      ...
    ],
  },
  ...
};

```

- Updating `AverageComponent`:

_./src/averageComponent.tsx_

```diff
import React from 'react';
import { getAvg } from './averageService';
+ const classes = require('./averageComponentStyles.scss');

export const AverageComponent: React.FunctionComponent = () => {
  ...

  return (
    <div>
-     <span className="result-background">
+     <span className={classes['result-background']}>
        Students average: {average}
      </span>
    </div>
  );
};

```

- Updating `TotalScoreComponent`:

_./src/totalScoreComponent.tsx_

```diff
import React from 'react';
import { getTotalScore } from './averageService';
+ const classes = require('./totalScoreComponentStyles.scss');

export const TotalScoreComponent: React.FunctionComponent = () => {
  ...

  return (
    <div>
-     <span className="result-background">
+     <span className={classes['result-background']}>
        Students total score: {totalScore}
      </span>
    </div>
  );
};

```

- Now we get the right isolation.

```bash
npm start
```

- To avoid reference classNames like `classNames['result-background']`, webpack provides us to transform `kebab case` to `camelCase`:

_./webpack.config.js_

```diff
...
  module: {
    rules: [
      ...
      {
        test: /\.scss$/,
        exclude: /node_modules/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]__[local]__[hash:base64:5]',
              },
+             localsConvention: 'camelCase',
            },
          },
          ...
        ],
      },
      ...
    ],
  },
  ...
};

```

- Updating components:

_./src/averageComponent.tsx_

```diff
...
  return (
    <div>
-     <span className={classes['result-background']}>
+     <span className={classes.resultBackground}>
        Students average: {average}
      </span>
    </div>
  );
};

```

_./src/totalScoreComponent.tsx_

```diff
...

  return (
    <div>
-     <span className={classes['result-background']}>
+     <span className={classes.resultBackground}>
        Students total score: {totalScore}
      </span>
    </div>
  );
};
```

- Running `npm start` again:

```bash
npm start
```

- If we take a look to the browser console, we can see how webpack transform css class names, adding prefixes (inspect element).

- Finally, let's do an example where we need to add styles to element that has a Bootstrap class:

_./src/averageComponent.tsx_

```diff
...

  return (
    <div>
      <span className={classes.resultBackground}>
        Students average: {average}
      </span>
+     <div className={`jumbotron ${classes.resultBackground}`}>
+       <h1>Jumbotron students average: {average}</h1>
+     </div>
    </div>
  );
};

```

- Running `npm start`, it looks a bit weird:

- Let's go to add own styles:

_./src/averageComponentStyles.scss_

```diff
$background: teal;
+ $jumbotronBackground: darkseagreen;

.result-background {
  background-color: $background;
}

+ .jumbotron.result-background {
+   background-color: $jumbotronBackground;
+   display: block;
+ }

```

- Running `npm start` nothing changes, why? Due to webpack is transform `local` class names to `'[name]__[local]___[hash:base64:5]'` we need to tell him that jumbotron is a `global` style ([more info](https://webpack.js.org/loaders/css-loader/#-css-modules-https-github-com-css-modules-css-modules-)):

_./src/averageComponentStyles.scss_

```diff
$background: teal;
$jumbotronBackground: darkseagreen;

.result-background {
  background-color: $background;
}

- .jumbotron.result-background {
+ :global(.jumbotron).result-background {
  background-color: $jumbotronBackground;
  display: block;
}

```

- If we run the project now we can see that the change has been applied globally.

```bash
npm start
```
