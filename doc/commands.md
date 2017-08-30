# Case Tracker Javascript commands

Commands are Javascript objects bound to an HTML element or *host element*, like a button or a drop-down list, with a `data-command` microformat attribute. They can be directly invoked client-side by user interaction or indirectly as a side effect of an Ajax response generated server-side.

In **client-side invocation** the commmand is triggered by user clicking the host element.

In **server-side invocation** the command is triggered by returning an Ajax response containing a `forward` element as explained in the [Ajax protocols](./ajax.md) chapter.

Commands are implemented in different files :

* `resources/lib/extensions.js` contains generic commands (TODO: rename to `widget.js`)
* `resources/lib/workflow.js` contains commands specific to the workflow accordion view, they are instantiated in `modules/workflow/workflow.xsl`

Commands often implement one or more elements from a domain specific language. You can get more details in the corresponding XML vocabulary chapters :

* [UI widgets vocabulary](./components.md) for vocabulary from workflow.xsl, widgets.xsl or commons.xsl
* [Workflow specification language](./workflow.md) for vocabulary from application.xml in workflow.xsl
* [Supergrid specification language](./supergrid-spec.md) for vocabulary from supergrid.xsl

## The `acc-drawer` command

> In: lib/workflow.js
>
> Implements: Document (containing Actions/Drawer) in workflow.xsl

NOT USED

The `acc-drawer` command manages opening and closing of a drawer inside a document inside an Accordion widget. The opening is triggered by an action button generated from a `Drawer` action.

## The `augment` command

> In: lib/extensions.js
>
> Implements: Augment in supergrid.xsl

The `augment` command opens a target modal window editor to edit an entity corresponding to the current value of a target field, or to edit and create a new entity and add it to the target field.

The target field is usually a selection list implemented by a compatible plugin, typically a `choice` plugin with or without a `select2` filter.

Mandatory atributes : 

* `data-augment-field` : CSS selector matching target field
* `data-augment-root` : CSS selector to scope data-augment-field search
* `data-target` : identifier target modal window editor
* `data-augment-mode`: create or update mode configuration
* `data-create-src` or `data-update-src` : URL of the controller where to POST new data to create an entity or where to GET and POST the existing entity data to update

Optional atributes : 
* `data-augment-noref` : warning to display when the target field is empty

Example :

```html
<button type="button" class="btn span3 btn-small btn-primary" 
  data-command="augment" data-target="case-enterprise"
  data-augment-field=".x-EnterpriseRef" data-augment-root=".x-ClientEnterprise" data-update-src="/exist/projects/scaffold/enterprises/$_.xml?goal=update" 
  data-augment-mode="update" 
  data-augment-noref="The activity does not contain any customer company">Edit company data</button>
```
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

## The `$axel.command.makeTableCommand` table factory

> In: lib/commons.js

The `$axel.command.makeTableCommand` is a command constructor function that creates and registers an AXEL command to manage rows of user data displayed inside a table. It also manages options to make each column sortable and filterable.

## The `open` command

> In: lib/extensions.js
>
> Implements: Open (in Plugins) in supergrid.xsl

The `open` command opens a a URL in a new window. The URL can use the URL micro-syntax described in [Supergrid specification language](./supergrid-spec.md).

The actual implementation uses a pre-generated HTML form element submission to open the new window.

Mandatory attributes :

* `data-form` is the id of the form element to submit
* `data-src` is the URL to open, it will be set as the form *action* attribute before submitting

## The `save` command

> In: axel-forms.js (native AXEL-FORMS command)
>
> Implements: Save in widgets.xsl

The 'save' command serializes the XML data contained inside a target editor and submit it with an AJAX POST request. By default it sends data with an `"application/xml; charset=UTF-8"` content type.

See more info on [AXEL-FORMS wiki: the 'save' command](https://github.com/ssire/axel-forms/wiki/The-%27save%27-command)

## The `status` command

> In: lib/workflow.js
>
> Implements: ChangeStatus (in Actions in Document) in workflow.xsl

The `status` command POST a change status request to the case tracker and usually redirects to a new page upon success.

The basic use of the command is to link it with a target editor in a hiden modal window : it will open the modal window upon success and load a template with optional initial data inside. The modal window can contain an e-mail composer window for instance and should contain its own command to save its content and close the window or redirect to a new page after saving.

Note that when the status change Ajax `success` response contains a `done` element it will prevent the opening of the modal window and it will directly redirect the browser as explained in the next paragraph.

The status change Ajax response must contain a *Location* header which is stored by the command while displaying the modal window. In case the target editor in the modal window triggers an *axel-cancel-edit* event or in case the modal window is closed, the status change command will redirect to the stored address. Usually the same redirection address should be returned by the Ajax response to the save command used by the target editor in the modal window to achieve the same redirection upon success.

You can also use the status command without a hidden modal window (no *data-target-modal* attribute). The status command will then immediately redirect the browser to the address returned by the Ajax *Location* header upon success. 

Note that in that later case you must anyway define a target editor, but it will not be used by the command. The rule of thumb is to use the editor associated with the menu where you place the status command. 

This command should be hosted on an HTML list element with specific parameters set on each list item. Each list item triggers a different change status request. It can also be hosted on a single HTML button when the list is reduced to one item (e.g. a *Submit* button to submit a previously edited formular form application).

Mandatory attributes :

* `data-target` is an editor embedded in a modal window (identifier with `data-target-modal`) or a neutral editor (unused by the command)
* `data-action` status change direction (increment or decrement)
* `data-argument` increment (or decrement) number
* `data-status-from` current status number from where to apply the change

Optional attributes :

* `data-confirm` text of a confirmation message to display before changing status
* `data-target-modal` identifier of the modal window containing the target editor
* `data-with-template` (mandatory with data-target-modal) template to load in the target editor of the target modal window
* `data-init` resource to load to initialize the target editor of the target modal window

```html
<button class="btn btn-primary" 
  data-command="status c-inhibit" 
  data-target="c-editor-event-form-apply-8" 
  data-status-from="1" 
  data-status-ctrl="8/status" 
  data-confirm="Are you sure you want to submit your application ? All fields marked as mandatory must be filled otherwise you won't be able to submit. Once you have submitted you can no longer edit the application form." 
  data-action="increment" 
  data-argument="1">Submit</button>
```

## The `view` command

> In: lib/workflow.js
>
> Implements: Document in workflow.xsl

The `view` command manages a single read-only document editor inside an Accordion widdget document.

Exceptionally this command does not subscribe to any user action. Instead it is executed indirectly by the _openAccordion_ function of `workflow.js`. That function is called when the user opens the drawer by clicking on the document bar. 

The execution of the view command loads the document template in read-only mode and loads the document data inside it. The command also listens to _axel-save-done_ and _axel-cancel-edit_ events to reload the template and the data.

It works with a _template_ command setup on the drawer's body. This _template_ command is shared with an _edit_ command hosted on the Edit button of the document menu bar.



