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

Whereas `@import`ed variables are all global, `@use` lets you namespace them.

```scss
@use 'src/assets/styles/tools' as tools;

h3 {
  @include tools.typography('Headline 4');

  margin-bottom: tools.$margin__default;
}
```

### Use `@forward` to combine utilities

https://sass-lang.com/documentation/at-rules/forward

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

```scss
$color-brand: #ff9999;
$color-brand_highlight: #ff0000;
$color-brand_lowlight: #cc9999;
```

### Use the same names for your typographies that the designers are using

Ideally, when inspecting an element in Sketch, you should only need to look at its "Text Style" name to know exactly what CSS class/mixin to use.

### Mixins are preferable to utility classes

This keeps your HTML cleaner, separates concerns, and when looking at individual component stylesheets it's easier to understand how they relate to the whole.

## JS/TS

### Import `*` when possible

If you can assume that all the items in a file or directory are going to be loaded into memory at some point anyway, then import them with `*`. This removes clutter, and allows you to namespace things for added readability.

**As a rule** you can probably use `*` when importing code you wrote yourself, and probably can't when importing code from a third-party library. Presumably all the code you wrote is intended to be used, whereas libraries provide additional utilities/functionality you may not need.

Good:

```ts
import * as Filters from 'src/app/services/filters';
import * as API from 'src/app/services/api';

Filters.Models...
Filters.Service...
API.Models...
API.Service...
```

Bad:

```ts
import { Service, Models } from 'src/app/services/filters';
import { Service, Models } from 'src/app/services/api'; // Variable name conflict!
```

Good:

```ts
import { map, tap, filter } from 'rxjs/operators';
```

Bad (unfortunately):

```ts
import * as Rx from 'rxjs/operators';
```

### Use `index.ts` when possible

If you can assume that a group of files in a directory are always going to be imported together, then use `index.ts` as an entry point for the files. That way, you only need to load the directory instead of loading all its individual files.

```ts
src/app/services/api/api.models.ts
src/app/services/api/api.service.ts
src/app/services/api/index.ts
```

```ts
// src/app/services/filters/index.ts

import * as Models from './filters.models';
import { FiltersService as Service } from './filters.service';

export { Models, Service };
```

```ts
// src/app/mycomponent/mycomponent.component.ts

import * as Filters from 'src/app/services/filters';

Filters.Service...
Filters.Models...
```

### Import order

Import should go from least-specific to most-specific.

1. Third-party libraries
2. Local utility libraries
3. Services
4. Subcomponents

```ts
import {
  Component,
  Input,
} from '@angular/core';
import { Observable } from 'rxjs';

import * as $ from 'src/lib/util';

import { StateService } from 'src/app/myapp/state';

import { MySubComponent } from 'src/app/mycomponent/subcomponents/';
```

## Angular components/modules

### Organize your components into pages

If applicable, organize your components into pages, grouped together with a module for each page. This way, you don't need to import all your components into the application's root, and can more clearly define which dependencies are relevant to which pages.

```
src/app/pages/
    home/
        home.module.ts
        home.component.html
        home.component.scss
        home.component.ts
        news/
            news.component.html
            news.component.scss
            news.component.ts
    contact/
        contact.module.ts
        contact.component.html
        contact.component.scss
        contact.component.ts
```

### Group related components together

If Component X is only going to be used as a sub-component within Component Y, make it a subdirectory within Component Y.

```
src/app/pages/profile/
    profile.module.ts
    profile.component.html
    profile.component.scss
    profile.component.ts
    notifications/
        notifications.component.html
        notifications.component.scss
        notifications.component.ts
    password/
        password.component.html
        password.component.scss
        password.component.ts
    profile-pic/
        profile-pic.component.html
        profile-pic.component.scss
        profile-pic.component.ts
        picker/
            picker.directive.ts
src/app/pages/signin/
    signin.module.ts
    signin.component.html
    signin.component.scss
    signin.component.ts
```

### Create a module for globally-used components

```
arc/app/shared-components/
    shared-components.module.ts
    chart-bar/
        chart-bar.component.html
        chart-bar.component.scss
        chart-bar.component.ts
    chart-donut/
    checklist/
    material/
        material.module.ts
```

```ts
// src/app/shared-components/shared-components.module.ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { MaterialModule } from 'src/app/shared-components/material/material.module';

import { ChartBarComponent } from './chart-bar/chart-bar.component';
import { ChartDonutComponent } from './chart-donut/chart-donut.component';
import { ChecklistComponent } from './checklist/checklist.component';

const COMPONENTS = [
    ChartBarComponent,
    ChartDonutComponent,
    ChecklistComponent,
];
@NgModule({
    declarations: [...COMPONENTS],
    exports: [
        MaterialModule,
        ...COMPONENTS,
    ],
    imports: [
        CommonModule,
        MaterialModule,
    ],
})
export class SharedComponentsModule {}
```

### If using Angular Material, create an interface module

If you're using Angular Material, odds are you're going to use the same several components through out your app.

A local module just for importing Material components will keep you from needing to add lots of imports to the modules throughout your app.

```ts
// src/app/shared-components/material/material.module.ts

import { NgModule } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatCheckboxModule } from '@angular/material/checkbox';

const MODULES = [
  MatButtonModule,
  MatCardModule,
  MatCheckboxModule,
];

@NgModule({
  declarations: [],
  imports: [...MODULES],
  exports: [...MODULES],
})
export class MaterialModule {}
```

## State management
