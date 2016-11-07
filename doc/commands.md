# Case Tracker Javascript commands

Commands are Javascript objects bound to an HTML user interface element like a button or a drop-down list with a `data-command` microformat attribute. They can be directly invoked client-side by user interaction or indirectly as a side effect of an Ajax response generated server-side.

In **client-side invocation** the commmand is triggered by user clicking the command host element.

In **server-side invocation** the command is triggered by returning an Ajax response containing a _forward_ element as explained in the [Ajax protocols](./ajax.md) chapter.

Commands are implemented in different files :

* `resources/lib/extensions.js` contains generic commands
* `resources/lib/workflow.js` contains commands specific to the workflow accordion view, they are instantiated in `modules/workflow/workflow.xsl`

The *Implements* annotation of each command presentation below gives some indication about the XML vocabulary it implements. You can get more details in the different XML vocabulary chapters :

* [UI components XML vocabulary](./components.md) for vocabulary from workflow.xsl, widgets.xsl or commons.xsl
* [Workflow](./workflow.md) for vocabulary from application.xml in workflow.xsl
* [Supergrid specification language](./supergrid-spec.md) for vocabulary from supergrid.xsl

## The `acc-drawer` command

> In: lib/workflow.js
>
> Implements: Document (containing Actions/Drawer) in workflow.xsl

NOT USED

The `acc-drawer` command manages opening and closing of a drawer inside a document inside an Accordion widget. The opening is triggered by an action button generated from a `Drawer` action.

## The `augment` command

> In: lib/extensions.js

NOT USED

## The `autoexec` command

> In: lib/workflow.js
>
> Implements: AutoExec from application.xml in workflow.xsl

The `autoexec` command triggers a target command with an optional event target. This command is meant to be triggered server-side through a `forward` element in Ajax response. 

## The `confirm` command

> In: lib/workflow.js

The `confirm` command implements a subset of the `save` command protocol limited to a two-steps confirmation protocol to generate any server-side effect.

TO BE COMPLETED

## The `c-delannex` command

> In: lib/workflow.js

NOT USED

## The `c-delete` command

> In: lib/extensions.js
>
> Implements: Delete in workflow.xsl, widgets.xsl

The `c-delete` command implements a protocol to delete a resource. The resource can be directly specified with the `data-controller` attribute or indirectly as the resource currently loaded into an editor specified with the `data-target` attribute.

The protocol can implement a one step or a two steps transaction.

In the one step version the procotol is a simple POST request with a single _\_delete=1_ parameter to confirm the delete.

In the two steps version the first request omits the _\_delete=1_ parameter, then the server must answer with an intermediate response with a 202  HTTP status and a resource specific message to prompt the user for confirmation. If the user confirms it then sends the POST request with a single _delete=1_ parameter as in the one step version.

In any case the server may answer with an error and a message in case it is not possible to delete the resource.

Optional attributes :

* `data-target` contains the target editor id
* `data-controller` contains the URL for the resource to delete if *data-target* is not set 
* `data-confirm` contains a resource independent confirmation message to prompt the user for confirmation before executing the command, in that case the command sends directly a confirmed request (1 step protocol)

The command sends `axel-transaction` and `axel-transaction-complete` events directly on the command host.

## The `c-inhibit` command

> In: lib/extensions.js

The `c-inhibit` command shows a specific message while executing another command. It also hide (and shows back on completion) any HTML button or a element inside its closest host ancestor with a *c-menu-scope* class. It must be declared on the `data-command` attribute of the command it is hooked to.

If the command is a `save`, then it starts showing the message on the `axel-save` event of the target editor. It stops showing it on any of the `axel-save-done`, `axel-save-error` or `axel-save-cancel` event.

Otherwise it starts showing the message on the `axel-transaction` event of the target editor. It stops showing it on the `axel-transaction-complete` message. For instance the `status` or the `c-delete` command trigger these *transaction* events.

The message to show must be available in the web page as an HTML span element with a *c-saving* class direct child of an HTML element with a *c-saving* id. The command clones the span element each time it needs it. You can for instance add a spinning wheel to the span.


## The `drawer` command

> In: lib/workflow.js
>
> Implements: Drawer (in Tab) in workflow.xsl

The `drawer` command manages opening and closing of a drawer inside a tab. It must be placed on the drawer div that contains the drawer editor.

This is used for instance to manage the case and activity messages drawers.

## The `edit` command

> In: lib/workflow.js
>
> Implements: Edit (in Actions in Document) in workflow.xsl

The `edit` command manages a single document editor view inside a drawer of an Accordion widdget. 

The execution of the command loads the document template in update mode and loads the document date. It also hides or shows the editor's menu (and its replica at the bottom of the document) with the Save or Cancel or any other action buttons.

It works with a _template_ command setup on the drawer's body. This _template_ command is shared with a _view_ command hosted on the drawer.

## The `header` command

> In: lib/workflow.js
>
> Implements: AlertsLists and Activities in workfow.xsl

The `header` command must be hosted on the HTML thead element of a table. It subscribes to its target editor and removes any *c-empty* class on the command's table parent each time the editor emits 'axel-save-done' event. This is useful to pre-generate a table with headers and make it invisible as long as it is empty.

The command can also subscribe to a second optional target editor with the `data-event-target` if the table content may be updated from a secondary editor. 

The command can also maintain a dynamical count to display the number of table rows.

Optional attributes :

* `data-event-target` secondary editor to listen to
* `data-counter` id of a counter to update (text content) with the number of table rows

## The `open` command

> In: lib/extensions.js
>
> Implements: Open (in Plugins) in supergrid.xsl

The `open` command opens a a URL in a new window. The URL can use the URL micro-syntax described in [Supergrid specification language](./supergrid-spec.md).

The actual implementation uses a pre-generated HTML form element submission to open the new window.

Mandatory attributes :

* `data-form` is the id of the form element to submit
* `data-src` is the URL to open, it will be set as the form *action* attribute before submitting

## The `status` command

> In: lib/workflow.js
>
> Implements: ChangeStatus (in Actions in Document) in workflow.xsl

The `status` command POST a change status request to the case tracker.

This command should be hosted on an HTML list element with specific parameters set on each list item. Each list item triggers a different change status request.

## The `view` command

> In: lib/workflow.js
>
> Implements: Document in workflow.xsl

The `view` command manages a single read-only document editor inside an Accordion widdget document.

Exceptionally this command does not subscribe to any user action. Instead it is executed indirectly by the _openAccordion_ function of `workflow.js`. That function is called when the user opens the drawer by clicking on the document bar. 

The execution of the view command loads the document template in read-only mode and loads the document data inside it. The command also listens to _axel-save-done_ and _axel-cancel-edit_ events to reload the template and the data.

It works with a _template_ command setup on the drawer's body. This _template_ command is shared with an _edit_ command hosted on the Edit button of the document menu bar.



