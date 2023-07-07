# Original Code

```javascript
export function setGridAttributes() {
  const options = { ...this.model.get("grid"), id: this.model.id };

  if (!options.x || !options.y) {
    options.autoPosition = true;
  }

  Object.entries(options).forEach(([key, value]) => {
    this.$el.attr(`data-gs-${kebabCase(key)}`, value);
  });
}
```

# Solution

```javascript
export function setGridAttributes() {
  const options = { ...this.model.get("grid"), id: this.model.id };

  // Changed the line below
  if (options.x === undefined || options.y === undefined) {
    options.autoPosition = true;
  }
  Object.entries(options).forEach(([key, value]) => {
    this.$el.attr(`data-gs-${kebabCase(key)}`, value);
  });
}
```

## Explanation

In the original code, with the bug, the bang operator (!) will negate any [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value.
While this works as expected when the grid's `x` or `y` values are `undefined`, it will incorrectly also capture and run the auto position function when the `x` or `y` values equals the valid grid number `0`, as `0` is also considered falsy. To fix this, we specify the exact values to capture inside the if-condition. In this case we only want to capture the case of `undefined`.
