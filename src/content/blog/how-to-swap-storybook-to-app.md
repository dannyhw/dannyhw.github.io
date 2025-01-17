---
title: "How to swap between React Native Storybook and your app"
description: "How to use expo constants and environment variables to swap between storybook and your app"
pubDate: "Feb 06 2023"
heroImage: "/blog-placeholder-3.jpg"
---

_EDIT: I have recently updated this guide to show a few different approaches I will update further if I find better solutions_

Storybook in react native is a component that you can slot into your app and isn't its own separate runtime which you might be used to on the web.

If you want to run storybook in the same environment as your app we're going to need a simple way to switch between them.

To access an environment variable with expo we can follow their documentation [here](https://docs.expo.dev/guides/environment-variables/). If you aren't using expo I recommend following [this guide](https://dev.to/dannyhw/multiple-entry-points-for-react-native-storybook-4dkp) instead.

Note: This guide assumes you have an expo application setup for storybook (see the [github readme](https://github.com/storybookjs/react-native/blob/next-6.0/README.md) for info on how to get setup).

## Using Expo constants

First rename app.json to app.config.js and edit the file so it looks something like this:

```js
module.exports = {
  name: "my-app",
  slug: "my-app",
  version: "1.0.0",
  orientation: "portrait",
  ...
};
```

Now add the `extra` config option like this

```js
module.exports = {
  name: 'MyApp',
  version: '1.0.0',
  extra: {
    storybookEnabled: process.env.STORYBOOK_ENABLED,
  },
  ...
};
```

Now we can access the storybookEnabled variable from our app using expo constants like this:

```js
import Constants from "expo-constants";
const apiUrl = Constants.expoConfig.extra.storybookEnabled;
```

Earlier I said to comment out the app function in the App.js file and this where we go back and fix that.

Edit App.js so that it looks like this:

```jsx
import { StatusBar } from "expo-status-bar";
import { StyleSheet, Text, View } from "react-native";
import Constants from "expo-constants";

function App() {
  return (
    <View style={styles.container}>
      <Text>Open up App.js to start working on your app!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

let AppEntryPoint = App;

if (Constants.expoConfig.extra.storybookEnabled === "true") {
  AppEntryPoint = require("./.storybook").default;
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    alignItems: "center",
    justifyContent: "center",
  },
});

export default AppEntryPoint;
```

Now in package.json we can add a script like this

```json
"scripts": {
	"storybook": "STORYBOOK_ENABLED='true' expo start",
}
```

We've added a new command that will run our app with the STORYBOOK_ENABLED flag set to true.

When you run `yarn start` you should see the app code and `yarn storybook` should show you storybook.

## Using entryPoint (expo go <48 only)

_This solution won't work in v48 of expo or with expo dev clients._

First we want to rename our app.json to app.config.js and edit the file so it looks something like this:

```js
module.exports = {
  name: "my-app",
  slug: "my-app",
  version: "1.0.0",
  orientation: "portrait",
  ...
};
```

Add the `entryPoint` config option like this

```js
module.exports = {
  name: 'MyApp',
  version: '1.0.0',
  entryPoint: process.env.STORYBOOK_ENABLED
      ? "index.storybook.js"
      : "node_modules/expo/AppEntry.js",
  ...
};
```

Add a file called index.storybook.js in the root of the project with the following content

```js
import { registerRootComponent } from "expo";

import Storybook from "./.storybook";

// registerRootComponent calls AppRegistry.registerComponent('main', () => App);
// It also ensures that whether you load the app in Expo Go or in a native build,
// the environment is set up appropriately
registerRootComponent(Storybook);
```

Now in package.json we can add a script like this

```json
"scripts": {
  "storybook": "STORYBOOK_ENABLED='true' expo start",
}
```

Which will run our app with the STORYBOOK_ENABLED flag set to true.

Now when you run `yarn start` you should see the app and `yarn storybook` should show you storybook.

You can pass the option `--ios` or `--android` to have the simulator autmatically open up otherwise press `i` or `a` after the command finishes running to open the simulator.

## Summary

As you can see there are multiple potential approaches here and I've tried to offer some possible solutions. Each of these have some potential drawbacks but you should assess your projects needs to see what fits for you.

If you are using Expo Go on version 47 or below the entryPoint might be an interesting option for you but it will be removed in 48. The version using expo constants has the possibility increase your bundle size however will work in expo 48.

Heres some comments from Evan Bacon about the entryPoint field:

import { Tweet } from "mdx-embed";

<Tweet tweetLink="https://twitter.com/Baconbrix/status/1624817071291858944?s=20" />

<Tweet tweetLink="https://twitter.com/Baconbrix/status/1624817852023701506?s=20" />

Other possible approaches include using env variables in metro config to change the project root or using storybook as a separate app outside of your project. You could also manually alternate the `main` field of the `package.json` between two different entry point paths (similar to the entryPoint option).

As I find better solutions I will add to this post to showcase the best options.

Thanks for reading.

You can follow me on twitter here: https://twitter.com/Danny_H_W
