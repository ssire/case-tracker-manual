# Supergrid specification language

## Common patterns

### Document skeleton

A formular  specification skeleton is presented below :

```xml
<Form>
  <Title>
  <Verbatim>
    1 or more <xt:component>
  </Verbatim>
  1 or more <Include>, <Row>, <Separator> or <site:conditional>
  <Modals>
    1 or more <Modal>
  </Modal>
  <Commands>
    1 or more command <Add>, <Open> or <Augment>
  </Commands>
  <Bindings>
    1 or more <Require> or <Condition>
    <Enforce>
      1 or more <RegExp>
    </Enforce>
  </Bindings>
  <Plugins>
    1 or more plugin among <Constant>, <Date>, <Input>, <MultiText>, <Photo>, <Plain>, <Text>, <RichText>
  </Plugins>
  <Hints>
    1 or more <Hint>
  </Hints>
</Form>
```

The `Commands`, `Bindings`, `Plugins` and `Hints` sections bind objects with user interface elements declared in the previous sections using the key pattern.

### The `@Key` pattern

Most of the objects declared in Bindings, Commands, Hints or Plugins sections are hosted on a user interface element declared somewhere else in the formular specification (e.g. a *Plugin* is set on a *Field*). The association between the host user interface element and the object works with a common `@Key` attribute : 

* the host element defines its own `@Key`
* the associated object targets the host with a similar `@Key`
* some objects also support a `@Prefix` attribute to target all the hosts with a key starting with the prefix

Note that a *current limitation* is that two keys set on two host elements should not be mutually inclusive (i.e. part of the other)!

Note that a key may not be unique in a given formular to share some declarations.

### Common attributes

Some attributes are used consistently within the specification, these are :

* `@Tag` specifies a tag name in the XML output vocabulary of the formular
* `@Gap` specifies a horizontal space left for a label or a title in grid units (0 means a label above the data entry field)
  * integer values *X* are converted to a class name *gapX*, thus only a limited number of gaps are defined by CSS rules, add more rules if you want to use more gaps 
  * decimal values (e.g. 1.1) are computed as fractions of a standard gap at formular installation time, so the computed gap will remain fixed (e.g. 66px) instead of using relative units and will not adapt to the local width of its container
* `@W` specifies the total horizontal space occupied by the field in grid units (12 maximum)
* `@L` specifies the left margin (usually set to 0 when vertically stacking fields w/o wrapping them into rows)
  * often used with 0 to reset a new line of `Field` elements without explicitly creating a new `Row`

Note that the default grid unit is 60px and a 20px gutter.

###  The URL micro-syntax

The URL micro-syntax is a facility to generate relocatable URLs independents of the current page address. They are used in a formular that embeds other formulars (e.g. in modal windows) to make the formular less dependent on the URL of the page that contains it.

The URL micro-syntax is usually employed on `@Template` or `@Resource` attributes that declare the address of a formular or a resource controller.

The URL micro-syntax is based on two prefixes :

* the ^ (carret) prefix concatenates the base URL as defined by the closest *data-axel-base* attribute found in the formular container parents (jQuery *closest* function), typically it should be set to the application base URL to get mode independent URLs (*dev* and  *prod* or *test* modes diverge with the presence or not of the */exist/project/cctacker* URL prefix);
* the ~ (tilde) prefix concatenates the current location path in front of the URL, this is convenient to keep the last segement path of the URL when the URL doesn't end with a trailing /

And on a variable : 

* $^ is replaced by the latest location path segment in front of the URL, this is convenient to address the case or activity from a URL of one of the case or activity document (TODO: find example)

The prefixes syntax does not preserve hash tag or query parameters of the URL to expand so you must not need it !

The URL micro-syntax is implemented by the *$axel.resolveUrl* Javascript function which is defined in AXEL-FORMS `command.js`.

## The `Form` element

> Contained in: document root
>
> Contains: Verbatim, Title, Row, Separator, Include, site:conditional, Modals, Commands, Bindings, Plugins, Hints

The `Form` element is the document root of a supergrid formular definition. 

Mandatory attributes :

* `@Tag` specifies the name of the root element of the XML output of the formular

Optional attributes :

* `@StartLevel` specifies the starting level for all the `Title` elements, for instance  when set to 1 a *Title* (w/o extra *Level* attribute) will be rendered by a `h1` element
* `@Width` specifies the desired width for generating the formular in the Supergrid simulator, note this is not necessarily the width that will be used in the application which is depending of the container


In addition you can specify namespaces used in the formular definition as in the next example :

```xml
<Form Tag="Information" StartLevel="1" Width="800px"
  xmlns:site="http://oppidoc.com/oppidum/site"
  xmlns:xt="http://ns.inria.org/xtiger">
  ...
</Form>
```

## The `Verbatim` element

> Contained in: Form
>
> Contains: one or more xt:component XTiger component definitions

The `Verbatim` element allows to generate a verbatim block in the `xt:head` section of the generated template. 

This is used to generate explicit XTiger components that you can later on instantiate into your grid layout with the `Use` element.

Note that the XTiger components (e.g. `xt:component` elements) may contain any markup or Supergrid markup and its name must be prefixed by *t_*.

Example : 

```xml
<Verbatim>
  <xt:component name="t_person_name">
    <Field Key="firstname" Tag="FirstName" W="6" Gap="1" L="0">Name</Field>
    <Field Key="lastname" Tag="LastName" W="6" Gap="1">Surname</Field>
  </xt:component>
</Verbatim>
```

which can be instantiated anywhere in a `Row` or a `Cell` with : 

```xml
<Use Tag="Name" TypeName="person_name"/>
```

## The `Row` element

> Contained in: Form, Cell
>
> Contains: any element (most likely Field)

A `Row` element stacks one or more `Field` elements in lines of a maximum 12 grid unit width and with a fixed left margin (usually *15px*). The left margin allows to create grid gutters in regular layouts (e.g. each line containing two 6 grid units width fields). Each new Row element starts with a zero left margin.

The field placement algorithm in a row works as follows: if there are enough grid units available to contain the full field in the current line, then the field is inserted into the line, otherwise a new line is created that starts with the field. Note that except on the first line which has no left margin set, you must explicitely set the left margin to 0 on a new field on a new line using the `L="0"` attribute of the `Field` element. Alternatively you can also use a new `Row` element each time you want to create a line. This more explicit version is more verbose.


## The `Separator` element

> Contained in: Form

The `Separator` draws a horizontal separation line

## The `Cell` element

> Contained in: Row
>
> Contains: (Title | SideLink), any element (most likely Field or Row)

The `Cell` element content area uses fluid layout, so that means it will change the grid unit into its content so that 12 units correspond to 100% of the width of the cell content.

The `Title` element contains the title to display in the cell left space, or above if `@Gap` is explicitly set to 0.

Optional attributes (see  *Common attributes*):

* `@Tag`
* `@Gap` specifies the left space
* `@W` specifies cell full width
* `@L`

Example :

```xml
<Cell W="7" Gap="1.1">
  <Title>Address</Title>
  <Field Key="street" Tag="StreetNameAndNo" W="12" Gap="1">Street &amp; no</Field>
  <Field Key="box" Tag="PO-Box" Gap="1">Box number</Field>
  <Field Key="careof" Tag="Co" Gap="1">c/o</Field>
  ...
</Cell>
```

## The `Field` element

The `Field` element is the placeholder for a label followed by a data entry field.

The label is the text content of the element or a dictionary key if you use a `loc` attribute for internationalization. 

The nature of the data entry field is determined either by : 

- an input field in the *Plugins* section, it is statically generated once during form installation
- a`<site:field Key="...">` extension point generated by formular`s `form.xql` model, it is dynamically generated at each formular template generation

Mandatory attributes :

* `@Key` key to identify the field

Optional attributes (see *Common attributes*):

* `@Tag`
* `@Gap`
* `@W`
* `@L`

## The `Button` element

> Contained in: Row, Cell, site:conditional
>
> Contains: #text

The `Button` element creates an HTML button element. It is mostly used to execute an associated command as described in the section on *Commands* below.

The `@Key` identifies the button for associating a command.

Optional attributes :

* `@W` specifies the button width in in grid units (defaults to 12);
* `@StickyClass` (*DEPRECATED* ?) sets the whole value of the button class attribute; 
* `@Class` adds extra class(es) to the button, this is not compatible with the use of a *StickyClass* attribute;
* `@Id` or `@id` sets the id of the button, use either one but not both;
* `@loc` and `@style` are transferred to the HTML button element.

## The `Use` element

> Contained in: Row, Cell, site:conditional
>
> Contains: empty

The `Use` element instantiates an XTiger component defined in a `Verbatim` element. 

Mandatory attributes : 

* `@TypeName` contains the name of the component, by convention it will be prefixed with *t_*

Optional attributes (see *Commons attributes*):

* `@Tag`

Example : 

```xml
<Use Tag="Address" TypeName="address"/>
```

## The `Modals` section element

### The `Modal` element

The `Modal` element defines a modal editor window. A modal editor window is a pre-configured modal window that loads a formular template for viewing or editing a resource. It is used by some commands as described in the next section, but it can also be used programmatically. The modal window is actually implemented with a bootstrap modal (a header, a body and a footer).

Mandatory attributes : 

* `@Id` defines the modal identifier which is generated as _{@Id}-modal_. The identifier itself is used to target the editor generated inside the modal, for instance using the *$axel* wrapped set functional object;
* `@Width` specifies the modal width in pixels, the *px* unit must be included.

Optional attributes : 

* `@Template` generates a *data-template* attribute to configure the _transform_ command generated on the body. (TODO: explain when template is loaded). It can use the URL micro-syntax (see above);
* `@EventTarget` generates a *data-event-target* attribute to tell the save command to duplicate events on the corresponding element; 
* `@AppenderId` generates a *data-replace-type="event append"* and a *data-replace-target* attribute equal to its text content;
* `@PrependerId` generates a *data-replace-type="event prepend"* and a *data-replace-target* attribute equal to its text content;
* `@SaveLabel` changes the label of the Save button

The `Title` element defines the title displayed in the header. By default it displays a cross to dismiss the modal window without performing any action. You can eventually remove the cross by adding a `@Dimiss="none"` attribute.

The optional `Footer` element defines actions to display in the footer of the modal window. By default the footer contains a Save and a Cancel buttons :

* the Save button holds a _save_ command,  some optional attributes on the `Modal` element (see below) can be used to configure the command;
* the Cancel button defines a _trigger_ command that sends an `axel-cancel-edit` event on the editor contained inside the modal.

Example : 

```xml
<Modal Id="c-entity-assignment" Width="700px" EventTarget="c-editor-case-init">
  <Title>EEN Managing Entity Assignment</Title>
</Modal>
```

## The `Commands` section element

> Contains: Add, Augment, Open

The `Commands` section declares AXEL commands to install on a `Button` host element of the formular (see above).

The `@Key` attribute of the command element matches the button `@Key` attribute. 

A current limitation is that a key should not be contained inside another key. It is theoritically possible to declare several buttons with the same key to instantiate the same commands multiple times.

### The `Add` element 

The `Add` element opens a modal editor window for editing a satellite document or creating a document (e.g. an e-mail message) 

Mandatory attributes : 

* `@TargetEditor` specifies the `@Id` attribute of a modal editor window that can be created with a `Modal` element (see above);
* `@Template` and `@Resource` attributes specify the address of the template and of the resource to dynamically load inside the modal editor window. They can use the URL micro-syntax (see above).

Note that you may trigger an *add* command with the `forward` element of an Ajax response (see [Ajax protocols](./ajax.md) chapter), the command is identified by the `@Id` attribute of its button host element. If you do not want the user to be able to execute the command, you need to hide the button element for instance with the `@Class` attribute of the button because the button is required to generate the command.

Example :

The following `Button` element 

```xml
<Button Id="coach-contracting-start" Key="cde.coach-contracting-start" Class="btn-small btn-primary hide" W="2" style="margin: 0px 0 0 10px">Send Email</Button>
```

is attached to the command 

```xml
<Commands>
  <Add Key="cde.coach-contracting-start" TargetEditor="c-coach-contracting-email" Template="templates/notification?goal=create&amp;auto=1" Resource="~/alerts?goal=init&amp;from=FundingDecision"/>
</Commands>
```

that will open the following modal editor 

```xml
<Modals>
  <Modal Id="c-coach-contracting-email" Width="800px" EventTarget="c-editor-funding-decision" PrependerId="c-activity-alerts-list" SaveLabel="action.email">
    <Title>Send notification about start of contracting process to Coach</Title>
  </Modal>
</Modals>
```

### The `Augment` element

The `Augment` element opens a modal editor window for editing a transclusion part of a document.

*To be explained*


### The `Open` element

The `Open` element triggers an HTML form element submission. This is convenient for instance to invoke a service and show the result in a new window.

Example :

```xml
<Commands>
  <Open Key="btn.contract-list" Resource="~/funding-decision/contracts" Form="c-open-form"/>
</Commands>
```

## The `Bindings` section element

> Contains: Require, Enforce (RegExp)

The `Require` element makes an input field mandatory

The `Enforce` section declares some contraints elements. Actually there is only a `RegExp` constraint.

## The `Plugins` section element

> Contains: Constant, Date, Input, MultiText, Photo, Plain, RichText, Text

The `Plugins` section declares basic editing widgets. Each widget MUST BE associated with one or more `Field` instances of the formular with the `@Key` pattern.

Most of the widgets are implemented by an AXEL plugin rendered with a single XTiger `xt:use` element. A few ones may also be complete higher level reusable XTiger components that will be copied to the formular inside one or more XTigger `xt:component` elements.

### The `Constant` widget

The `Constant` element renders read-only content of different nature depending on the value of its *Media* attribute : 

- `@Media="html"` : displays an XHTML blob using AXEL 'html' plugin (XML input in that case is usually generated by the XSLT blender)
- `@Media="image"` : displays an image using AXEL 'constant' plugin
  -  the image URL will be obtained by concatanating `$xslt.base-url` with the optional `@Base` relative path that you can declare on the *Constant* element and the XML input
  - in case of empty XML input it will show `{$xslt.base-url}static/cctracker/images/identity.png`
  - set the `img-polaroid` class on the HTML `img` tag

### The `Date` widget

TBD

### The `Input` widget

TBD

### The `MultiText` widget

TBD

### The `Photo` widget

TBD

### The `Plain` widget

The `Plain` element implements different types of editors depending on the value of its *Type* attribute :

- @Type="constant" :  
- @Type="text" :  
- @Type="number" :  

TBD

### The `RichText` editor

The `RichText` element generated a complete XTiger component for editing rich text (paragraphs, lists, etc.).

TBD

### The `Text` widget

The `RichText` element generated a complete XTiger component for editing rich text (paragraphs, lists, etc.).

TBD

## The `Hints` section element

TBD

