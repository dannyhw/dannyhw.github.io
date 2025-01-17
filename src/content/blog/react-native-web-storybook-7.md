---
title: "How to use React Native Web with Storybook v7 beta"
description: "A quick guide on trying out Storybook v7 with addon-react-native-web"
pubDate: "Feb 12 2023"
heroImage: "/blog-placeholder-4.jpg"
---

You may have heard that Storybook is getting close to a v7 release. If you're using addon-react-native-web you might be wondering how to try out the beta. In this quick guide I will show you how to setup a new project and add `@storybook/addon-react-native-web` Storybook to the project.
This enables you to use the web Storybook with your react native components.

If you already have this setup in an existing project it should be pretty straight forward to get updated. Change the version of `@storybook/addon-react-native-web` to `next` and update all your other relevant storybook dependencies to the latest v7 beta version (you can use `next` here too). Then you should check this [migration guide](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#from-version-65x-to-700) to see if you need to update anything to get your project ready for v7.

To setup a fresh new project you can follow this guide:

First I'll generate a project.

```sh
yarn create expo-app
```

Next I'll open up the project and in the root of the project run

```sh
npx sb@next init --type react
```

After its done add `@storybook/addon-react-native-web@next` and the dependencies for it.

```sh
yarn add --dev react-dom react-native-web babel-plugin-react-native-web @storybook/addon-react-native-web@next
```

Add the addon to .storybook/main.js as follows:

```js
const config = {
  /* existing config */
  addons: [/*existing addons,*/ "@storybook/addon-react-native-web"],
  /* more config here */
};
export default config;
```

If you're familiar with the Storybook config you'll notice there are some slight differences in the `main.js` file. One cool new feature is that you can now get benefits of autocomplete even if you aren't using typescript for your config files. This change comes from the comment above the config object that looks like this

```js
/** @type { import('@storybook/react-webpack5').StorybookConfig } */
const config = {
```

Now that the setup is done, let's add a React Native component and a story in the stories folder.

You may want to delete the ReactJS example components. To do that delete everything inside of the stories folder.

```sh
rm -rf stories/*
```

Then in that folder you can add this example button component.

Inside `stories/` create `Button.jsx`

```jsx
// stories/Button.jsx
import React from "react";
import { StyleSheet, Text, TouchableOpacity, View } from "react-native";

const styles = StyleSheet.create({
  button: {
    paddingVertical: 8,
    paddingHorizontal: 16,
    borderRadius: 4,
    alignSelf: "flex-start",
    flexGrow: 0,
    backgroundColor: "purple",
  },
  buttonText: {
    color: "white",
    fontSize: 16,
    fontWeight: "bold",
  },
  buttonContainer: {
    alignItems: "flex-start",
    flex: 1,
  },
});

export const Button = ({ text, onPress, color, textColor }) => (
  <View style={styles.buttonContainer}>
    <TouchableOpacity
      style={[styles.button, !!color && { backgroundColor: color }]}
      onPress={onPress}
      activeOpacity={0.8}
    >
      <Text style={[styles.buttonText, !!textColor && { color: textColor }]}>
        {text}
      </Text>
    </TouchableOpacity>
  </View>
);
```

Now add the files `Button.stories.jsx` in the `stories` folder with this content:

```jsx
// stories/Button.stories.jsx
import { Button } from "./Button";

export default {
  component: Button,
};

export const Basic = {
  args: {
    text: "Hello World",
    color: "purple",
  },
};
```

One of the really cool updates in V7 is the introduction of CSF3 which you can see above. There is now much less boilerplate needed to write a story!

At this point all thats left to do is run `yarn storybook` and get to work building your components!

![button in the storybook ui](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/byl54ykbmyddlirsh46c.png)

Thanks for reading.

Follow me on twitter [@Danny_H_W](https://twitter.com/Danny_H_W)
