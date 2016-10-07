# Case Tracker Javascript Commands

## Invocation

### Client-side

Commands are usually triggered by user clicking the command host element (i.e. HTML DOM element holding the _data-command_ attribute).

### Server-side

Commands are triggered by returning an Ajax response containing a _forward_ element under the root element :

    <forward command="{command name}">{ command host element id }</forward>

## List of commands

### The 'status' command

This command POST a change status request to the case tracker.

This command should be hosted on an HTML list element with specific parameters set on each list item. Each list item trigger a different change status request. 


### The 'autoexec' command

This command triggers a target command with an optional event target.

This command is meant to be triggered server-side through a _forward_ element.

This command is generated with the following DSL language (in _application.xml_)

    <AutoExec AtStatus="1" Id="ae-advance" i18nBase="action.status">
      <Forward Command="status" EventTarget="go-coaching-plan">cmd-change-status</Forward>
    </AutoExec>

### The 'view' command

File: `workflow.js

This command manages a single document read-only viewing inside a drawer of an Accordion widdget.

Exceptionally this command does not subscribe to any user action. Instead it is executed indirectly by the _openAccordion_ function of `workflow.js`. That function is called when the user opens the drawer by clicking on the document bar. 

The execution of the view command loads the document template in read-only mode and loads the document data inside it. The command also listens to _axel-save-done_ and _axel-cancel-edit_ events to reload the template and the data.

It works with a _template_ command setup on the drawer's body. This _template_ command is shared with an _edit_ command hosted on the Edit button of the document menu bar.

### The 'edit' command

File: `workflow.js

This command manages a single document editor view inside a drawer of an Accordion widdget. 

The execution of the command loads the document template in update mode and loads the document date. It also hides or shows the editor's menu (and its replica at the bottom of the document) with the Save or Cancel or any other action buttons.

It works with a _template_ command setup on the drawer's body. This _template_ command is shared with a _view_ command hosted on the drawer.

### The 'header' command

To be set on a thead in a table. Subscribes to its editor and removes any c-empty potential class on the command's table parent each time the editor emits 'axel-save-done' event. Subscribes to a second optional target data-event-target if provided.

### The 'drawer' command

This command tracks drawer button to open / close drawer for drawers inside accordions. It MUST be placed on the drawer div that contains the drawer editor.

### The 'acc-drawer' command

### The 'confirm' command

### The 'augment' command

### The 'c-delete' command

### The 'c-inhibit' command

### The 'open' command




