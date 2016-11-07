# Case Tracker Javascript Filters

## The `autofill` filter

> In: lib/extensions.js

The `autofill` filter listen to a change of value in its editor caused by user interaction (and not by loading data), then submit the change to a web service and loads its response into a target inside the editor.

Optional attributes :

* `autofill_container` : CSS selector of the HTML element containing the editor containing the target field, when this parameter is defined the filter also react to 'axel-content-ready' (editor's load completion event), this is useful for implementing lightweight transclusion
* `autofill_root` / `autofill_target` : CSS selector(s) of the subtree to fill with data, if not defined filling starts at the first ancestor of the host element handle that matches autofill_target selector 

