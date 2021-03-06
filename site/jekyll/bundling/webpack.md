---
layout: docs
title: Webpack
---

#### 👀  Just want to see the code? Check out the [sample project](https://github.com/reactjs/React.NET/tree/main/src/React.Template/reactnet-webpack).

## For new projects:

```
dotnet new -i React.Template
dotnet new reactnet-webpack
dotnet run
```

## For existing projects:

[Webpack](https://webpack.js.org/) is a popular module bundling system built on top of Node.js. It can handle not only combination and minification of JavaScript and CSS files, but also other assets such as image files (spriting) through the use of plugins. Webpack is the recommended bundling solution and should be preferred over Cassette or ASP.NET Bundling.

Your project will bundle its own copy of react and react-dom with webpack, and ReactJS.NET will be used only for server-side rendering.

Copy from the sample project to the root of your project:

- [package.json](https://github.com/reactjs/React.NET/blob/main/src/React.Template/reactnet-webpack/package.json), which includes everything you need to bundle with webpack
- [webpack.config.js](https://github.com/reactjs/React.NET/blob/main/src/React.Template/reactnet-webpack/webpack.config.js), which contains the configuration needed for webpack to create the bundles
- [.babelrc](https://github.com/reactjs/React.NET/blob/main/src/React.Template/reactnet-webpack/.babelrc), which contains the Babel settings needed to compile JSX files

Run `npm install` to start the package restore process.

Then, create the `Content/components/expose-components.js` file which will be the entrypoint for both your client and server-side Javascript.

```javascript
// Content/components/expose-components.js

import React from 'react';
import ReactDOM from 'react-dom';
import ReactDOMServer from 'react-dom/server';

import RootComponent from './home.jsx';

// any css-in-js or other libraries you want to use server-side
import { ServerStyleSheet } from 'styled-components';
import { renderStylesToString } from 'emotion-server';
import Helmet from 'react-helmet';

global.React = React;
global.ReactDOM = ReactDOM;
global.ReactDOMServer = ReactDOMServer;

global.Styled = { ServerStyleSheet };
global.Helmet = Helmet;

global.Components = { RootComponent };
```

Once Webpack has been configured, run `npm run build` to build the bundles. Once you have verified that the bundle is being created correctly, you can modify your ReactJS.NET configuration (normally `App_Start\ReactConfig.cs`) to load the newly-created bundle.

Reference the runtime, vendor, and main app bundles that were generated:

```csharp
ReactSiteConfiguration.Configuration
  .SetLoadBabel(false)
  .SetLoadReact(false)
  .AddScriptWithoutTransform("~/dist/runtime.js")
  .AddScriptWithoutTransform("~/dist/vendor.js")
  .AddScriptWithoutTransform("~/dist/main.js");
```

This will load all your components into the `Components` global, which can be used from `Html.React` to render any of the components:

```csharp
// at the top of your layout
@using React.AspNet

@Html.React("Components.RootComponent", new {
  someProp = "some value from .NET"
})
```

Reference the built bundle directly in a script tag at the end of the page in `_Layout.cshtml`:

```html
// at the top of your layout
@using React.AspNet

<head>
  <link rel="stylesheet" href="/dist/main.css">
</head>
<body>
  @RenderBody()
  <script src="/dist/runtime.js"></script>
  <script src="/dist/vendor.js"></script>
  <script src="/dist/main.js"></script>
  @Html.ReactInitJavascript()
</body>
```

A full example is available in [the ReactJS.NET repository](https://github.com/reactjs/React.NET/tree/main/src/React.Template/reactnet-webpack).

### 💡  Beta feature: Asset manifest handling

An asset manifest is generated by the `webpack-asset-manifest` plugin, written to `asset-manifest.json`. See the webpack config example above for details on how to set this up. This manifest file contains a list of all of the bundles required to run your app. To use it, call `.SetReactAppBuildPath("~/dist")`. You may still provide exact paths to additional scripts by calling `AddScriptWithoutTransform("~/dist/path-to-your-file.js")`.

```csharp
ReactSiteConfiguration.Configuration
  .SetLoadBabel(false)
  .SetLoadReact(false)
  .SetReactAppBuildPath("~/dist");
```

Then, make calls to `@Html.ReactGetScriptPaths()` and `@Html.ReactGetStylePaths()` where you would normally reference styles and scripts from your layout.

```html
// at the top of your layout
@using React.AspNet

<head>
  @Html.ReactGetStylePaths()
</head>
<body>
  @RenderBody()

  @Html.ReactGetScriptPaths()
  @Html.ReactInitJavascript()
</body>
```
