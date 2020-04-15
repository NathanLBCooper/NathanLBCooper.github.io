---
layout: post
title: "How to validate min and max in your Angular form"
excerpt: "So the `min` and `max` constraints on number field in your template based form aren't working? Here's the code to fix that"
date: 2017-09-26 12:00:00
categories: [how-to, front-end, programming, angular]
comments: false
image:
  feature: angular.png
---

So the `min` and `max` constraints on number field in your template based form aren't working? Users can type in out of range values and the form is still valid. What's happening?

Let's say you've got a form, named `myform`, with this input:

```html
<input type="number" [(ngModel)]="model.count" name="count" #count="ngModel" required min="0">
```

With Angular's `FormsModule`, you get the validation on the `required` attribute basically for free. With no further code, if the user doesn't fill in this input then `count.hasError('required')` is true and  `myform.valid` is false. It's the same with other attributes like `pattern`, `maxLength` etc.

However, you don't get this with `min` or `max`.

This actually depends on your version of Angular. Angular 4 briefly had it, but it was [removed](https://github.com/angular/angular/pull/17622) in [version 4.2.3](https://github.com/angular/angular/blob/e17128e7cb44729c0df7d9cfcaf3dc7d92466813/CHANGELOG.md#423-2017-06-16)). Apparently was a breaking change for some, but not for me, so let's get it back.

The `angular/forms` package still has all the validation code we need, all we have to do is create our own directive. Here it is:

```javascript
import { Directive, Input, forwardRef } from "@angular/core";
import {
	Validator, AbstractControl, NG_VALIDATORS, Validators, ValidatorFn
	} from "@angular/forms";

@Directive({
    selector: "[min][formControlName],[min][formControl],[min][ngModel]",
    providers: [
        { provide: NG_VALIDATORS,
			useExisting: forwardRef(() => MinDirective),
			multi: true }
    ]
})
export class MinDirective implements Validator {
    private _validator: ValidatorFn;
    @Input() public set min(value: string) {
        this._validator = Validators.min(parseInt(value, 10));
    }

    public validate(control: AbstractControl): { [key: string]: any } {
        return this._validator(control);
    }
}
```

Create another one for `max`, add it into the declarations in your module, and you're all set.
