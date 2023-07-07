# Original Code

```javascript
import { clone } from "underscore";

function getBoundingBoxFor(selector) {
  let bBox;
  $(selector).each(function () {
    const eleBb = this.getBoundingClientRect();
    if (bBox === undefined) {
      bBox = clone(eleBb);
    }
    bBox.top = Math.min(bBox.top, eleBb.top);
    bBox.left = Math.min(bBox.left, eleBb.left);
    bBox.right = Math.max(bBox.right, eleBb.right);
    bBox.bottom = Math.max(bBox.bottom, eleBb.bottom);
  });

  bBox.width = bBox.right - bBox.left;
  bBox.height = bBox.bottom - bBox.top;

  return bBox;
}
```

# Original Code Refactored with the Bug

```javascript
function getBoundingBoxFor(selector) {
  let bBox;
  $(selector).each(function () {
    const eleBb = this.getBoundingClientRect();
    if (bBox === undefined) {
      bBox = { ...eleBb };
    }
    bBox.top = Math.min(bBox.top, eleBb.top);
    bBox.left = Math.min(bBox.left, eleBb.left);
    bBox.right = Math.max(bBox.right, eleBb.right);
    bBox.bottom = Math.max(bBox.bottom, eleBb.bottom);
  });

  bBox.width = bBox.right - bBox.left;
  bBox.height = bBox.bottom - bBox.top;

  return bBox;
}
```

# Solution #1 - Always Return `DOMRect`

```typescript
type BoundingBox = Pick<DOMRectReadOnly, `top` | `left` | `bottom` | `right`>;

function createInitialBoundingBox(): BoundingBox {
  return {
    top: 0,
    left: 0,
    bottom: 0,
    right: 0,
  };
}

function getElementsBySelector(selector: string): Element[] {
  if (!selector.length) {
    return [];
  }
  return [...document.querySelectorAll(selector)];
}

function expandBoundingBoxByElement(
  initialBoundingBox: BoundingBox,
  element: Element
): BoundingBox {
  const boundingBoxToExpandWith = element.getBoundingClientRect();
  return {
    top: Math.min(boundingBoxToExpandWith.top, initialBoundingBox.top),
    left: Math.min(boundingBoxToExpandWith.left, initialBoundingBox.left),
    bottom: Math.max(boundingBoxToExpandWith.bottom, initialBoundingBox.bottom),
    right: Math.max(boundingBoxToExpandWith.right, initialBoundingBox.right),
  };
}

function createDOMRectByBoundingBox(boundingBox: BoundingBox) {
  const width = boundingBox.right - boundingBox.left;
  const height = boundingBox.bottom - boundingBox.top;
  return new DOMRect(boundingBox.left, boundingBox.top, width, height);
}

function getBoundingBoxFor(selector: string): DOMRect {
  const initialBoundingBox = createInitialBoundingBox();
  const selectorElements = getElementsBySelector(selector);
  const expandedBoundingBox = selectorElements.reduce(
    expandBoundingBoxByElement,
    initialBoundingBox
  );
  const domRectBoundingBox = createDOMRectByBoundingBox(expandedBoundingBox);
  return domRectBoundingBox;
}
```

## Interactive Example

[CodePen](https://codepen.io/henriq88/pen/mdQMmZz?editors=0011)

## Explanation

The returned value of `getBoundingClientRect` is of the class type `DOMRect`. And in JavaScript classes are "special objects". The properties of `DOMRect` are actually on the prototype of the object. Actually, to list all of the properties you have to get the prototype (`DOMRectReadOnly`) of the prototype (`DOMRect`). When spreading the object you create a shallow copy, however, [the prototype is excluded](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals), therefore no of the properties are copied over to the new object.

If one wants to copy the prototype properties, the easiest option, and maybe the clearest, is to directly call each properties and store it in a custom object.

## Additional Improvements

### Add Types by Converting to TypeScript

Types help to exterminate bugs that could easily have been found with a linter.

Types can also help with understanding of how native browser functionality can be used, by going into the reference type files.

### Avoid Mutability When Possible

Immutable code tends to be more reliable and easier to debug as the same input always lead to the corresponding output, e.g. f(x) = y. Although it can also lead to more memory usage, as objects and arrays tend to be recreted multiple times.

In the given solution, apart from the spreading of the `selectorElements` array, there shouldn't be any noticable performance loss.

### Handling of Selectors Not Found

Perhaps this was out of scope, as both the original code and the refactored code both result in an error when providing a selector that can't be found.

```
Uncaught TypeError: Cannot read properties of undefined (reading 'right')
```

Regardless, an initial bounding box value has been defined to handle the case where no elements where found.

### Split Up Code In Smaller Logical Parts

The initializing of the bounding box, selection of elements, expanding of the bounderies of a given bounding box and constructing of the final bounding box area has been split up into smaller reusable parts. Although quite verbose, it should be easy to change, replace and test parts of the code, without knowing the rest of the code, due to the types each function has been provided with.

## Remove Unncessary Libraries

Continuing the quest to drop unncessary library functions, `jquery`'s `$(selector)` function has been replaced with a browser native alternative [document.querySelectorAll(selector)](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll).

# Solution #2 - Can Also Return `undefined`

```typescript
type BoundingBox = Pick<DOMRectReadOnly, `top` | `left` | `bottom` | `right`>;

function getElementsBySelector(selector: string): Element[] {
  if (!selector.length) {
    return [];
  }
  return [...document.querySelectorAll(selector)];
}

function expandBoundingBoxByElement(
  initialBoundingBox: BoundingBox | undefined,
  element: Element
): BoundingBox {
  const boundingBoxToExpandWith = element.getBoundingClientRect();
  return {
    top: Math.min(boundingBoxToExpandWith.top, initialBoundingBox?.top ?? 0),
    left: Math.min(boundingBoxToExpandWith.left, initialBoundingBox?.left ?? 0),
    bottom: Math.max(
      boundingBoxToExpandWith.bottom,
      initialBoundingBox?.bottom ?? 0
    ),
    right: Math.max(
      boundingBoxToExpandWith.right,
      initialBoundingBox?.right ?? 0
    ),
  };
}

function createDOMRectByBoundingBox(boundingBox: BoundingBox) {
  const width = boundingBox.right - boundingBox.left;
  const height = boundingBox.bottom - boundingBox.top;
  return new DOMRect(boundingBox.left, boundingBox.top, width, height);
}

function getBoundingBoxFor(selector: string): DOMRect {
  const selectorElements = getElementsBySelector(selector);
  if (!selectorElements.length) {
    return;
  }
  const expandedBoundingBox = selectorElements.reduce(
    expandBoundingBoxByElement,
    undefined
  );
  const domRectBoundingBox = createDOMRectByBoundingBox(expandedBoundingBox);
  return domRectBoundingBox;
}
```

## Interactive Example

[CodePen](https://codepen.io/henriq88/pen/BaGdRXE?editors=0011)

## Explanation

If it was intended that the `getBoundingBoxFor` function could accept a selector that couldn't be found in the DOM, the `createInitialBoundingBox` function has been removed, `initialBoundingBox` has been replaced with `undefined`, and fallback values have been added to the `expandBoundingBoxByElement` function.

With the above mentioned changes, if no elements are found with the given `selector`, the function `getBoundingBoxFor` returns `undefined`.

# Solution #3 - Minimal Change That Uses Native Function

```javascript
import $ from "https://cdn.skypack.dev/jquery@3.5.1";

function getBoundingBoxFor(selector) {
  let bBox;

  $(selector).each(function () {
    const eleBb = this.getBoundingClientRect();

    if (bBox === undefined) {
      bBox = { ...eleBb.toJSON() };
    }

    bBox.top = Math.min(bBox.top, eleBb.top);
    bBox.left = Math.min(bBox.left, eleBb.left);
    bBox.right = Math.max(bBox.right, eleBb.right);
    bBox.bottom = Math.max(bBox.bottom, eleBb.bottom);
  });

  bBox.width = bBox.right - bBox.left;
  bBox.height = bBox.bottom - bBox.top;

  return bBox;
}
```

## Interactive Example

[CodePen](https://codepen.io/henriq88/pen/oNQeVxq?editors=0011)

## Explanation

As mentioned before, an instance of `DOMRect` is a special object. One of the properties of this object is a function called `toJSON` that creates a new normal object with the values of the `DOMRect`. This object can be used as expected.
