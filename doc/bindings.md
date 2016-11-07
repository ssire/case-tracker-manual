# Case Tracker Javascript bindings

Bindings are javascript objects bound to an AXEL or AXEL-FORMS plugin with the `data-binding` microformat attribute. They usually listen to `axel-update` event on the bound field and apply some checkings and side effects based on the result.

## The `switch` binding

> In: lib/extensions.js

The `switch` binding hides or shows different fragments of an editor depending on content changes inside the editor. This is a rewrite of original AXEL-FORMS `condition` binding inspired from the SVG *Switch* element.

TO BE COMPLETED

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
