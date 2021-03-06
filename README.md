# Angular Standards

This should be considered an addition to [Google's Angular standards](https://angular.io/guide/styleguide), which are fairly broad and not super specific.

Whereas Google covers the _essentials_, the following should be viewed as _recommendations_, and also a starting place for new Angular developers. They are based on experiences building a variety of Angular apps and lessons learned in the process.

# CSS/Styles

See also: [Nerdery HTML/CSS Standards](https://github.com/thenerdery/html-css-standards/blob/master/standards/css.md)

Angular uses SCSS by default.

## Directory structure

Path | Description
--|--
`src/assets/styles/` | Global styles
`src/assets/styles/index.scss` | Global styles entry point. Where you should import third-party libraries and resets, define global element styles (e.g. `:root`, `<a>`, `<input>`, etc.), and define global utility classes (e.g. `.button-row`, `.modal`, `.tooltip`, etc.)
`src/assets/styles/tools/` | Where you should define variables, functions, and mixins. **Should not contain any actual style rules** -- if you import something from this directory, it shouldn't add anything to your compiled CSS.
`src/assets/styles/tools/index.scss` | Should `@forward` all the other files in the `tools` directory so that you only need to `@use 'src/assets/styles/tools'`, instead of individual files.
`src/assets/styles/tools/colors.scss`<br>`src/assets/styles/tools/typography.scss`<br>`src/assets/styles/tools/z-indexes.scss`<br>`src/assets/styles/tools/...` | Example tools.
`src/assets/styles/lib/` | For third-party libraries

## Use `@use`; don't use `@import`

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

## Use `@forward` to combine utilities

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

## Name variables with the same methodology as your CSS classes

See: https://github.com/thenerdery/html-css-standards/blob/master/standards/css.md#name-delimiters

```scss
$color-brand: #ff9999;
$color-brand_highlight: #ff0000;
$color-brand_lowlight: #cc9999;
```

## Mixins are preferable to utility classes

This keeps your HTML cleaner, separates concerns, and when looking at individual component stylesheets it's easier to understand how they relate to the whole.

## Use the same names for your typographies that the designers are using

Ideally, when inspecting an element in Sketch you should only need to look at its "Text Style" name to know exactly what CSS class/mixin to use, and vice-versa when inspecting an element in your browser you should see exactly which Sketch Text Style it ues.

Although you could do this simply through a class name (e.g. `.headline-1`, `.subtitle-2`, `.caption`) this has limitations. You probably want use the `Subtitle` Text Style for all `h3`, and it isn't practical to add the `.subtitle` class to all `<h3>` elements, so it won't be clear to which Text Style `h3` corresponds. You could `@extend` the `.subtitle` class for `h3`, but the name `subtitle` won't carry over.

My favored approach uses a big map of typographies, and a `@mixin` to retrieve the appopriate one, which adds a CSS variable (e.g. `--styleguide`) the contents of which is the Text Style name:

```scss
// src/assets/tools/typography.scss
$typography-configs: (
  'Headline 1': (
    font-family: $font-family-hdg,
    font-size: 32,
    font-weight: 400,
    letter-spacing: 0,
    line-height: 40,
  ),
  'Subtitle': (
    font-family: $font-family-hdg,
    font-size: 24,
    font-weight: 400,
    letter-spacing: 0,
    line-height: 32,
  ),
  'Caption': (
    color: color.$neutral-x2,
    font-family: $font-family-body,
    font-size: 12,
    font-weight: 400,
    letter-spacing: .3px,
    line-height: 16,
    -webkit-font-smoothing: auto,
  ),
);

@mixin typography($name) {
  $config: map-get($typography-configs, $name);
  $font-size: map-get($configm 'font-size');
  $formatted-config: map-merge($config, (
    line-height: (map-get($config, 'line-height') / $font-size), // Converts line-height to a fraction of font-size
    --styleguide: $name,
  ));
  @return $formatted-config;
}

// src/app/some-other-stylesheet.scss
@use 'src/assets/tools';

h3 {
    @include tools.typography('Subtitle');
}

.something-that-looks-like-h3 {
    @include tools.typography('Subtitle');
}
```

```css
/* What I see in the browser's element inspector */
h3 {
    font-family: 'Times New Roman';
    font-size: 24px;
    font-weight: 400;
    letter-spacing: 0;
    line-height: 1.33;
    --styleguide: 'Subtitle';
}

.something-that-looks-like-h3 {
    font-family: 'Times New Roman';
    font-size: 24px;
    font-weight: 400;
    letter-spacing: 0;
    line-height: 1.33;
    --styleguide: 'Subtitle';
}
```

# JS/TS

## Import `*` when possible

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

## Use `index.ts` when possible

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

## Import order

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

# Angular components/modules

## Organize your components into pages

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

## Group related components together

If Component X is only going to be used as a sub-component within Component Y, make it a subdirectory within Component Y.

(For small applications, this does conflict with the 'F' for 'Flat' in [LIFT](https://angular.io/guide/styleguide#lift). It is intended for applications that will grow to be large.)

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

## Create a module for globally-used components

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

## If using Angular Material, create an interface module

Odds are you're going to use the same several components through out your app. A local module just for importing Material components will keep you from needing to add lots of imports to the modules throughout your app.

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

# State management (Ngrx)

## Use StoreDevtoolsModule

This lets you use Redux Dev Tools, which is essential for debugging.

https://ngrx.io/guide/store-devtools

## Organize your files by feature, not by type

Good:

```
src/app/store
    index.ts
    auth/
        auth.actions.ts
        auth.effects.ts
        auth.reducers.ts
        auth.selectors.ts
        auth.types.d.ts
        index.ts
    users/
        users.actions.ts
        users.effects.ts
        users.reducers.ts
        users.selectors.ts
        users.types.d.ts
        index.ts
```

Bad:

```
src/app/store
    actions/
        auth.actions.ts
        users.actions.ts
    effects/
        auth.effects.ts
        users.effects.ts
    ...
```

## Use `.d.ts` files to define models/interfaces

**Note:** This is not compatible with Typescript <4.

The `.d.ts` extension signals to Typescript that these files are for type declarations only -- code statements in them will cause an error.

```
src/app/store
    index.ts
    auth/
        auth.actions.ts
        auth.effects.ts
        auth.reducers.ts
        auth.selectors.ts
        auth.types.d.ts
        index.ts
```

```ts
// src/app/app/store/auth/auth.types.d.ts
export interface SigninRequest {
    username: string;
    password: string;
}

// src/app/my-component/my-component.component.ts
import * as Auth from 'src/app/store/auth';

Auth.Type.SigninRequest...
```

## Define an `index.ts` for each store feature

If you need to import the actions for a store feature, odds are you'll need to import a lot of the other files too.

```ts
// src/app/store/auth/index.ts

import { AuthEffects as Effects } from './auth.effects';
import * as Actions from './auth.actions';
import * as Selectors from './auth.selectors';
import * as Store from './auth.reducers';
import * as Types from './auth.types.d';

export { Actions, Effects, Selectors, Store, Types };
```

```ts
// src/app/my-component/my-component.component.ts

import * as Auth from 'src/app/store/auth';

Auth.Actions...
Auth.Effects...
...
```

## Use `.createAction` to define actions

https://ngrx.io/api/store/createAction

This creates _much_ less boilerplate than the old method of using classes and tracking action names in big enums.

Note the use of the `prefix` variable below to properly and simply namespace actions.

```ts
// src/app/store/user/user.actions.ts

import { createAction, props } from '@ngrx/store';
import * as Model from './user.models';
import * as API from 'src/app/services/api/user';

const prefix = `[User]`;

export const getUser = createAction(
    `${prefix} Get User`,
    props<Model.User>,
);

export const getAllUsers = createAction(`${prefix} Get All Users`);

export const getAllUsersSuccess = createAction(
    `${prefix} Get All Users Success`,
    props<API.Models.NormalizedUserResponse>(),
);

export const getAllUsersFail = createAction(
    `${prefix} Get All Users Fail`,
    props<{
        status?: number;
        message: string;
    }>,
);
```

## Create reducers for mapping API data to local POJOs

There are a couple dos/donts in here:

### API data should be mapped to local models with their own schema

This separates concerns in case the API's schema changes, and provides a central place for transforming/manipulating the data as necessary.

### API data should be mapped when it goes _into_ the store

If you map the data when it comes _out of_ the store, e.g. in a selector, the logic will be executed run *every time the data is accessed* -- not very efficient!

### API data should be mapped to POJOs, not class instances

Class instances cannot be directly stored in Redux; they have to be serialized to POJOs, which in addition to not being very efficient also strips them of any instance methods.

Good:

```ts
// src/app/store/posts/posts.reducers.ts
import * as Actions from './posts.actions';
import * as Type from './posts.types.d';

export function _reducer = createReducer(
    initialState,
    on(Actions.getPostsSuccess, (state, apiPosts) => ({
        ...state,
        posts: apiPosts.map(postFromAPI),
    })),
)

export function reducers = (state: Model.State, action: Action) {
    return _reducer(state, action);
}

export function postFromAPI(
    apiPost: Type.APIPost,
): Type.Post {
    const _createdOn = new Date(apiPost.created_on);
    return {
        body: apiPost.body,
        createdOn: apiPost.created_on,
        createdOnDate: format(_createdOn, 'MMM d, yyyy'),
        createdOnTime> format(_createdOn, 'H:mm:ss'),
    }
}
```

Less good:

```ts
// src/app/store/posts/posts.models.ts
import * as Type from './posts.types.d';
export class Post {
  body: string;
  createdOn: string;
  createdOnDate: string;
  createdOnTime: string;

  constructor(apiPost: Type.APIPost) {
    const _createdOn = new Date(apiPost.created_on);
    this.body = apiPost.body;
    this.createdOn = apiPost.created_on;
    this.createdOnDate = format(_createdOn, 'MMM d, yyyy');
    this.createdOnTime = format(_createdOn, 'H:mm:ss');
  }
}

// src/app/store/posts/posts.reducers.ts
import * as Actions from './posts.actions';
import * as Model from './posts.models';
import * as Type from './posts.types.d';

export function _reducer = createReducer(
    initialState,
    on(Actions.getPostsSuccess, (state, apiPosts) => ({
        ...state,
        posts: apiPosts.map(apiPost => new Model.Post(apiPost)),
    })),
)

export function reducers = (state: Type.State, action: Action) {
    return _reducer(state, action);
}
```

Less good:

```ts
// src/app/store/posts/posts.models.ts
export class Post {
  body: string;
  createdOn: string;
  createdOnDate: string;
  createdOnTime: string;

  constructor(apiPost: Model.APIPost) {
    const _createdOn = new Date(apiPost.created_on);
    this.body = apiPost.body;
    this.createdOn = apiPost.created_on;
    this.createdOnDate = format(_createdOn, 'MMM d, yyyy');
    this.createdOnTime = format(_createdOn, 'H:mm:ss');
  }
}

// src/app/store/posts/posts.reducers.ts
import * as Actions from './post/actions';
import * as Model from './post.models';

export function _reducer = createReducer(
    initialState,
    on(Actions.getPostsSuccess, (state, posts) => ({
        ...state,
        posts,
    })),
)

export function reducers = (state: Model.State, action: Action) {
    return _reducer(state, action);
}

// src/app/store/posts/posts.selectors.ts
import * as Model from './post.models';

export const selectPosts: MemoizedSelector<
    Type.State,
    Model.APIPost,
> = createSelector(state: Type.State) => ({
    posts: state.apiPosts.map(apiPost => new Model.Post(apPost)),
}));
```
