---
title: "Test IDs and React Testing Library"
date: 2021-03-27T17:59:12-04:00
draft: false
toc: false
images:
tags: 
  - react
  - testing
  - javascript
---

The React Testing Library is pretty opinionated about how you find DOM nodes in order to make assertions. They want you to stick with presentational content like text whenever possible. That makes sense, but there are cases where you'll want to use test IDs. When doing test-driven development, for instance, you'll want to make assertions on the DOM structure as you build out your feature, before there's any user-visible content.

You may think, hey, why can't I just assert on classnames and styles? Those exist while I'm building the feature and before I have user-visible content. You can, but they often get refactored, which makes your tests brittle. Test IDs are better. Here's how to use them.

By default the test ID attribute will be `data-testid`. You can configure it to be something different using the `testIdAttribute`, see the [docs](https://testing-library.com/docs/dom-testing-library/api-configuration/).

Usage is straightforward:

```
<div data-testid="custom-element" />
```

```
import { render, screen } from '@testing-library/react'

render(<MyComponent />)
const element = screen.getByTestId('custom-element')
```

One thing you will want to do is remove these `data-testid` attributes from your production assets. They're only for testing, and they increase the size of the bundle. If your bundle is poorly optimized this isn't going to save you much; on the other hand, it's easy and every bit counts.

There's a [Babel plugin](https://github.com/oliviertassinari/babel-plugin-react-remove-properties) for this (of course).

```
{
  "env": {
    "production": {
      "plugins": [
        ["react-remove-properties", {"properties": ["data-testid"] }]
      ]
    }
  }
}
```

This is a pretty clean approach in my opinion and greatly assists test-driven development. It was a common pattern in Ember testing, and I'm glad I can do the same thing in React.
