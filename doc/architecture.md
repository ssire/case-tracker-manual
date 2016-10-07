# Architecture and conventions

Please take the [grand tour](./tour.md) first to get an overiew of the application features before reading that chapter.

## User interface overview

The application landing page upon successful login is called the *stage* (or the dashboard). It displays a search mask and a results list to retrieve and navigate into a case and/or activity. It is structured into three vertically laid down parts :

* a top primary navigation bar (or application menu) with drop down menus
* a search mask
* a result table

The application also comprises transversal search pages like search for persons, enterprises or regions. They have a similar layout. Statistics pages too : the search masks allows to select a case and/or activity sample set and the results area displays several tables and associated graphs.

The case and activity workflow view are more specific. They show a current workflow status on a time line widget at the top (just below the application menu), and a list of documents on an accordion widget some of which can be edited in place depending of the worklow status and user's rights.

Some listing pages like the todo lists or data export pages, do not have a specific layout, they just show tables or summaris for printing or navigating into cases and/or activities.

Finally a management page groups on a vertical tab below the application menu several management functionalities like user's account management to create user's login and renew passwords.

The navigation structure is implemented both in mesh files (inside the *mesh* folder) and in specific functions of the *epilogue.xql* file according to Oppidum conventions.

## Programming style

### Patterns

The Case Tracker source code implements a limited set of patterns and constructs : 

1. XML configuration files (see next section) that describe application behavior
2. dynamical formulars pre-generated with [Supergrid](./doc/supergrid.md) embedded inside [user interface widgets](./doc/components.md)
3. client-side [Commands](./doc/commands.md) hosted on [user interface widgets](./doc/components.md)
4. client-server interactions with simple [Ajax protocols](./doc/ajax.md) (XML or JSON)
5. CRUD controllers to update application entities into the database (including implementing functional dependencies or splitting of entities into multiple database locations)

Points 2. and 3. use HTML micro-format attributes (e.g. *data-command="save"*) for associating formulars and commands with user-interface components.

### Specification driven

The Case Tracker follows a *specification driven programming* style. Core programming involves using or inventing small pieces of developer-friendly XML vocabularies (or *Domain Specific Languages* - DSL). For instance specific vocabularies describe most of the workflow transitions and access control rules, and user interface components of the application. Thus a huge part of the application source code and librairies implements XML vocabularies interpretors and generators. It follows that an important part of the programming effort is to write XML configuration files that directly translate business specifications and requirements. 

In summary *application programming* is splitted into editing XML configuration files and creating some XQuery controllers to update the database content.

The two tables below show different configuration files in use by the Case Tracker and their location in the file system. All these files are copied to the application config collection in the database (`/db/www/{app}/config`) by the deployment script. 

The Oppidum framework itself comes with a set of XML vocabularies. They are listed in the table below.

| File                   | Location                | Goal |
|:-----------------------|:------------------------|:-----|
| mapping.xml            | config                  | application REST mapping |
| modules.xml            | config                  | application REST mapping modules |
| skin.xml               | config                  | application CSS and Javascript dependencies |
| dictionary.xml         | config                  | application localisation strings |
| errors.xml             | config                  | application error messages |
| messages.xml           | config                  | application information messages |

Then the Case Tracker application adds more XML vocabularies for higher level functionalities. They are listed in the table below.

| File                   | Location                | Goal |
|:-----------------------|:------------------------|:-----|
| application.xml        | config                  | workflows, document access rights |
| services.xml           | config                  | inter-applications communication |
| settings.xml           | config                  | application global parameters |
| variables.xml          | config                  | variables definition for e-mail templates |
| email.xml              | data/global-information | e-mail templates  |
| checks.xml             | modules/alerts          | todo list computation and reminder alerts |
| stats.xml              | modules/stats           | masks, graphs and export table for statistics |
| global-information.xml | data/global-information | data types definition for selectos (drop-down lists, etc.)  |
| proxies.xml            | config                  | data model functional dependencies |
| _register.xml          | formulars               | list of formulars |
| {name}.xml             | formulars/{name}.xml    | supergrid document formular {name} definitions |

## Coding conventions

The code is divided between server-side code, XML application configuration files and client-side code all stored in the git depot.

The code layout at the 1st-level folder is a direct application of the coding conventions as summarized in the table below.

| 1st-level folder | Description |
|:-----------------|:------------|
| build            | configuration files to build external dependencies (e.g. AXEL library) |
| caches           | files to bootstrap caches |
| config           | configuration files to be loaded into config collection |
| data             | data files to be loaded into global-information collection |
| debug            | files to bootstrap in-database tracing (debug.xml, login.xml) using debug collection |
| formulars        | search mask and/or document formular XML definitions |
| indexes          | collection.xconf files to create DB indexes |
| lib              | shared XQuery or XSLT modules |
| mesh             | application mesh files (equivalent to application page templates) |
| models           | shared or generic XQuery models |
| modules          | parent folder for CRUD controllers and resources usually grouped in sub-folders by entity or functionality |
| resources        | parent folder for static resources (CSS, js, etc.) |
| scripts          | installation script (bootstrap, deployment, etc.) |
| templates        | XTiger templates to be loaded into mesh collection (not much used since most XTiger templates are generated from formular definitions by supergrid) |
| test             | test files |
| untracked        | static files not under version control served by the application  |
| views            | shared or generic XSLT views |

### Server-side code

The server-side code is divided into shared libraries of XSLT transformation rules and shared XQuery modules (both in first level `lib` folder), generic Oppidum XSLT views (in first level `views` folder) and generic Oppidum XQuery models (in first level `models` folder), and specific modules. Note that a generic Oppidum view or model is usually a pipeline component which can be reused at the file granularity to render several application pages or between applications (by cut-and-paste).

The specific modules can be divided into *entity modules* and *functional modules*. 

The entity modules group together XSLT views and XQuery CRUD controllers inside a sub-folder of the `modules` folder. Usually the sub-folder is named after the managed entity (e.g. `modules/cases` to handle Case entities, or `modules/persons` to hande Person entities).

The functional modules group together XSLT views and XQuery controllers to manage a set of related functionalities inside a sub-folder of the `modules` folder. Usually the sub-folder is named after the functionality (e.g. `modules/stats` to handle statistics, or `modules/management` to hande diverse management functionalities like user's accounts). 

### XML application configuration

The XML application configuration files can be divided into system configuration and data configuration. System configuration is more about defining the behavior of the application (e.g. workflow transitions, rules for alert notifications) and environmental settings (e.g. SMTP server address), whereas data configuration is more about defining data used to create user generated content or to display to the user (e.g. data types definitions, e-mail templates content, localized labels, etc.).

The system configuation is mainly available in the `config` folder, whereas data configuration is available in the `data` folder. Some configuration files may also be scattered into functional modules (e.g. `modules/stats/stats.xml`). The specification of formulars, which can be considered in-between system and data configuration, is stored inside the first level `formulars` folder.

### Client-side code

The client-side code is stored inside the `resources` folder. It contains one sub-folder for each major third party library it depends on (e.g. bootstrap, d3, etc.). The only exceptions are the AXEL and AXEL-FORMS dependencies which are copied inside the generic `resources/lib` sub-folder which is detailed below.

The `resources/lib` folder contains the Javascript code of the application. Some files are named after a functional module they implement (e.g. `stats.js` that implements client-side `modules/stats`), or after a set of transverse functionalities they implement (e.g. `search.js` for `module/persons` or `modules/regions` search). Most of the other files implement the case tracker specific AXEL and AXEL-FORMS widgets extensions (i.e. plugins, filters, bindings or commands javascript objets, see [AXEL](https://github.com/ssire/**axel**) and [AXEL-FORMS](https://github.com/ssire/**axel-forms**) documentation).

The `resources/css` and the `resources/images` sub-folders store respectively the case tracker CSS and image files.

The boostrap forms adaptations and specific forms CSS rules are defined in `resources/css/forms.css`.

The mapping bewteen the resource files and the application pages is declared in the `config/skin.xml` file.

Note that per Oppidum construction and per NGINX configuration the content of the first level resources folder is not served by eXist-DB but by NGINX.

## Data model

### Database implementation

Collections

### REST interface

URLs


