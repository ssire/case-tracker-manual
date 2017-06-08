# Case Tracker Javascript bindings

Bindings are javascript objects bound to an AXEL or AXEL-FORMS plugin with the `data-binding` microformat attribute. They usually listen to `axel-update` event on the bound field and apply some checkings and side effects based on the result.

## The `switch` binding

> In: lib/extensions.js

The `switch` binding hides or shows different fragments of an editor depending on content changes inside the editor. This is a rewrite of original AXEL-FORMS `condition` binding inspired from the SVG *Switch* element.

The current value of the binding's host element variable can be used to dynamically add or remove classes on a set of targets elements.

The target elements are any HTML element holding a `data-avoid-{name}` attribute where *name* is the variable name. 

The optional `data-disable-class` attribute on the binding host element defines a generic class name. This class name will added to the target elements when their `data-avoid-{name}` value is the same as the variable value. On the contrary it will be removed from the target elements when their `data-avoid-{name}` value is different from the variable value. For instance you can use a CSS rule to hide the elements that hold this class name. 

So basically you can read the data-avoid-{name}="foobar" attribute as give me the class in `data-disable-class` when the variable value is *foobar* (to hide myself).

In addition or instead of using the optional `data-disable-class` attribute you can individually set some `data-off-class` or some `data-on-class` attributes on the target elements. 

The `data-off-class` name will be added to the class name of the target element when their `data-avoid-{name}` attribute has the same value as the variable. It will be removed otherwise.

The `data-on-class` name will be added to the class name of the target element when their `data-avoid-{name}` attribute has not the same value as the variable. It will be removed otherwise.

In the example below the mention "The coach contract should not be signed" will appear when the signature field is empty (no date selected). The mention "The coach contract can be signed" will appear when the signature field is not empty (there is a date entered).

```html
<div class="control-group">
  <label class="control-label a-gap">Date of signature</label>
  <div class="controls" data-binding="switch" data-variable="signature">
      <xt:use types="constant" label="Date" param="class=uneditable-input span a-control"></xt:use>
  </div>
</div>
...
<p data-avoid-signature="" data-on-class="hide" class="hide text-hint" style="color:red!important">The coach contract should not be signed</p>
<p data-avoid-signature="" data-off-class="hide" class="hide text-hint">The coach contract can be signed</p>
```


## The `mandatory` binding

> In: lib/extensions.js
>
> Supergrid: Mandatory element of Hints section

The `mandatory` binding listen to its bound field updates and sets a specific class on its bound field when it has en empty text content. 

The `mandatory` binding is actually compatible with the `input` plugin, `text` plugin and with the `choice` plugin with a `select2` filter.

Attributes :

* `data-validation` when *off* validation will not block from saving
* `data-mandatory-invalid-class` name of the class to be set when the mandatory field is empty
* `data-mandatory-type` name of the bound field HTML handle for applying the specific class, this must be *input* for an `input` plugin, *ul* for a `select2` filter of a `choice` plugin and *p* for a `text` plugin

The `mandatory` binding should be associated with an hint on the field label to augment usability.

Note that the difference of this binding with setting the *required=true* property of the bound field plugin is that it gives a continuous feebback while editing and not just on saving. It can also be non blocking for saving the content with the *data-validation=off* attribute.

Example :

```xml
<div class="control-group">
  <label class="control-label a-gap3">The top management understands the value of the coaching.
    <span title="To proceed to the next step please fill in the mandatory fields highlighted in red" rel="tooltip" class="sg-mandatory">*</span>
  </label>
  <div class="controls" data-binding="mandatory" data-validation="off" data-mandatory-invalid-class="af-mandatory" data-mandatory-type="ul">
    <site:field force="true" Size="9" Key="likert-scale" Tag="RatingScaleRef" Filter="event">likert-scale[RatingScaleRef,9]</site:field>
  </div>
</div>
```

which works for instance with the following CSS rule to display a red border around the bound field HTML handle :

```css
.axel-core-editable.af-mandatory,
ul.af-mandatory:not(.readonly),
a.af-mandatory,
input.af-mandatory {	
  border-style: solid !important;
  border-color: #ff4444 !important;
  border-radius: 4px !important;
  border-width: 1px !important;
}
```
