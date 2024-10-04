# Framer Motion

![framer_motion_logo](../assets/framer_motion_logo.png)

## Table of Contents

- [What is Framer Motion?](#what-is-framer-motion)
- [Key Animation Features](#key-animation-features)
  - [Layout](#layout)
  - [Gestures](#gestures)
  - [Scroll](#scroll)
  - [Transition](#transition)
- [What is the Motion Component?](#what-is-the-motion-component)
- [Conclusion](#conclusion)

##  What is Framer Motion?

Framer Motion is a powerful animation library for React, designed to simplify the process of adding animations to React applications. It provides a range of features, including declarative syntax, animation controls, and support for both simple and complex animations. The library offers several elements enhanced with "animation values" and motion-hooks, allowing for smooth animations of single elements to entire sub-trees of components with minimal additional code.

##  Key Animation Features

###  Layout

Framer Motion excels in layout animations, which are typically costly due to the layout system being triggered on every animation frame, potentially leading to poor framerates. To address this, Framer Motion uses transforms in animations for smooth framerates, instead of relying on the layout system. This is achieved by attaching the layout prop to the motion component, enabling shared layout animations for components, such as creating a moving underline component between tabs.

```html
<motion.div layout />
```

###  Gestures

The library supports a variety of gestures, including hover, focus, tap, pan, and drag. These gestures extend the base event listener from React, and can be integrated into a component by attaching a gesture prop to a motion component.

```html
<motion.div whileHover={{ scale: 1.2 }} />
```

### Scroll

Framer Motion offers two types of scroll animations: scroll-linked animations, where the animation progress is directly coupled to the scroll progress, and scroll-triggered animations, which trigger an animation when an element enters or leaves the viewport.

```JS
import { motion, useScroll } from "framer-motion"

function Component() {
  const { scrollYProgress } = useScroll()

  return <motion.div style={{ scaleX: scrollYProgress }} />
}
```

###  Transition

Framer Motion includes a transition animation type, which can animate an element between two states.

```html
<motion.div animate={{ x: 100 }} transition={{ delay: 1 }} />
```

##  What is the Motion Component?

The motion component is a central feature of the Framer Motion library. It is the foundation for all animations, with the library providing a motion component for every HTML and SVG element, such as motion.div, motion.circle, etc. These components work similarly to their static counterparts but offer additional animation props like animate, layout, etc. Any React component can be turned into a motion component by wrapping it with the motion() function and forwarding the component's ref.

```JS
const ClickableButton = ({ children, onClick }) => {
 return (
    <button
      onClick={onClick}
      style={{ padding: "10px", backgroundColor: "black", color: "white" }}
    >
      {children}
    </button>
 )
}

export ClickableButton

// Wrap component into motion function to create a motion Button, which then can be animated with motion props
const MotionClickableButton = motion(ClickableButton);

export MotionClickableButton;
```

## Conclusion

Framer Motion makes it remarkably easy to create animations in React. It requires no in-depth CSS animation knowledge, allowing developers to create detailed animations with just a few additional elements and props. Its intuitive approach and the wide range of features make it a popular choice for implementing animations in React applications.

Citation:

- <https://www.framer.com/motion/>
- <https://www.nan.fyi/magic-motion>
