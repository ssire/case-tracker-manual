# Workflow definition language

The case and activity workflow of the application are defined in the `config/application.xml` file. They are written using the workflow definition language.

These definitions have multiple purposes among which :

* to generate the **workflow accordion view** (see also the Accordion widget in [UI components](components.md)). This is implemented by the `modules/workflow/workflow.xqm` XQuery module and by the `modules/workflow/workflow.xsl` view library;

* to control and to apply the **workflow transitions** in the case and activity `status.xql` controllers;

* to control the **generation of automatic and semi-automatic e-mail notifications**;

* to control **access rights on workflow document and other resources** in every document CRUD controller.

The workflow accordion view shows documents in tabs in an accordion widget. Each tab displays a document in read-only mode which can be turned into a document editor if the user has enough access rights. The document may also contain satellite documents which can be edited in a modal window. The buttons to edit satellite documents are visible only if the user has enough access rights. Some read-only documents may be composite documents made of aggregated data from several documents. Some documents may only be edited through satellite documents.

Note that the current implementation uses numbers to identify the case or workflow status. This is used to trigger status change by incrementing or decrementing the current status by a fixed increment or decrement number.

## Document skeleton

The workflow specification skeleton looks like :

```xml
<Application>
  <Messages>
    ...Email elements...
  </Messages>
  <Workflows>
    <Workflow Id="Case">
      <Documents TemplateBaseURL="../templates/">
        ...Document elements...
      </Documents>
      <Transitions>
        ...Transition elements...
      </Transitions>
    </Workflow>
    <Workflow Id="Activity">
      ...same as above..
    </Workflow>
  </Workflows>
  <Security>
    <Documents>
      ...Document elements...
    </Documents>
    <Resources>
      ...Resource elements...
    </Resources>
  </Security>
  <Description>
    ...Role elements...
  </Description>
</Application>
```

## Common elements

Some elements are reused in different parts of the workflow definition, these are the `Email` element to specify notification e-mails and the `Meet` element to specify access control rules.

### Notification e-mail : the `Email` element

The `Email` element is a common building block which appears in different parent elements to specify a notification mail.

The `@Template` attribute is the name of the e-mail template to use to generate the e-mail message and some extra headers. E-mail templates must be defined in the `data/global-information/email.xml` file as an *Alert* or *Email* element with a corresponding `@Name` attribute (TODO: see also the Media engine documentation).

The `Recipients` element lists the recipients of the message. It's textual value is the list of direct recipient (To: field). The optional `@CC` attribute is the list of indirect recipients (carbon copy or CC: recipicents). The *recipients* syntax follows the application *roles specification micro-language* syntax.

Example : 

```xml
<Email Template="coaching-plan-submission">
  <Recipients CC="r:kam">r:coach</Recipients>
</Email>
```

### Access control : the `Meet` element 

The `Meet` element of the workflow specification contains an expression that defines a set of users by their roles. This can be used for instance to test if the current user is member of the set and can perform some actions.

The expression contains a whitespace separated list of **role tokens**. Each role token starts with a scope letter followed by a colon and by a role name. The scopes are :

* `g` for global scope (or static role) : the targeted users are independent from the database content (e.g. a system administator, a coaching assistant, etc.)
* `r` for relative scope (or dynamical role) : the targed users depends on a current case or activity context (e.g. the region manager for a case or the coach for an activity)

The `Meet` element inside a `Transition` element defines who can trigger the transition.

The `Meet` element inside an `Action` element defines who can execute a given action. This concerns either an `Action` element linked to a workflow document (in the `Documents` section of the `Security` section), or an `Action` element linked to another type of application resource (in the `Resources` section of the `Security` section).

The different situations are summarized on the table below.

| Context           | `Security` section  | Supported scope | Implemented by  |
|:------------------|:--------------------|:----------------|:----------------|
| case or activity  | `Documents`         | r:, g:          | *pre-check-case*, *pre-check-activity* |
| other resource    | `Resources`         | g:              | *check-omnipotent-user-for* |

The *Implemented by* column refers to the function(s) implementing the `Meet` element in the `lib/access.xqm` file. These functions must be called in the CRUD controllers of the workflow documents and of the resources to prevent unauthorized operations, and on the view generation code to insert the editing commands.

Example:

```xml
<Document TabRef="case-init" Root="Information" Form="case-information.xml">
  <Action Type="update">
    <Meet>g:coaching-manager g:coaching-assistant r:kam</Meet>
  </Action>
</Document>
```

The example means that any coaching manager or coaching assistant can update the document, or the region manager (KAM) for the case to which belong the document under consideration.

## The `Messages` section

The `Messages` section is a placeholder to define e-mail notifications which are not directly triggered by workflow transitions but which can be triggered by other means in any part of the application code. The `Messages` section of the workflow definition language has been defined to group them together for convenience.

The `Email` elements in the `Messages` section may also use the following extra-atrtibutes :

* the `@Context` attribute tells in which document context (document root name) the e-mail maybe triggered; this applies to the case when the e-mail sending is triggered by the CRUD controller of the document (e.g. saving the document triggers a notification e-mail)

* the `@In` attribute tells which XQuery script actually sends the e-mail

These two attributes are used for documentation purpose only.

Example :

```xml
<Email Template="coach-contracting-start" Context="FundingDecision" In="alert.xql">
  <Recipients CC="r:kam">r:coach</Recipients>
</Email>
```

In this example the *coach-contracting-start* e-mail is a semi-automatic e-mail which appears pre-generated in a modal window when the user saves the funding decision document (note that the controller also checks the funding decision has been approved but this does not appear in the `Email` element defintion). It is sent in a second time by the *alert.xql* script if the user confirms (i.e. clicks on *Send* button).

## The `Workflows` section

### The `Document` element inside the `Workflow` elements

Each `Document` element inside the `Workflow` element defines a tab of the Accordion widget displaying the workflow documents. 

The `@Tab` attribute gives a unique name to the tab. Note that this is not a unique ID since the same tab can be defined both in the case and in the activity workflow. The `@Tab` attributes must match the `@TabRef` attribute of a `Document` element inside the `Security` section.

The `@AtStatus` attribute is a whitespace separated list of the worklfow status at which the tab is visible. 

The optional `@Blender` attribute must be set to *yes* if the document within the tab uses the *XSLT blender* to render in read-only mode.

The `@class` attribute adds the corresponding class attribute to the accordion tab rendering. This is used to color tab menu bar depending on the workflow origin of the document (the workflow origin beeing the workflow from which the document is initially created).

The unique `Controller` element is the name of the controller to load/save the document.

The unique `Template` element is the name of the template to display the document.

Both the *Controller* and *Template* names are concatenated with the `@TemplateBaseURL` attribute of the parent `Workflow` element to generate the URL of the corresponding resource.

Then if may contain one or mor *Action*, *AutoExec* and *Host* elements.

Example :

```xml
<Document Tab="funding-request" AtStatus="2 3 4 5 6 7 8" Blender="yes">
  <AutoExec AtStatus="2" Id="ae-advance" Mode="direct">
    <Forward Command="status" EventTarget="go-consultation">cmd-change-status</Forward>
  </AutoExec>
  <Controller>funding-request</Controller>
  <Template>funding-request</Template>
  <Action Type="status" AtStatus="2" Id="cmd-change-status"/>
  <Action Type="update" AtStatus="2" Forward="submit"/>
  <Host RootRef="SME-Agreement">
    <Action Type="update" AtStatus="2"/>
    <Flag Name="smeagree" Action="update"/>
  </Host>
</Document>
```

#### The `Action` element

The `Action` elements specify the types of actions that can be executed on the document in the tab. These elements are used to generate the accordion menu bar of the tab, usually one button per action.

The `@AtStatus` attribute is a whitespace separated list of the worklfow status at which the action is available. 

The `@Type` specifies the type of an action :

* *update* : update the document
* *create* : create the document
* *status* : change workflow status
* *spawn*  : create a new activity from a case workflow (very specific)

The *status* action generates a *status* command. The optional `@Id` attribute is used to generate an *id* for the DOM host element of the generated command (drop down list). This can be used to control the status change as a tierce command triggered from an *autoexec* command as explained below to implement automatic satus change on save. The list of options is generated from the current workflow status and the list of available transitions as defined in the `Transitions` section of the workflow.

#### The `AutoExec` element

The optional `AutoExec` element generates an *autoexec* command in the accordion view with an *id* set to it's `@Id`. It implements the `forward` element of the Ajax response sent by a document CRUD controller on saving as explained in the [Ajax protocols](ajax.md) chapter. This way server-side code can trigger the pre-generated tierce command execution  through the Ajax response.

The `Forward` element identifies the tierce command.

The `@Command` attribute of the `Forward` element gives the name of the tierce command to execute (e.g. *status* to trigger a status change). The text value of the `Forward` element gives the *id* of the DOM element hosting the tierce command. The optional `@EventTarget` attribute allows to simulate a synthetic user event with a *target* property. This is useful when the tierce command is naturally executed by a click or a selection in a menu (e.g. the *status* command is triggered by selecting an option in a drop down list). 

#### The `Host` element

Each document may host one or more satellite documents declared through a `Host` element. A satellite document is a document which can be displayed inside the main tab document but which has its own editor appearing in a modal window triggered by a specfic edit button.

The `@RootRef` attribute points to the `@Root` attribute of a `Document` element inside the `Security` section that defines access control rules.

The `Action` element ...

The `Flag` element drives the generation of the flag parameters of the HTTP request loading the document template in the tab. It contains the name of a flag to add to the generated template URL if the user can perform the action on the host document. This flag is usually associated to a `site:conditional` rule in the tab document formular specification (see [Supergrid](supergrid.md)) to display or not the action button.

### The `Transition` elements inside the `Workflow` elements

TBD

## The `Security` section

The `Security` section defines access control rules for editing all the entities of the application. By entities that means workflow documents (e.g. a *funding request*) and other application resources (e.g. a *person*). 

The `Documents` section made of `Document` elements defines rules for editing workflow documents.

The `Resources` section made of `Resource` elements defines rules for editing the other types of resources. 

### The `Document` element 

The `Document` element defines the access control rules for updating a document.

The `@TabRef` attribute links a document with a tab (see above the `@TabId` attribute).

The `@Root` attribute links a satellite document with a host document (see above the `@RootRef` attribute). It must match the name of the XML element storing the document in the case resource.

There is an `Action` element for each type of action allowed on a document (e.g. *update*, *delete*). It contains rules to define who is allowed to perform the corresponding action. Rules are expressed using the access control *rules specification micro-language* syntax.

Example :

```xml
<Document Root="SME-Agreement" Form="email.xml">
  <Action Type="update">
    <Meet Policy="strict">r:coach</Meet>
  </Action>
</Document>
```

The example tells that the document with root SME-Agreement can only be update by the coach of the activity. 

### The `Resource` element

The `Resource` element defines the rules to edit an application resources different from a workflow document. These resources are usually stored outside of the *cases* collection and have their own controllers (one per resource type).

The `Action` element contains for each type of action the access control rules defined with the *acces control rules specification language*.

These rules are implemented by the *access:check-omnipotent-user-for* function of the `lib/access.xqm module`. This function must be called to enforce access control rules before updating an application resource in its CRUD controller. It can also be used to pre-generate the edit buttons and widgets.

Actually *access:check-omnipotent-user-for* only supports static roles (i.e. starting with *g:*).

Example :

```xml
<Resource Name="Person">
  <Action Type="create">
    <Meet>g:coaching-manager g:coaching-assistant</Meet>
  </Action>
  <Action Type="update">
    <Meet>g:coaching-manager g:coaching-assistant</Meet>
  </Action>
  <Action Type="delete">
    <Meet>g:coaching-manager g:coaching-assistant</Meet>
  </Action>
</Resource>
```

## The `Description` section

The `Description` section is used to store full text description of the different role expressions used by the application and other meta-data. This an be used for diverse purposes, for instance to generate a view of users by roles in the management section of the application.

The `Role` element describes a role.

Example : 

```xml
<Role>
  <Name>r:coach</Name>
  <Legend>Coach of coaching activity</Legend>
  <Domain>ServiceRef</Domain>
</Role>
```
