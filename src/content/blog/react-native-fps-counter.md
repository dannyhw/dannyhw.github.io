---
title: "A simple FPS counter for React Native"
description: "How to create a simple fps counter for react native"
pubDate: "Nov 12 2021"
heroImage: "/tech-generic-2.jpg"
---

If you've ever wanted to optimize performance on a react native app you've probably used the built in frame rate monitor from the dev menu. You've also probably read in the documentation that performance in dev mode is much worse. This means it's hard to get a real picture of what your frame rate is for production apps.

To get around this I have been using a custom hook and component to get an idea of FPS in a local release build. From my experience it's fairly close to the same as the JS fps that you get from the dev tools.

The code here is largely based on an example found online by my colleague. I've just adjusted it for my needs. You can see the original [here](https://gist.github.com/ronanamsterdam/1460a81088e9cb17185d3f31d872fdde)

Arguably the fps counter itself could have some impact on performance, however for some simple comparisons it's useful since if there is an impact it will be the same in both cases. If you see any area for improvement please let me know!

```tsx
import { useEffect, useState } from "react";

type FrameData = {
  fps: number;
  lastStamp: number;
  framesCount: number;
  average: number;
  totalCount: number;
};

export type FPS = { average: FrameData["average"]; fps: FrameData["fps"] };

export function useFPSMetric(): FPS {
  const [frameState, setFrameState] = useState<FrameData>({
    fps: 0,
    lastStamp: Date.now(),
    framesCount: 0,
    average: 0,
    totalCount: 0,
  });

  useEffect(() => {
    // NOTE: timeout is here
    // because requestAnimationFrame is deferred
    // and to prevent setStates when unmounted
    let timeout: NodeJS.Timeout | null = null;

    requestAnimationFrame((): void => {
      timeout = setTimeout((): void => {
        const currentStamp = Date.now();
        const shouldSetState = currentStamp - frameState.lastStamp > 1000;

        const newFramesCount = frameState.framesCount + 1;
        // updates fps at most once per second
        if (shouldSetState) {
          const newValue = frameState.framesCount;
          const totalCount = frameState.totalCount + 1;
          // I use math.min here because values over 60 aren't really important
          // I calculate the mean fps incrementatally here instead of storing all the values
          const newMean = Math.min(
            frameState.average + (newValue - frameState.average) / totalCount,
            60,
          );
          setFrameState({
            fps: frameState.framesCount,
            lastStamp: currentStamp,
            framesCount: 0,
            average: newMean,
            totalCount,
          });
        } else {
          setFrameState({
            ...frameState,
            framesCount: newFramesCount,
          });
        }
      }, 0);
    });
    return () => {
      if (timeout) clearTimeout(timeout);
    };
  }, [frameState]);

  return { average: frameState.average, fps: frameState.fps };
}
```

I then put this in a simple component at the root of the project and make it toggle-able.

Here is an example of that

```tsx
import React, { FunctionComponent } from "react";
import { StyleSheet, Text, View } from "react-native";
import { useFPSMetric } from "./useFPSMetrics";

const styles = StyleSheet.create({
  text: { color: "white" },
  container: { position: "absolute", top: 100, left: 8 },
});

export const FpsCounter: FunctionComponent<{ visible: boolean }> = ({
  visible,
}) => {
  const { fps, average } = useFPSMetric();
  if (!visible) return null;
  return (
    <View pointerEvents={"none"} style={styles.container}>
      <Text style={styles.text}>{fps} FPS</Text>
      <Text style={styles.text}>{average.toFixed(2)} average FPS</Text>
    </View>
  );
};
```

Then in the app entry point

```tsx
export default (): ReactElement => (
  <View>
    <App />
    <FpsCounter visible={true} />
  </View>
);
```

It's important to have it at the root of the app otherwise some updates to the FPS might be delayed and the FPS won't be accurate.

Hopefully that's useful to someone out there and if you have a better way to measure JS FPS in release config then please share that so we can all improve together.

Thanks for taking the time to read my post, heres my [github](https://github.com/dannyhw) if you want to see my other work.
