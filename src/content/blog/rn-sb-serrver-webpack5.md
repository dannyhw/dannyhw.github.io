---
title: "How to use React Native Storybook Server with webpack 5"
description: "A quick guide on how to use the React Native Storybook Server beta with webpack 5"
pubDate: "Mar 18 2023"
heroImage: "/blog-placeholder-5.jpg"
---

This is a quick guide for the prerelease version (6.5.0-rc.6) with a stable release coming out soon for React Native Storybook 6.5.

The React Native Storybook Server is an optional companion app for react native storybook that lets you control the storybook on your mobile device via a web interface.
Until recently webpack 5 was not supported in the react native storybook server. Recently that support was added by a contributor [@leonardo-fc](https://github.com/leonardo-fc) and is now available in the release candidate version of the server.

## The Guide

First bootstrap an app with react native storybook

```sh
yarn create expo-app --template expo-template-storybook AwesomeStorybook1
```

Go to the directory of the new app

```sh
cd AwesomeStorybook1
```

Add the react native storybook server package and some webpack5 dependencies to your project

```sh
yarn add -D @storybook/react-native-server@next webpack @storybook/builder-webpack5 @storybook/manager-webpack5
```

Make sure you add all of these packages otherwise it won't work.

Add the following script to your `package.json` file

```json
{
  "scripts": {
    "start-server": "react-native-storybook-server"
  }
}
```

Create a folder called `.storybook_server` and add a main.js file with the following content

```js
module.exports = {
  stories: [
    "../components/**/*.stories.mdx",
    "../components/**/*.stories.@(js|jsx|ts|tsx)",
  ],
  env: () => ({}),
  core: {
    builder: "webpack5",
  },
  addons: ["@storybook/addon-essentials"],
};
```

By setting the builder to `webpack5` we are telling the storybook server to use webpack 5 instead of the default webpack 4.

In this bootstrapped project the ondevice storybook is configured in the `.ondevice` folder.
In the index.tsx file we need to update some options so that websockets are enabled.

```tsx
const StorybookUIRoot = getStorybookUI({
  enableWebsockets: true,
});
```

Now you can start the server with `yarn start-server` and open the storybook on your mobile device with `yarn storybook`.

Make sure you restart the app on your device so that the server can pick up the stories sent over the websockets (sent on app startup).

At this point you should be running the react native server with webpack 5.

Thanks for reading. If you have any questions or feedback please tweet or send me a message [@Danny_H_W](https://twitter.com/Danny_H_W).
