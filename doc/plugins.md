# Case Tracker Javascript Plugins

The case tracker adds a few plugins to the AXEL library, most of them can be directly instantiated with a custom plugin Tag in Supergrid.

## The `constant` plugin

> In: lib/extensions.js
>
> Implements: Constant (in Plugins) in supergid.xsl

The `constant` plugin is a readonly field to display different types of data

The plugin makes a special interpretation of two attributes which can be used in its XML input content model :

* the `_Display` attribute replaces the value loaded into the field by its value instead of the XML input;
* the `_Output` attribute replaces the data value serialized to XML output by its value.

So for instance loading `<Country _Display="Italy">IT</Country>` will result in loading *Italy* instead of *IT*. This is interesting if you use the *misc:unreference* function to generate the XML document representation to load into the editors, this way a single representation can be used for read-only presentation and for editing.

Reciprocally serializing `<TargetedMarkets _Output="502010">Renewable Energy</TargetedMarkets>` would result in `<TargetedMarkets>502010</TargetedMarkets>`. 

Microformat attributes :

* `handle` overwrites the generated HTML handle element, this only works when case *constant_media* is the default one (or when it is undefined)

Plugin parameters : 

* `constant_media` defines how to render the data
   * *image* renders it inside an HTML *img* element
   * *url, email, file* renders it inside an HTML *a* element
   * *text* (default) renders it as a *span* element
* `noxml=true` prevents the field serialization to XML output
* `xValue` forces serialization of its data into a corresponding Tag (or list of tags if the data is a whitespace separated list of tokens)
* `_Output` forces a constant serialization of its value as the field value whatever data it loaded, note that this is overloaded by the *_Output* attribute of the XML input if any

## The `attachment` plugin

> In: lib/extensions.js
>
> Implements: no XML vocabulary

The `attachment` plugin generate an HTML div element where it directly copy the children of its XML input content model.

This can be used to display multi-lines text messages formatted with line feeds and carriage returns and serialized in a `<pre>` content model : the *pre* element is directly copied to the *div*.

That plugin is not intended for editing formular since it serializes its XML output as a constant "HTML BLOB" string.

Example :

XML input model to display a pre-formatted generated e-mail attachment 

```xml
<Attachment>
  <pre>{ media:message-to-plain-text(local:gen-coaching-plan-for($case, $activity)) }</pre>
</Attachment>
```
