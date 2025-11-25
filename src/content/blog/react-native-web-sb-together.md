---
title: "React Native and React Native Web Storybook together"
description: "How to use react native storybook with react native web storybook"
pubDate: "July 04 2023"
heroImage: "/rnsbwebnative.jpg"
---

Storybook for react native allows you to develop your components in isolation on their native platforms, but its not the easiest to share with stakeholders or team members. Running your components on the web using react native web can be a nice way to make your components more shareable and you can integrate with lots of web tooling built around storybook for the web. It's possible to merge the setup for react native storybook together with the react native web storybook.

Its not been the easiest setup process for new users so I've made this quick guide for those with an existing project wanting to integrate `@storybook/react-native` and `@storybook/addon-react-native-web` in the same project.

If you are creating a fresh project it would be easier to start with one of the templates I've already created.

I'll be starting with a basic expo template generated with `yarn create expo-app`

## Ondevice setup

Now to add @storybook/react-native I'll run the following command:
`npx sb@latest init --type react_native`

Then in my App.js file I'll make some changes to import the storybook ui from `.storybook` and export it as the the entry point of the app.

```js
// App.js
import StorybookUI from "./.storybook";

export default StorybookUI;
```

Now I'm going to customize metro config to complete the setup

This command will generate the config file: `npx expo customize metro.config.js`

```js
// metro.config.js
const { getDefaultConfig } = require("expo/metro-config");

const config = getDefaultConfig(__dirname);

config.resolver.resolverMainFields.unshift("sbmodern");

module.exports = config;
```

Add a folder called `components` and lets add a basic component called Button

```jsx
// components/Button.js
import { Text, TouchableOpacity } from "react-native";

export const Button = ({ onPress, text }) => {
  return (
    <TouchableOpacity
      onPress={onPress}
      style={{
        borderColor: "lightgrey",
        borderWidth: 1,
        borderRadius: 2,
        paddingVertical: 8,
        paddingHorizontal: 32,
        alignSelf: "flex-start",
      }}
    >
      <Text>{text}</Text>
    </TouchableOpacity>
  );
};
```

```jsx
// components/Button.stories.js
import { Button } from "./Button";

export default {
  title: "Button",
  component: Button,
  argTypes: {
    onPress: { action: "onPress" },
  },
};

export const Base = {
  args: { text: "Hello World" },
};
```

lets update the `main.js` config file to look for stories in this folder

```js
// .storybook/main.js
module.exports = {
  stories: ["../components/**/*.stories.?(ts|tsx|js|jsx)"],
  addons: [
    "@storybook/addon-ondevice-controls",
    "@storybook/addon-ondevice-actions",
  ],
};
```

Lets also update the `preview.js` to add some padding to our stories

```jsx
// .storybook/preview.js
import { View } from "react-native";

export const parameters = {};

export const decorators = [
  (Story) => (
    <View style={{ padding: 8 }}>
      <Story />
    </View>
  ),
];
```

Next I'll run `storybook-generate` to make sure the `storybook.requires.js` file has been generated.

```sh
yarn storybook-generate
```

This file is generated based on your config and handles importing all the necessary story files.

This concludes the on-device part of the setup. You should be able to run `yarn ios` or `yarn android` and see storybook render your button story.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ch4t7afun9xiforkdubd.jpg)

## Web setup

The best react-native-web Storybook experience will come from using `@storybook/addon-react-native-web` on top of a ReactJS storybook setup. This is a slightly more complex setup because it requires you to have two sets of storybook config and scripts. However those tradeoffs give you the ability to use all the features of web storybook and tooling that comes with it.

Since React Native Storybook is currently at compatibility with 6.5 we'll be using that version for both web and on-device.

First install the dependencies

```sh
yarn add -D @storybook/addon-react-native-web @storybook/addon-essentials@^6.5 @storybook/builder-webpack5@^6.5 @storybook/manager-webpack5@^6.5 @storybook/react@^6.5 babel-plugin-react-native-web react-native-web @babel/plugin-proposal-export-namespace-from
```

Now add the scripts to `package.json`

```json
"storybook:web": "start-storybook -p 6006 -c .storybook-web",
"build-storybook": "build-storybook -c .storybook-web"
```

Create a new folder called `.storybook-web` and add the files `main.js` and `preview.js`

```js
// ./storybook-web/main.js
module.exports = {
  stories: [
    "../components/**/*.stories.mdx",
    "../components/**/*.stories.@(js|jsx|ts|tsx)",
  ],
  addons: ["@storybook/addon-essentials", "@storybook/addon-react-native-web"],
  core: {
    builder: "webpack5",
  },
  framework: "@storybook/react",
};
```

```js
// ./storybook-web/preview.js
import { View } from "react-native";

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};

export const decorators = [
  /* by wrapping everything in a view the parent container is always using flexbox and behaves like a react-native-web component would  */
  (Story) => (
    <View>
      <Story />
    </View>
  ),
];
```

Now if you run `yarn storybook:web` you should see something like this.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lv16rr017j09a8vxzzvf.jpg)

Likely this won't be the end of the configuring and setup you'll need to do but this should get you most of the way.

Thanks for reading.

I'm on twitter at [@Danny_H_W](https://twitter.com/Danny_H_W) and github at [dannyhw](https://github.com/dannyhw)
