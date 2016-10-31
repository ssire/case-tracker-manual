# Ajax protocols

Application pages communicate with the server with different Ajax protocols to issue commands to the server and to retrieve changes to update the user's page or redirect to a new page.

Examples include :

* search for cases from Case Tracker dashboard on hitting Search button
* saving a document from a case or activity panel on the document editor embedded inside the workflow accordion
* editing a user's roles from the management user interface
* etc.

Basically an Ajax protocol consists of :

* request payload (XML)
* zero or more request parameters (URL query parameter string)
* response status code
* response HTTP headers (e.g. *Location* header for client-side soft redirection)
* response payload (XML or JSON)

The request payload is usually an XML document sent with POST to the server (available in XQuery with `request:get-data` or `oppidum:get-data`). It may also by completed with some request parameters (available in XQuery with `request:get-parameter` ). In few cases the XML document payload may be empty.

The response payload contains an XML document which can eventually be serialized as JSON. That later case is specially interesting to generate tables or graphics using the d3.js library. The only thing to change response payload from XML to JSON is to call `util:declare-option("exist:serialize", "method=json media-type=application/json")` before the end of your XQuery code.

The description of an Ajax protocol is most of the time the description of the request XML vocabulary and of the XML response vocabulary.

The Case Tracker commands and user interface are built with a few Ajax protocols describes in this section. Usually when you need to develop a new command, you should try to apply / extend one of the existing protocols instead of reinventing a new one.￿

The `lib/ajax.xqm` XQuery module is a low-level helper module to generate responses to the client using Ajax protocols.

AXEL-FORMS javascript commands and Case Tracker custom javascript commands often implement one of the Ajax protocols.

## Ajax XML / JSON Oppidum success / error

The response payload must contain a *success* or *error* root element. Then it may contains zero or more child elements to trigger different actions client-side. They can be combined to implement complex feedbacks and side-effects.

Example success response :

    <success>
      <message>...</message>
    </success>
  
Example error response :

    <error>
      <message>...</message>
    </error>

The reponse vocabulary of most builtin AXEL-FORMS commands like the *save* command includes the following *default actions* : 

- the HTTP `Location` header
- the `message` element
- the `forward` element
- the `payload` element

The HTTP `Location` header is a soft Javascript browser redirection using `window.location.href`. This one is exclusive from the other actions.

The `message` element displays an alert message with the browser's Javascript *alert* function.

The `forward` element executes another client-side AXEL-FORMS command which must be already contained (pre-generated) into the page. The `@command` gives the name of the command to execute (e.g. *autoexec*) and the textual content is the *id* of the DOM element hosting the command.

For instance the *forward* element is used to trigger a status change after saving a document on the accordion of a case or activity. 

The `payload` element is a conventional element name to store additional payload. This payload can be used to update page content. It can be used to create more complex Ajax protocols like the JSON table protocol (see below).

The AXEL-FOMS file `core/oppidum.js` exports a *filterRedirection*, a *handleMessage* and a *handleForward* functions into an `$axel.oppidum` API object which can be used to integrating the default actions into your custom commands. You can have a look at the `saveSuccessCb` function of the `save` command implementation (into `commands/save.js`) for an example.

Oppidum success / error protocol is implemented server-side with `oppidum:throw-error` and `oppidum:throw-message` or their variant `ajax:report-success-redirect`, `ajax:report-success` and `ajax:report-success-json` (including a *payload* parameter) available in `lib/ajax.xqm`.

## Ajax XML custom protocol

You are encouraged to extend Oppidum success / error protocol by adding more action elements to the response vocabulary. This way you can create custom protocols.

## Ajax JSON table protocol

The JSON table protocol interprets the payload element as an array of rows to display in a table. The Table key identifies the target table. The Action key specifies how to update the rows, either replacing the existing one (*create* value) or adding to the existing one (*update* value). Finally the rows are available in a Users hash key that contains an array. Each row is a hash entry, usually there is one key by column (but they may be more). The hash is interpreted by the Javascript encoding function from which the table command has been built (see the `$axel.command.makeTableCommand` table factory).

Exempe success response (JSON hash) :

    payload :
      Table : table name
      Action : create, update
      Users : [ array of users rows ]

*NOTE*: rows are stored in a *Users* array, this will most probably renamed *Rows* in the future as the table factory component is still under development

