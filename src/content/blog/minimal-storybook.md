---
title: "Manual setup for a minimal Storybook"
description: "How to setup storybook manually with the minimal packages and configuration."
pubDate: "May 13 2024"
heroImage: "/sbminimal.png"
---

If you're having issues with the cli or you just want to know what you're installing then this is for you. I've put together what I consider the minimal setup. From there you can browse addons and docs to see how to enhance your setup.

Heres a video of me following the same steps:

<iframe class="mb-4" width="100%" height="480" src="https://www.youtube.com/embed/C4Qnl_zOP8Y?si=Vka4kMZqeBj9Iiyw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

I'll use bun here but replace with any package manager.

Also this example uses [`react` with `vite`](https://storybook.js.org/docs/get-started/react-vite). If you're using a different framework you can swap `@storybook/react` and `@storybook/react-vite` for the alternative.

## Packages

Heres the minimal packages you'll need to get started with a react storybook.

```
bun add @storybook/react @storybook/react-vite @storybook/addon-essentials storybook
```

## Scripts

Add to your scripts in `package.json`

```json
"storybook": "storybook dev -p 6006",
"build-storybook": "storybook build"
```

## Configuration files

Add a `.storybook` folder with a `main.ts` and `preview.ts`.

### main.ts

[Find the docs here](https://storybook.js.org/docs/configure)

```ts
// .storybook/main.ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  stories: ["../src/**/*.mdx", "../src/**/*.stories.@(js|jsx|mjs|ts|tsx)"],
  addons: ["@storybook/addon-essentials"],
  framework: {
    name: "@storybook/react-vite",
    options: {},
  },
  docs: {
    autodocs: "tag",
  },
};
export default config;
```

### preview.ts

[Start here to learn about the preview](https://storybook.js.org/docs/configure#configure-story-rendering)

```ts
// .storybook/preview.ts
import type { Preview } from "@storybook/react";

const preview: Preview = {
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
  },
};

export default preview;
```

## Example story

Lets add a component file

```tsx
// src/Button.tsx
export const Button = ({ text }: { text: string }) => {
  return <button>{text}</button>;
};
```

Then we create a stories file and import it there.

[Look here to learn about writing stories](https://storybook.js.org/docs/writing-stories)

```tsx
// src/Button.stories.tsx
import { Button } from "./Button";

export default {
  component: Button,
};

export const Primary = {
  args: {
    text: "Primary Button",
  },
};
```

## Run it!

Now run this to see your storybook in action.

```
bun run storybook
```

The final product should look like this:

![screenshot of the final storybook](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1xqhkiup56bzobaibd0x.png)

## All done!

Thats the basics, now I recommend you explore the docs to learn more about how to customize storybook for your needs. Especially look into [interactions](https://storybook.js.org/docs/essentials/interactions#play-function-for-interactions) they are really nice.

Find me here:
twitter: [@Danny_H_W](https://twitter.com/Danny_H_W)
github: [dannyhw](https://github.com/dannyhw)
