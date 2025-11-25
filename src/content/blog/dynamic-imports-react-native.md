---
title: "Dynamic imports supported in react native"
description: "With the release of react native 0.72 and the latest metro changes we now have access to `require.context` which powers expo router and has potential to power other developer tooling or libraries."
pubDate: "July 26 2023"
heroImage: "/dynamic-requires.png"
---

With the release of react native 0.72 and the latest metro changes we now have access to `require.context` which powers expo router and has potential to power other developer tooling or libraries.

This got me really excited because I can finally solve one of the biggest problems with react native storybook which is that you needed to generate imports for your stories. I wanted to talk a bit about what this new feature is and why I think its really cool.

Shout out to [@Baconbrix](https://twitter.com/Baconbrix) for contributing this feature to metro!

## What is require.context?

From the [webpack documentation](https://webpack.js.org/guides/dependency-management/#requirecontext):

> It allows you to pass in a directory to search, a flag indicating whether subdirectories should be searched too, and a regular expression to match files against.

Historically for react native we've never been able to do dynamic imports, neither based on a variable or a regex. However this kind of functionality has been in webpack and other web bundlers for some time. Now that we have access to require.context we can build tools like storybook and expo router without needing to generate imports at build time, we can instead dynamically import them at runtime.

Heres some examples:

```javascript
require.context(
  directory,
  (useSubdirectories = true),
  (regExp = /^\.\/.*$/),
  (mode = "sync"),
);
```

```javascript
require.context("../", true, /\.stories\.js$/);
```

The result of require context is a object like interface (RequireContext) that has two main functions.

- keys: returns an array of keys (ids) which will correspond to a file path
- An accessor `(id) => module` which given one of the keys returns the module

_note: the metro implementation of require context doesn't include the `resolve` and `id` functionalities that webpack does._

For example

```ts
const req = require.context("components", true, /\.stories\.tsx$/);

req.keys().forEach((filename: string) => {
  const module = req(filename);

  // you could then access the exported fields
  // if I had a default export like
  // export default { title: "hello" }
  console.log(module.default.title);
});
```

For details on the implementation you can see Evan's PR to metro [here](https://github.com/facebook/metro/pull/822).

## Why?

The main driver for this feature seems to have been Expo router. The router is a file based routing library for react native and it will define your routes implicitly based on the file directory structure and the files you have. If you didn't have dynamic imports then to implement this you would have to generate a file of static exports/config that the router would read. Thats also the exactly what we've been doing in react native storybook 6.5 before require context was introduced.

Now that we have require context, the router uses a require.context call with a clever regex and the routes can all be figured out at runtime. This also comes with the benefit that fast refresh and other metro features will work since its built in.

This is really exciting for me because now I can remove lots of code and improve the developer experience of react native storybook.

I also imagine that this opens the door to other developer tools that can take advantage of dynamic imports in other ways.

## Mini Storybook reimplementation

To further illustrate my point I'll show how easily you can now implement a simplified version of storybook in react native. Note that I simplified the code snippet slightly by removing imports etc (full repo linked at the end).

For context a story written using the CSF syntax looks like this:

```ts
import MyComponent from "./MyComponent"

export default {
  title: "MyCoolComponent"
  component: MyComponent,
}

export const MyStory = {
  args: {
    text: "hello"
  }
}
```

We'll be dynamically importing any `.stories.tsx` file in our `components` folder and rendering the component with the props listed in its `args`.

Heres the code, make sure to look at the comments I added so you can follow whats going on.

```tsx
// get all story files
const req = require.context("components", true, /\.stories\.tsx$/);

// we create a map with id: story-title and value: module
let storiesMap = new Map<StoryTitle, ModuleExports>();

// for every module found lets add it to the storiesMap
req.keys().forEach((filename: string) => {
  try {
    // fileExports contains all exports, including default and named
    const fileExports = req(filename) as ModuleExports;

    // now we set an entry in the map like 'title' => module
    storiesMap.set(fileExports.default.title, fileExports);
  } catch (e) {}
});

let firstModule = "";

// get everything from our map as an array so we can access the first element
const entries = Array.from(storiesMap.entries());

if (entries.length) {
  const [title, exports] = entries[0];

  // each non 'default' export is a story
  const namedExports = Object.keys(exports).filter((key) => key !== "default");

  // we'll use title-storyname as an id
  firstModule = `${title}-${namedExports[0]}`;
}

const Content = () => {
  // we track the current componentId with this
  const [currentComponent, setCurrentComponent] = useState<string>(firstModule);

  const { Component, props } = useMemo(() => {
    if (currentComponent) {
      // we split our title-storyname id to get the key we need to access the map
      const [title, exportName] = currentComponent.split("-");

      // this gives us all the exports
      const exports = storiesMap.get(title);

      // get this current export (the story)
      const story = exports?.[exportName];

      return {
        // in csf the default export has a component field
        Component: exports?.default?.component,

        // a story's args represent our props
        props: story?.args,
      };
    }

    return {
      Component: null,
      props: {},
    };
  }, [currentComponent]);

  // we use gorhoms bottom sheet to display a list of stories
  const storiesBottomSheetModalRef = useRef<BottomSheetModal>(null);

  return (
    <SafeAreaView style={{ flex: 1, padding: 16 }}>
      {/* Render the current story with props from the args */}
      {Component && <Component {...props} />}

      {/* A button to open our story list */}
      <Button
        text="Stories"
        onPress={() => storiesBottomSheetModalRef.current?.present()}
        icon={icons.stories}
        style={{ flex: 1 }}
      />

      {/* A bottom sheet holding the list of stories */}
      <BottomSheet stackBehavior="replace" ref={storiesBottomSheetModalRef}>
        <StoryList
          setStory={setCurrentComponent}
          stories={Array.from(storiesMap.entries())}
          selectedStory={currentComponent}
        />
      </BottomSheet>
    </SafeAreaView>
  );
};
```

If you're curious then heres a repo with the full code https://github.com/dannyhw/sb-ui-experiment

Also heres the pr where I'm adding support for require.context to React Native Storybook https://github.com/storybookjs/react-native/pull/501

So what do you think? What could you build now that dynamic imports are possible?

Thanks for reading!

Get in touch with me at: [@Danny_H_W](https://twitter.com/Danny_H_W)
Check out my work on github: https://github.com/dannyhw
