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

## Ajax table protocols

A generic user interface for editing list of items (e.g. user's accounts) is to present them in a table, a row representing one item. Each column represent one item property. From this design it is possible to edit item properties individually or collectively, for instance using a click on a property to open a model editor window.

Then the table must react to changes to the items :

- creation of new items
- updating of existing items

Over the time we have used different Ajax protocols to update the table which are described below.

### Ajax redirection protocol

> In: lib/search.js (implements Add person functionality)

This protocol redirects the browser to a page showing the new item. Thus it does not require to dynamically insert a new row in the table.

By convention the client should issue the POST item creation request with a *next* query property set to *redirect* to activate that protocol.

Example server-side implementation :

```xml
ajax:report-success-redirect('ACTION-CREATE-SUCCESS', (), concat($cmd/@base-url, $cmd/@trail, '?preview=', $newkey))
```

In this example the browser is redirected to the same page but with a *preview* query parameter containing the key of the new resource. Since it is called from a search page with a table showing the results list the effect is to prefill the results list with the new item.

### Ajax HTML table protocol

> In: lib/search.js (implements updating of a person functionality)
>
> NOTE: DEPRECATED

This protocol returns an HTML representation of the new table row. The client-side code must then replace the legacy row with the new one.

The server side implementation is usually a pipeline with an XSLT view, sometimes called *ajax.xsl* per convention.

The client side implementation can be done with the generic function below :

```javascript
// Updates a single result table row from the XHR responseXML returned when updating an item
function updateRow(xhr) {
  var buffer = $axel.oppidum.unmarshalPayload(xhr),
      m = buffer.match(/data-id="(-?\d+)"/),
      dest, skip;
  if (m) {
    dest = $('tr[data-id="' + m[1] + '"]');
    skip = parseInt(dest.children('td[rowspan]').attr('rowspan'));
    if (!isNaN(skip)) {
      while (--skip > 0) {
        dest.next('tr').remove();
      }
    }
    dest.replaceWith(buffer);
  }
}
```

The function above supposes each HTML *tr* row element holds a unique *data-id* attribute which must be present in the request result.

### Ajax XML table protocol

> In: lib/management.js
>
> NOTE: DEPRECATED

This protocol returns an XML representation of the new table row or of the table row to replace.

For ease of use it is recommended that the responseXML is a simple flat fragment containing data in first level children elements which is simple to parse with jQuery. One of these elements must contain a unique key to locate each item row. In the following examples this is done with the *Value* (XML fragment) and *data-remote* pair (descendant of the host *tr* element) but you can use any names.

The client side implementation for new item creation can be done with a function similar to the function below :

```javascript
  function reportRemoteCreate(responseXML) {
    var id = $('Value', responseXML).text(),
        label = $('Name', responseXML).text(),
        contact = $('Contact', responseXML).text()
        realm = $('Realm', responseXML).text()
    $('table[name="users"]').find('tbody > tr:first').before('<tr><td><span class="fn">'+ contact +'</span></td><td>'+ realm +'</td><td><a><span class="rn" data-remote="profiles?key=' + contact + '">'+label+'</span></a></td></tr>');
  }
```

The function simply unmarshalls all the properties from the responseXML fragment and creates a new HTML table row definition that it inserts at the begining of the table.

The client side implementation for item updating can be done with a function similar to the function below :

```javascript
function reportRemoteUpdate(responseXML) {
  var id = $('Value', responseXML).text(),
      label = $('Name', responseXML).text(),
      contact = $('Contact', responseXML).text()
      realm = $('Realm', responseXML).text()
  $("span[data-remote$='" + id + "']").parents('tr').children('td:eq(0)').text(contact)
  $("span[data-remote$='" + id + "']").parents('tr').children('td:eq(1)').text(realm)
  $("span[data-remote$='" + id + "']").text(label).attr('data-remote', 'profiles?key=' + contact);
}
```

You must write one function per type of item and per table, since the function manually unmarshal the different properties required to regenerate the table row.

### Ajax JSON table protocol

> Implemented by: $axel.command.makeTableCommand (in lib/commons.js)
>
> NOTE: preferred method

This protocol uses an *Ajax JSON Oppidum success / error* enveloppe as described above, into which it copies a specific payload content.

The `payload` entry contains predefined elements : 

- the `Table` property identifies the target table. 
- the `Action` property specifies how to update the rows, either replacing the existing one (*create* value) or adding to the existing one (*update* value). 
- the rows are available in a `Users` hash property that contains an array. Each row is a hash entry, usually there is one key by column (but they may be more). The hash is interpreted by the Javascript encoding function from which the table command has been built (see the `$axel.command.makeTableCommand` table factory).

Exempe success response (JSON hash) :

```javascript
payload :
  Table : table name
  Action : create, update
  Users : [ array of users rows ]
```

*NOTE*: rows are stored in a *Users* array, this will most probably renamed *Rows* in the future as the table factory component is still under development

