# Case Tracker software documentation

This is the documentation about the case tracker pilote application by Oppidoc

The case tracker pilote is written and maintained by Stéphane Sire (<s.sire@oppidoc.fr>)

The case tracker pilote is a reference application which can be customized to create mutiple types of case management applications with user-driven and data-driven workflows :

- user-driven because the workflow status can be directly updated by end-users depending on their roles
- data-driven because the workflow status can be automatically updated depending on the data submitted into semi-structured documents

The easiest way to create a case management application from the case tracker is to fork it and start a new version (*source code depot will be available soon*).

The case tracker pilote is also a workbench for a new programming architecture that we call XML full-stack programming. It also has some flavors of the specification driven programming style.

> This is a work in progress, initiated for the Platinn coaching application in Switzerland in 2013/2014, and continued under the auspices of the Coachcom 2020 project from september 2014 to november 2016. Coachcom is a coordination and support action (H2020-635518; 09/2014-08/2016) selected by the European Commission to develop a framework for the business innovation coaching offered to the beneficiaries of the Horizon 2020 SME Instrument.

The Case Tracker actually runs on the [eXist-DB database](http://exist-db.org) on Linux and Mac OS X computers, thus it can be deployed on any cloud infrastructure supporting Java. It is written in XQuery, XSLT and Javascript languages and runs with the [Oppidum](https://github.com/ssire/oppidum) XQuery framework, [AXEL](https://github.com/ssire/axel) and [AXEL-FORMS](https://github.com/ssire/axel-forms) Javascript editing library.

## Main components

You can start with the [grand tour](./doc/tour.md) to get an overiew of the application features before going farther.

The main Case Tracker components can be use to build case management applications with semi-structured data entry. You may also pick-up some of the Case Tracker components that you can use independently from the others to build more specific applications.

**Overview**

- [Architecture](./doc/architecture.md) :  Architecture and conventions
- [Ajax protocols](./doc/ajax.md) :  Ajax protocols

**Server-side components**

- [Workflow](./doc/workflow.md) : user-driven and data-driven workflows
- [Supergrid](./doc/supergrid.md) : grid-base form generator
- Alerts : todo list generation and notifications
- Stats : statistics and graphs rendering
- [Services](./doc/services.md) :  library for services integration
- Media : template-driven media engine that currently generates and sends e-mail
- Users : users management

The other functionalities not in that list like search or data exportation tools are natively implemented as specific XQuery/XSLT scripts.

**Client-side components**

The client-side components encapsulate each front-end functionality, mainly client-server communication (Ajax protocol) and user interaction, into javascript objects (mainly commands, but also bindings, plugins, filters) which can be inserted into the application page using specific _data-*_ (aka micro-format instructions) attributes. This way it is easy to develop and maintain Javascript code on one side and to instantiate it from XSLT transformations that generate the views of the application.

- [Commands](./doc/commands.md) :  Javascript commands to communicate with the server
- [UI components](./doc/components.md) :  UI components (aka *web components*)

