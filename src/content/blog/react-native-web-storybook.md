---
title: "Introducing React Native Web Storybook"
description: "Announcing react native web storybook"
pubDate: "Nov 14 2021"
heroImage: "/sb-screenshot.png"
---

Storybook is a UI development workshop for components and pages. It's used by many teams for all kinds of reasons. You can develop components in isolation, run visual tests and even validate the accessibility of your UI.

Storybook started off web focused and has expanded to many different platforms since then. One of those platforms is React Native.

Storybook for React Native is a catalog of all your components and states, that runs inside your iOS or Android app, on your mobile device. This is great for seeing your React Native components in their production setting, but also has some disadvantages when compared to Storybook's web UI.

That's why I'm excited to announce [Storybook for React Native Web](https://www.npmjs.com/package/@storybook/addon-react-native-web), a new complementary approach to developing and sharing React Native components in Storybook.

## A different take on Storybook for React Native

I've been working together with [Michael Shilman](https://twitter.com/mshilman) on a new approach for React Native Storybook that has all the functionality of Storybook on the web. This is achieved by using an [addon](https://www.npmjs.com/package/@storybook/addon-react-native-web) we built that uses the react-native-web library to make your components available in the browser. All you have to do is set up the addon in your project.

In the video below I am interacting with React Native components directly in the browser. The components displayed use features from popular React Native libraries such as the animation library react-native-reanimated, and the common library for handling interactions react-native-gesture-handler.

<iframe
  width="100%"
  height="480"
  style="margin: 1rem 0;"
  src="https://www.youtube.com/embed/zv6dKY7hxlc"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

Heres the [example storybook](https://618d44792e48fd003a482f88-fgqejyulkb.chromatic.com/?path=/story/libraries-gesture-handler-draggable--basic) embedded below (use the link if the embed doesn't work)

<iframe
  style="margin: 1rem 0;"
  width="100%"
  height="600"
  src="https://618d44792e48fd003a482f88-fgqejyulkb.chromatic.com/?path=/story/libraries-gesture-handler-draggable--basic"
  class="Box-jLJQJw dFiwvv"
></iframe>

## The challenge of React Native

React Native has its own Storybook implementation at [@storybook/react-native](https://github.com/storybookjs/react-native), however it has some limitations.

Storybook for React Native is built on a different architecture than Storybook for web frameworks (React, Vue, Angular, etc.), and it is built as a separate project. Because of this architectural difference a lot of effort is required to achieve feature parity with Storybook for web.

Storybook for React Native supports only a small portion of the addons and features that exist for Storybook on the web platform. This also means external tools like visual testing for Storybook need a custom implementation in order to support react native. Unfortunately, this leads to users having an incomplete experience for React Native in comparison with the web

Ideally, Storybook for React Native should support all popular addons and existing tools used on Storybook for Web. Is this possible?

Well, if you're here you probably guessed that the answer is yes. There are of course some limitations, however I think that for many use cases it will be worth it. The main use case that comes to mind is UI library maintainers that want to show off their React Native components in an interactive way.

### React Native components on the Web?

You might be surprised at the number of React Native libraries that already fully support the web platform. Also if you've been following the React Native scene you might know that facebook is putting a strong emphasis on the [many platform vision](https://reactnative.dev/blog/2021/08/26/many-platform-vision).

These are some of the top downloaded libraries for React Native (based on npm numbers):

- react-navigation
- react-native-screens
- react-native-gesture-handler
- react-native-svg
- react-native-reanimated
- react-native-vector-icons

All of these libraries have support for the web platform in some way. In fact I've tested many of these popular libraries with this new addon and they all worked very well on the web.

Now, I would like to share with you the project I've been working on, which should help bridge the gap today for React Native and Storybook, allowing you to have full functionality that you know and love from Storybook for web.

## Introducing @storybook/addon-react-native-web

This addon provides you with a simple setup for using the ReactJS Storybook in a React Native project. You can run it locally or easily deploy it as a website for others to see. The best part is that since it's running on the web with the ReactJS configuration most addons should work with no changes necessary.

That means:

- Much better UI/UX.
- Popular addons such as [Docs](https://storybook.js.org/addons/@storybook/addon-docs/) supported out of the box.
- [Chromatic](https://www.chromatic.com/) and other visual testing tools supported.
- Easily publish your Storybook and share it with stakeholders such as designers and product owners.
- Full support for the latest Storybook features.

There are however a few drawbacks:

- Some libraries won't have web support
- The components may not display exactly as they would on a native device
- Some extra config needed for libraries that aren't transpiled

If those drawbacks aren't dealbreakers, then I think you will be happy with the results. However If you are unable to use this package because you need a native mobile experience then consider using [@storybook/react-native](https://github.com/storybookjs/react-native). You could even use both if it makes sense for your use case.

If you are concerned about components being different on the web please see the [FAQ](https://github.com/storybookjs/addon-react-native-web/blob/main/FAQ.md) where I have tried to address this.

I've also provided some example config in [the project readme](https://github.com/storybookjs/addon-react-native-web#configuring-popular-libraries)

Now the fun part!

### Getting started

Heres the steps to follow if you want to try this out yourself.

Create your react native project if you haven't already

```sh
npx react-native init AwesomeApp --template react-native-template-typescript
cd AwesomeApp
```

Initialize Storybook.

```sh
npx sb init --type react
```

You can delete the example stories if you like (not necessary).

```sh
rm -rf stories/*
```

Now add @storybook/addon-react-native-web and it's dependencies

```sh
yarn add --dev react-dom react-native-web babel-plugin-react-native-web @storybook/addon-react-native-web
```

Open up .storybook/main.js and add the addon to the addons list.

```javascript
module.exports = {
  /* existing config */
  addons: [/*existing addons,*/ "@storybook/addon-react-native-web"],
};
```

Now that the setup is done, let's add a React Native component and a story in the stories folder.

Here's an example of a button component:

```tsx
// stories/MyButton.tsx
import React from "react";
import { StyleSheet, Text, TouchableOpacity, View } from "react-native";

export type ButtonProps = {
  onPress: () => void;
  text: string;
  color?: string;
  textColor?: string;
};

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

export const MyButton = ({ text, onPress, color, textColor }: ButtonProps) => (
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

And a story for that component:

```tsx
// stories/MyButton.stories.tsx
import React from "react";
import { ComponentMeta, ComponentStory } from "@storybook/react";

import { MyButton } from "./MyButton";

export default {
  title: "components/MyButton",
  component: MyButton,
} as ComponentMeta<typeof MyButton>;

export const Basic: ComponentStory<typeof MyButton> = (args) => (
  <MyButton {...args} />
);

Basic.args = {
  text: "Hello World",
  color: "purple",
};
```

Now run the Storybook with `yarn storybook` and it should open in your browser.

You should see something like this:

<img
  style="width:100%;"
  src="https://user-images.githubusercontent.com/3481514/140612591-554d0b2c-d15d-4e07-afa8-087400134d01.png"
/>

Running on the web without a single div or span in sight! 🎉

Thanks for reading this far, I would appreciate hearing your thoughts on this! If you want to see an example project showcasing Storybook for React Native and web using libraries such as reanimated and gesture handler, make sure to check this [repository](https://github.com/dannyhw/addon_react_native_web_example) out.

I'm one of the maintainers of @storybook/react-native, currently working hard on improving it and preparing a 6.0 release, so keep an eye out! I would love to talk more about my vision for the upcoming release of Storybook for React Native in a future post, so let me know if you're interested!

You can find me on twitter or discord in the #react-native channel of the Storybook discord server:

- Twitter: [@Danny_H_W](https://twitter.com/Danny_H_W)
- Github: [dannyhw](https://github.com/dannyhw)
- Storybook discord: [Storybook](https://discord.gg/sMFvFsG)

Big thank you to [Michael Shilman](https://twitter.com/mshilman) for setting up the repository and helping me figure out all the Storybook config. Also for the time spent and guidance. This wouldn't have been possible without his hard work!

Thanks to everyone who helped me edit this post and gave feedback.
