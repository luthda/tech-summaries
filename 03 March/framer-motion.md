# Framer Motion

Framer Motion is an simple animation library for react.
![framer_motion_logo](../assets/framer_motion_logo.png)

## What is Framer Motion?

The React motion library of Framer provides several elements which can be enhanced with "animation values" and motion-hooks for the animation from single elements to entire sub-trees of components.
React components can be smoothly animated with little additonal code from the Motion library.

## What are the key animation features?

### Layout

Layout animations are usually costly because every animation frame triggers the layout system, which then results in a poor framerate. Motion layout uses transforms in animations for smooth framerates, instead of using the layout system. It is possible to create shared layout animations for components by attaching the layout prop to the motion component, for example t ocreate a moving underline component between tabs.

```html
<motion.div layout />
```

### Gestures

Motion supports hover, focus, tap, pan and drag gestures. This gestures extend the base event listener from React. Gestures can be integratet into a component by attaching a gesture prop to a motion component.

```html
<motion.div whileHover={{ scale: 1.2 }} />
```

### Scroll

Motion has two types of scroll animation. Firstly the scroll-linked animation where the animation progress is directly coupled to the scroll progress. Secondly scroll-triggered animations when a element enters or leaves the viewport by an animation.

```javascript
import { motion, useScroll } from "framer-motion"

function Component() {
  const { scrollYProgress } = useScroll()

  return <motion.div style={{ scaleX: scrollYProgress }} />
}
```

### Transition

Motion has an animation type transition which can animate an element between to states.

```html
<motion.div animate={{ x: 100 }} transition={{ delay: 1 }} />
```
