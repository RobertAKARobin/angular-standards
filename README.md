# Angular Standards

## CSS/Styles

See also: [Nerdery HTML/CSS Standards](https://github.com/thenerdery/html-css-standards/blob/master/standards/css.md)

Angular uses SCSS by default.

### Directory structure

Path | Description
--|--
`src/assets/styles/` | Global styles
`src/assets/styles/index.scss` | Global styles entry point. Where you should import third-party libraries and resets, define global element styles (e.g. `:root`, `<a>`, `<input>`, etc.), and define global utility classes (e.g. `.button-row`, `.modal`, `.tooltip`, etc.)
`src/assets/styles/tools/` | Where you should define variables, functions, and mixins. **Should not contain any actual style rules** -- if you import something from this directory, it shouldn't add anything to your compiled CSS.
`src/assets/styles/tools/index.scss` | Should `@forward` all the other files in the `tools` directory so that you only need to `@use 'src/assets/styles/tools'`, instead of individual files.
`src/assets/styles/tools/colors.scss`<br>`src/assets/styles/tools/typography.scss`<br>`src/assets/styles/tools/z-indexes.scss`<br>`src/assets/styles/tools/...` | Example tools.
`src/assets/styles/lib/` | For third-party libraries

### Use `@use`; don't use `@import`

[`@import` has been deprecated.](https://sass-lang.com/documentation/at-rules/import)

https://sass-lang.com/documentation/at-rules/use

Whereas `@import`ed variables are all global, `@use` lets you namespace them. For example:

```scss
@use 'src/assets/styles/tools' as tools;

h3 {
  @include tools.typography('Headline 4');

  margin-bottom: tools.$margin__default;
}
```

### Use `@forward` to combine utilities

https://sass-lang.com/documentation/at-rules/forward

For example:

```scss
// src/assets/styles/tools/z-indexes.scss

$navbar: 11;
$content-overlay: 10;
```

```scss
// src/assets/styles/tools/index.scss

@forward './utils';
@forward './color' as color__*;
@forward './variables';
@forward './typography';
@forward './z-indexes.scss' as z__*;
```

```scss
// my-component.scss

@use 'src/assets/styles/tools' as tools;

.popup {
    z-index: tools.$content-overlay;
}
```

### Name variables with the same methodology as your CSS classes

See: https://github.com/thenerdery/html-css-standards/blob/master/standards/css.md#name-delimiters

For example:

```scss
$color-brand: #ff9999;
$color-brand_highlight: #ff0000;
$color-brand_lowlight: #cc9999;
```

### Use the same names for your typographies that the designers are using

Ideally, when inspecting an element in Sketch, you should only need to look at its "Text Style" name to know exactly what CSS class/mixin to use.

### Mixins are preferable to utility classes

This keeps your HTML cleaner, separates concerns, and when looking at individual component stylesheets it's easier to understand how they relate to the whole.
