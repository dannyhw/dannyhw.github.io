---
title: "Quick guide for @storybook/react-native-server v6"
description: "A short guide to get you started with react-native-server in the v6 beta"
pubDate: "Dec 03 2022"
heroImage: "/rnsbserverscreenshot.png"
---

### What is react-native-server

The package @storybook/react-native-server is used alongside @storybook/react-native to control the current story and adjust controls remotely via a storybook UI on a simple web app.

This was a popular tool in storybook/react-native v5 and recently we were able to bring it back for v6.

Note: The web UI will not show a preview of any component and your components will only show on the mobile device.

### The setup

Initialise a project with react-native storybook configured (replace AwesomeStorybook with your own name)

```sh
expo init --template expo-template-storybook AwesomeStorybook
```

Add the react-native-server package

```sh
yarn add -D @storybook/react-native-server@next
```

Add a folder called `.storybook_server/` in the root of your project and create a main.js file in there with the following content

```js
module.exports = {
  stories: ["../components/**/*.stories.?(ts|tsx|js|jsx)"],
  env: () => ({}),
  addons: ["@storybook/addon-essentials"],
};
```

Add a script to start the server in package.json

```json
"start-server": "react-native-storybook-server"
```

Edit .ondevice/Storybook.tsx and adjust the options so that websockets are enabled and optionally disable the ondevice UI.

```js
const StorybookUIRoot = getStorybookUI({
  enableWebsockets: true,
  onDeviceUI: false,
});
```

If you disable the ondevice UI you might find it useful to add a top margin so that nothing gets hidden under the status bar. In the same Storybook file you can add that as follows.

```jsx
import { View } from "react-native";
export default () => (
  <View style={{ marginTop: 52, flex: 1 }}>
    <StorybookUIRoot />
  </View>
);
```

Start your app and open the ios or android app with 'i' or 'a' like you might normally with expo

```sh
yarn start
```

Now you can run the server with the command like

```sh
yarn start-server
```

Then open the url of the server that it will print out

<image alt="Image of the server and ondevice storybook" src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ymzg0i8g2mzuj4d10ihu.png" style={{height:"400px"}}>

### Some caveats

As mentioned the package is companion app for react-native storybook it also relies on the device to send the story list via websockets. Sometimes its useful to refresh the mobile app once the web ui has loaded so that it receives the story list and current story. It then uses websockets to update the current story on the device whenever you navigate to a new story on the web UI.

You must use storybook 6.5.14 and above along with react-native storybook 6.0.1-beta.11 or newer for this to work due to patches made in those versions.

### Feedback

Thanks for reading and let me know if you try it out

This release and many of the recent updates have been made possible due to the help of multiple people on the storybook core team.

Specifically I'd like to thank [Norbert](https://twitter.com/NorbertdeLangen), [Shillman](https://twitter.com/mshilman) and [Tom](https://twitter.com/tmeasday) who all paired with me multiple times to help me overcome various blockers.

If you want to ask anything leave a comment or DM me on twitter [@Danny_H_W](https://twitter.com/Danny_H_W)
