---
title: "Automatic args in React Native Storybook"
description: "How to use docgen to generate args and arg types for your React Native Storybook based on your typescript types."
pubDate: "Nov 21 2021"
heroImage: "/tech-generic-3.jpg"
---

This is a quick guide on how to setup automatic args for React Native Storybook V6 (currently in beta).

If you've used the web version of Storybook you'll know that your types are used to infer the types of your args. This is done using a transform called docgen in the Storybook webpack bundle process.

## Docgen for React Native?

In React Native Storybook you might have noticed that this doesn't happen by default. That's because we don't bundle a separate app like in the web so it's not possible to insert a transform into the bundling of the app. In order to do this you the user need to add it to your configuration.

Recently I worked with Michael Shillman one of the core maintainers of Storybook to find a way to make this work for React Native. We came up with something that works for typescript types but requires some setup from user (for the reasons mentioned above).

## The setup

To use docgen in React Native Storybook you will need at least version `6.0.1-beta.10` and you will want to have types for your component props.

Then you will want to install these packages:

```bash
yarn add -D @storybook/react @storybook/docs-tools babel-plugin-react-docgen-typescript
```

Then add the docgen plugin to your babel config:

```js
plugins: [['babel-plugin-react-docgen-typescript', { exclude: 'node_modules' }]],
```

Now add the following code to `Storybook.tsx`

```tsx
import { extractArgTypes } from "@storybook/react/dist/modern/client/docs/extractArgTypes";
import { addArgTypesEnhancer, addParameters } from "@storybook/react-native";
import { enhanceArgTypes } from "@storybook/docs-tools";

addArgTypesEnhancer(enhanceArgTypes);
addParameters({
  docs: {
    extractArgTypes,
  },
});
```

You can also add it to another file and import it from `Storybook.tsx`.

Once this is done then you should have Args and ArgTypes without needing to manually specify them.

Thanks for reading!
Please try it out and let me know if it worked for you.

twitter: [Danny_H_W](https://twitter.com/Danny_H_W)
github: [dannyhw](https://github.com/dannyhw)

Shoutout to [@mshilman](https://twitter.com/mshilman) for helping make this possible.
