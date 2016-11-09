# Glossary

**Application configuration**

The application configuration is the set of XML resources deployed inside the `config` collection of the application. Most of them are copied from the *config* folder of the depot but some others are directly copied from the module code folder they configure.

**Application data collection**

The application data collection is the collection that contains all the user generated content in the database. By convention this is the `/db/sites/cctracker` collection where *cctracker* is the project name (see below).

**Application configuration collection**

The application configuration collection is the collection that contains most of the application configuration and source code files in database. By convention this is the `/db/www/cctracker` collection where *cctracker* is the project name (see below). Actually it contains only few source code files since most of the code is executed from the file system. One notable exception is all the code for scheduled jobs which must be stored in database per eXist-DB construction.

**Binding** (AXEL client-side Javascript object)

A binding is a Javascript object implementing a binding in the AXEL Javascript library. Most bindings are used to constrain the user input on one or more fields they are attached to with a *data-binding* microformat instruction. Supergrid allows to configure some bindings directly on the XML formular specification.

**Case resource**

The case resource is the document that contains all the XML model for a case, including its activities. By convention the case resource is a `case.xml` file stored inside a `cases/YYYY/MM` collection inside the application data collection. *YYYY* and *MM* correspond respectively to the year and the month of a significant date of case history (e.g. case creation date, case call date, etc.). 

**Command** (AXEL client-side Javascript object)

A command is a Javascript object implementing a command in the AXEL Javascript library. Most commands execute on behalf of user's actions in the browser. They are executed by user interaction on an HTML host element on which they are attached with a *data-command* microformat instruction. Most of the time the command does something with the content of an editor (e.g. an XTiger XML formular generated with Supergrid) like saving data to or loading data from the server.

**eXist-DB**

[eXist-DB](http://exist-db.org) is the native XML database for running a case tracker application. The project is maintained on [github](https://github.com/exist-db/exist/).

**Field handle**

Formular templates are turned into in-page editor by the AXEL and AXEL-FORMS libraries. For each primitive input field associated to a plugin, the plugin generates an HTML fragment implementing the input widget, the field handle is the root of this fragment.

**Host element**

The HTML element that serves as a host for a binding or a command. The link is declared as a *data-binding* or *data-command* attribute directly on the host element.

**Plugin** (AXEL client-side Javascript object)

A plugin is a Javascrip object implementing an editing field in the AXEL Javascript library.

**Project name**

The project name is the abbreviated application name. By convention it is used to name the file system folder of the application, and to name several database collection (e.g. application data collection and application configuration collection).

**Roles specification micro-language**

The roles specification micro-language is a simple text expression that defines a set of users by a whitespace separated list of tokens. A roles expression is usually embedded inside a higher level language (e.g. access control rules specification language).

The expression contains a whitespace separated list of **role tokens** (e.g. "g:admin-system", "r:coach"). Each role token starts with a scope letter followed by a colon and by a role name. The scopes are :

* `g` for global scope (or static role) : the targeted users are independent from the database content (e.g. a system administator, a coaching assistant, etc.)
* `r` for relative scope (or dynamical role) : the targed users depends on a current case or activity context (e.g. the region manager for a case or the coach for an activity)

**Access control rules specification micro-language**

The access control rules specification micro-language is a simple language that defines a set of users authorized to perform some actions or target of some actions. That language actually defines only a `Meet` element whose text value uses the *Roles specification micro-language*. The `Meet` elements includes in the set all of the users targeted by the roles specification in its textual content. The actions depend of the parent of the access control rules element(s). Some actions may impose restrictions on the available roles scopes (for instance some actions are not specific to a case or activity and thus do not support the relative scope).

**Supergrid**

Supergrid is an XSLT transformation that generates formulars from an XML formular specification. Supergrid also comes with a simulator to test and deploy formulars in different modes (e.g. read-only, update or create modes). The Supergrid XML vocabulary allows to layout form input elements on a grid and to handle several aspects of the XTiger XML template with a higher level language.

**XML formular specification**

Most of the formulars are generated from an XML document called the XML formular specification and written with the Supergrid vocabulary. By convention these documents are in the `formulars` folder of the code depot which is a good starting point to learn the language through examples.

**XTiger XML template**

The formulars loaded into the web browser are written using the [XTiger XML template](http://ssire.github.io/xtiger-xml-spec/) language. Most of the time you do not need to directy write the templates since they are generated from an XML formular specification using the Supergrid formular generator. The templates are loaded and transformed into editors using the AXEL javascript library.

**XSLT Blender**

The XSLT blender is the XSLT transformation that turns database XML documents into XML documents for read-only viewing in a read-only formular. That transformation is required to transform documents that contain fragments created with a rich text input field. These fragment must be transformed in a suitable HTML format for being displayed by the *html* plugin. The XSLT blender is implemented by the `views/blend.xsl` file. By convention XML documents served with the XSLT blender use a resource URL with the *.blend* suffix.

**XML input content**

The XML representation of a document for loading into an editor.

