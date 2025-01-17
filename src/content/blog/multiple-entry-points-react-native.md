---
title: "Multiple entry points for React Native Storybook"
description: "Use metro to create multiple entry points for React Native Storybook"
pubDate: "Apr 16 2022"
heroImage: "/blog-placeholder-2.jpg"
---

## Intro

React Native storybook is setup in such a way that the decision is left to the user how it should be loaded. This gives you flexibility on how you do things, however I often get asked how to easily switch between running application code and Storybook.

### An alternative approach

For a while now I've been thinking about an easy way to do this by using metro directly.

This will be a quick guide on how to setup a secondary entry point for React Native

At the end you'll be able to run `yarn storybook-metro` when you want Storybook and `yarn start` when you want to run your app.

The key to this solution looks like this:

```
react-native start --config ./.ondevice/metro.storybook.js
```

This solution works with React Native cli I haven't worked out a solution for expo yet, but maybe someone can help me with that.

## The setup

I'll be working from the [react native storybook template](https://github.com/dannyhw/react-native-template-storybook) based on the react native typescript template. However your own setup should work too if you adjust it in the same way.

To use it you can run

```console
npx react-native init MyApp --template react-native-template-storybook
```

At the time of writing when you use this template your directory structure will look something like this

```
.ondevice/
  Storybook.tsx
  main.js
  ...
components/
index
App.tsx
...
```

The files in `.ondevice` are related to React Native storybook and currently the `App.tsx` file imports the default export from `.ondevice/Storybook.tsx` and renders that as the Application.

What if you want to write an application here and run storybook separately, what do you do?

You might just comment out the storybook import when you aren't using it, or you might add some kind of conditional to load storybook when an environment variable is set.

There is another way, you can actually use metro to do all the work for you.

### Creating a second entry point

First I'm going to rename `.ondevice/Storybook.tsx` to `.ondevice/index.js` and replace the content with this:

```js
import { AppRegistry } from "react-native";
import { getStorybookUI } from "@storybook/react-native";
import { name as appName } from "../app.json";
import "./storybook.requires";

const StorybookUIRoot = getStorybookUI({});

AppRegistry.registerComponent(appName, () => StorybookUIRoot);
```

This will act as the secondary entry point for Storybook and when used registers the Storybook UI as the Application to render just like before.

Now in `App.tsx` lets just remove all storybook related code and make it look like this

```tsx
import React from "react";
import { SafeAreaView, Text } from "react-native";
export default () => {
  return (
    <SafeAreaView>
      <Text>Not Storybook!</Text>
    </SafeAreaView>
  );
};
```

This App file gets loaded by index in the root of the project which is the original entry point. This component will be registered as the app when that entry point is used.

We have two entry-points `.ondevice/index.js` and `index.js`. Now we just need a way to swap between them.

Create a file in the `.ondevice/` folder called `metro.storybook.js` with the following content:

```js
const path = require("path");
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: false,
      },
    }),
  },
  resolver: {
    resolverMainFields: ["sbmodern", "react-native", "browser", "main"],
  },
  watchFolders: [path.resolve(__dirname, "..")],
};
```

The important thing here is the watchFolders config so that metro knows to look for files in the root of the project.

We're also using the way metro works to our advantage, by default it will look for an index file relative to the projectRoot. Since we didn't set the root it will assume that the current directory is the project root.

We're now mostly done, all thats left is to tell metro which config file to load.

## The final pieces

Now we just setup some scripts in package json to make our life easier

The template we used gives us these commands

```json
"scripts": {
	...
    "start": "react-native start",
    "prestart": "yarn update-stories",
    ...
},
```

The `prestart` script uses the a prefix to tell yarn to always run this script before the start script.

Now that we've updated our setup we'll be using start specifically for our application code so we'll adjust this.

Let's add a new script for starting metro from our new storybook config file.

```json
"storybook-metro": "react-native start --config ./.ondevice/metro.storybook.js",
```

Here we use the --config option to tell metro to use our storybook specific config.

Now rename the prestart script to look like this:

```json
"prestorybook-metro": "yarn update-stories",
```

Now the update-stories script only runs when we need it (when running storybook).

### Running storybook

Whenever you want to run storybook you run `yarn storybook-metro` in one terminal followed by running `yarn ios` or `yarn android` in another terminal.

To go back to your app you can just stop storybook metro and run `yarn start` instead. This will start metro with config from `metro.config.js` and all you have to do is reload the app to get the application javascript bundle.

Note that since only the bundle changes you won't need to rebuild the entire app.

## Closing thoughts

As you can see it's possible to separate your storybook code from your application code with just the built in behaviour of metro and React Native.

A great benefit of this is that you can easily leave storybook code outside of your application code since the entry points are completely separate.

This setup could be the default for storybook however a solution is still needed for expo.

In this guide I'm using the React Native Storybook v6 beta which is currently under development however you can follow a similar approach for v5.3 also.

### Contact me

If you want to get in touch you can contact me on twitter or the storybook discord

- Twitter: [@Danny_H_W](https://twitter.com/Danny_H_W)
- Github: [dannyhw](https://github.com/dannyhw)
- Storybook discord: [Storybook](https://discord.gg/sMFvFsG) (find me in the react-native channel)
