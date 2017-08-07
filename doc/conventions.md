# Code and data model convention

## Attribute vs. Element conventions

The use of attributes in data modelling is reserved to very specific cases. The rule of thumb is to prefer modelling using elements.

## Tag name conventions

Unless specified otherwise the tag name of an element determines its category. The tag name usually represent a data type or an application entity. 

The most common constructions involve :

* complex data types : a tag name that contains other tag names
  * repeatable data type : a tag name that contains a repetition of similar tag names
* terminal data type : a tag name that represent a terminal data value (e.g. a string, an integer, a date, etc.)
* reference data type : a tag name that represent a reference to something
  * a reference to a selector data type (name ending with Ref)
  * a reference to an application entity (name ending with Key)
* selector data type : always a `Selector` tag name that represent a finite list of possible values (e.g. a day of the week)

Application *entities* (e.g. a `Case`, a `Person`) are implemented with complex data types.

Available selector data types are stored in the `/db/sites/{application}/global-information` collection (usually copied from the *global-information* folder).

### Tag name ending with Ref

Names ending with Ref represent selector data types (eg. `PartnerRoleRef`, `CommunicationAdviceRef`).

The corresponding selector data type is derived from the tag name by removing the Ref termination and taking the plural form of the corresponding prefix (e.g. `PartnerRoles`, `CommunicationAdvices`).

This rules is implied when you called the *display:gen-name-for* and *misc:unreference* functions to respectively convert a reference data type to a string or to unreference a complete XML fragment to load into formular.

Known exceptions : 

* tag name `Country` refers to the `Countries` selector data type
* in `StatusHistory` data type 
  * `CurrentStatusRef`, `PreviousStatusRef` are used to encode workflow status
  * `ValueRef` encode a workflow status independently of the selector data type reprensenting the workflow status (e.g. `CaseWorkflowStatus`, `ActivityWorkflowStatus`)


### Tag name ending with Key

Names ending with Key represent a reference to an application entity (e.g. `EnterpriseKey`, `PersonKey`).

The name of the collection containing the corresponding entity is derived from the tag name by removing the Key termination and adding '-uri' to the lower-case version of the plural form of the suffix (e.g. `enterprises-uri`, `persons-uri`).

The tag name of the dereferenced entity is derived from the tag name by removing the Key termination (e.g. `Enterprise`, `Person`).

This way you can use the *globals:collection* function to access the given resource such as in `globals:collection('enterprises-uri')//Enterprise[Id eq $key]`.

### Tag name always associated with a data type

Some tag names are always used to represent content with a similar data type.

#### Comment/s tag name

The `Comment` tag name (or `Comments` tag name) always represent user generated textual comments :

- the singular form `Comment` represents a single line of text, text is directly stored inside the `Comment` element
- the plural form `Comments` represent either :
  * multiple lines text entries, each entry is contained in a `Text` element
  * rich text content composed of multiple text entries, each entry is either a `Parag`, a `List` or a `Title` element

The nature of the `Comment` may be encoded into the data model either :

* using a more specific tag name ending with `Comment` (or `Comments`) such as in `ResponsibleCoachComment` or `PositiveComments`
* by storing the `Comment` (or `Comments`) element inside a more specific container element (e.g. a `Comments` inside a `Challenge` element; a `Comments` inside a `Dissemination` element)

The second method is a powerful modelling trick since it is then possible to complete the comments with some ratings, such as a `RatingScaleRef` element for instance.

### Other tag names not ending with Ref or Key

All the other tag names represent complex data types or terminal data types.

There is no specific convention to devise the corresponding data types from the tag name.

The only convention is that a repeatable data type should be named with a plural form (e.g. `Activities`, `Alerts`, `Tasks`). Whenever possible the repeated content should be named using the singular form (e.g. `Task` repeated inside `Tasks`).

