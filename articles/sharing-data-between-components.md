# Sharing Data Between Components

How do you get your components to talk to each other? That is one of the first questions you may run into when getting started with Aurelia.

This could be parent/child components or sibling components.

Most often the answer is simply "Dependency Injection" (or DI as you might see it referred to). But what exactly is dependency injection?

## Dependency Injection Demystified

Basically, whenever we have data we want shared between components, the answer is to create a separate class and inject this class into each component view-model where we need it.

It is best explained with a simple example. Check out this [gist.run](https://gist.run/?id=060f4d1db4bf7679af00296c0fdd8e5b), then we go through it.

<iframe src="https://gist.run/embed.html?id=060f4d1db4bf7679af00296c0fdd8e5b" width="100%" height="400px"></iframe>

#### app.html:

```html
<template>
  <require from="./comp1"></require>
  <require from="./comp2"></require>
  
  <comp1></comp1>
  <comp2></comp2>
</template>
```

We're basically including to components, `comp1` and `comp2` and displaying them on the page. Nothing fancy here.

#### shared-service.js

```js
export class Shared {
  constructor() {
    this.val = 'Initial value.';
  }
}
```

This is basically our shared class (or service) that we want to inject into each of our components. You could put all kinds of variables and objects in here to share, but basically we are just going to use `val` in each of our components.

#### comp1.js

```js
import {inject} from 'aurelia-framework';

import {Shared} from 'shared-service';

@inject(Shared)
export class Comp1 {
  constructor(shared) {
    this.shared = shared;
  }
  
  setVal() {
    this.shared.val = 'Component 1 set value';
  }
}
```

Here is our first component view-model. Notice we are importing `inject` from `aurelia-framework` and our `Shared` class from `shared-service.js`.

We can then use the `@inject` decorator, which is an ES2016 feature, to inject this class into our view-model class. Notice we need to set it to a class variable in our constructor. If this pattern looks new to you, don't worry, you'll get used to it fast in Aurelia, because you'll use it everywhere.

If you prefer not to use the decorator for whatever reason, check out this [stack overflow example on how to inject with ES5](http://stackoverflow.com/questions/39122600/how-to-inject-dependencies-in-aurelia-without-using-es6-class-feature).

#### comp1.html

```html
<template>
  <require from="./comp1child"></require>
  
  <div>
    Comp1: ${shared.val}<br>
    <button click.delegate="setVal()">Set Value</button>
  </div><br>
  
  <comp1child></comp1child>
</template>
```

Here you can basically see how we're including a child component and outputting the `shared.val` value.

If you look through the other components you will notice they pretty much follow the same pattern so there is not much else to explain. Basically our shared class is a singleton that is injected to our component classes.

*Bonus:* One small thing to point out is the `comp2child` component. Notice there is no view model. All we have is `comp2child.html` with no associated javascript file.

#### comp2child.html

```html
<template bindable="val">
  <div>
    Comp2 Child component with no view-model.<br>
    Edit here too: <input type="text" value.bind="val">
  </div>
</template>
```

We are doing this with the `bindable` attribute in the `<template>` tag.

We then pass in the attribute we want to bind when we add the component: `<comp2child val.two-way="shared.val"></comp2child>`. Notice we're specifying `two-way` data binding. If we just used `.bind` it would default to `one-way` binding and editing the value in the comp2child component would not propagate back up to the service.

If we were to have a view-model for `comp2child` what would it look like if we wanted to have a bindable instead of using DI? Something like this:

```js
import {bindable} from 'aurelia-framework';

export class Comp2child {
  @bindable val
}
```

