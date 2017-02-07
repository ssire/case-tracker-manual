# Statistics module

The statistics module permits to display a page with a search mask and to query the database to generate graphs and export tables. Its main components are : 

* a page construction service that returns a page containing a search mask and a result view area
* a graph generation service that returns a data set and generates statistical graphs in the result view area
* a table exportation service that returns a data set directly as an HTML table

The search masks are Supergrid formulars with a specific SearchMask extension. 

The graph generation service queries the database and returns the data set using a simple JSON Ajax protocol. The data set is then processed client-side to generate the graphs and some short HTML summary tables in the result view area. Charts are pre-embedded into a statistics page (Case, Activity or KPI statistics for instance) using HTML microformat attributes pre-generated in the HTML result view area. The SVG graphs and the tables are dynamically generated with Javascript code on reception of the JSON data set. The final rendering uses the [d3.js](https://d3js.org) and the [C3.js](http://c3js.org) library.

The table exports are directly generated as HTML tables server-side.

The graphs and the export tables are declared in a `stats.xml` file which must be deployed to the application configuration collection.

## Manifest

### File hierarchy

The statistics module is still under development and actually it is more a scaffold module that you must edit to create your own statistics that depend on your application database model.

* `modules/stats` 
  * `stats.xml` : XML file where you define your statistics pages, graphs and table exportations, to be customized and copie to the application configuration collection
  * `stats.xqm` : shared utility functions
  * `form.xql` : search mask formulars input selector generation
  * `stats.xql` and `stats.xsl` : page construction service
  * `filter.xql` : graph generation service, server-side graph JSON protocol
  * `export.xql` : table exportation service, actually direct HTML tables generation
* `resources` : client side graph generation code
  * `lib/stats.js` and `css/stats.css` : the `stats`command (client side ajax protocol, graph and short tables generation), you actually need to customize that file for you data sets
  * `d3` : d3.js dependency 
  * `c3` : c3.js dependency 
* `formulars`
  * `stats.xml` : common search masks components, useful if you have mulitple statistics pages with common groups of criteria
  * `stats-XXX.xml` : a Supergrid search mask definition
  * `_register.xml` : do not forget to register your search masks so that Supergrid can generate the corresponding mesh files in the application mesh collection
* `modules/formulars/search-mask.xsl` : SearMask element extension to Supergrid

### Oppidum mapping

The typical way to map the statistics module to REST resources to show statistics about *cases* is shown below :

```xml
  <item name="stats" supported="filter export">
    <access>
      <rule action="GET POST filter export" role="g:users" message="registered user"/>
    </access>
    <item name="cases" epilogue="home">
      <model src="modules/stats/stats.xql"/>
      <view src="modules/stats/stats.xsl"/>
    </item>
    <action name="filter">
      <model src="modules/stats/filter.xql"/>
    </action>
    <action name="export">
      <model src="modules/stats/export.xql"/>
    </action>
  </item>
```

And of course you MUST have mapped the [Supergrid formular generator](./supergrid-use.md) too for generating the templates corresponding to your search masks.

Note that fine grain access control can be declared in `stats.xml` using the statistics markup language, so Oppidum level access control is there only to block guest access.

This defines 3 resources with their corresponding pipelines :

* the `GET stats/cases` pipeline generates the statistics page that must contain the pre-generated graph micro-format instructions for each graph and include the Javascript client-side part of the graph generation service 
* the `POST stats/filter` pipeline is the server-side part of the graph generation service, it returns a JSON data set
* the `POST stats/export` pipeline is the table exportation service, it returns an HTML page with tables

Both the graph generation and the table exportation services take XML submitted data as input. The submitted root element tag name serves as a key to identify the type of data set (e.g. Cases or Activities or KPI).

The search masks are available in the `formulars` folder of the application, together with the other document XML formular specifications since they use the same Supergrid generator. The root element name of the formular XML output model must match the key identifying the data set.

The table export currently directly generates HTML tables with a short embedded Javascript function to expand the labels to lower the bandwidth required. *This should be replaced with a JSON table export in the future.*

## Statistics markup language

This section describes syntax used in the `stats.xml` file. 

### Document skeleton

A formular  specification skeleton is presented below :

```xml
<Statistics>
  <Filters>
    <Filter Page="key">
      <Formular>
        <Template>
        1 or more <Command>
      </Formular>
      <Charts>
        1 or more <Chart>
      </Charts>
    </Filter>
  </Filters>
  <Tables>
    1 or more <Table>
  </Tables>
</Statistics>
```

The document contain two main section: 

* the `Filters` section contains one `Filter` element for each statistics page of the application
* the `Tables` section contains one `Table` element for each export table of the application

### The `Filter` element

The `Filter` elements defines a statistics page with a search mask and a pre-configured result view area.

The mandatory `@Page` attribute is the key used to find the page definition in the controllers. It usually matches the Oppidum resource mapping of the page.

The mandatory `Formular` element defines the search mask to load into the page. The `Template` element contains the URL of the formular to load. The `Command` elements define the available commands. There is usually one command to call the graph generation service, and zero or more commands to export tables in new windows.

The `Charts` element defines one `Chart` element per chart to draw in the result view area.

#### The `Command` element

The `Command` element defines the statistics commands: usually one or more table export commands and a single statistics command to render the graphs.

The `@Allow` attribute defines the categories of users that can run the command. It uses the role token language.

The `@Name` attribute ...

The `@Action` and the `@Form` attributes ...

The `@ExcelAllow` attribute ...

The `@W` attribute sets the button width in grid units.

Example : 

```xml
<Formular>
  <Template>../templates/stats-cases</Template>
  <Command Allow="g:admin-system g:business-intelligence g:region-manager g:kam" Name="submit" Action="export?t=anonymized" Form="e1" W="3" Offset="1">Anonymized Exportation</Command>
  <Command Allow="g:admin-system g:region-manager g:kam" Name="submit" Action="export?t=all" Form="e2" W="2">All Exportation</Command>
  <Command Allow="g:admin-system g:region-manager g:kam" ExcelAllow="g:admin-system" Name="submit" Action="export?t=list" Form="e3" W="2">Contact List</Command>
  <Command Allow="g:admin-system g:business-intelligence g:region-manager g:kam" Name="stats" Action="filter" W="2">Statistics</Command>
</Formular>
```

This example loads the *../templates/stats-cases* formular into the search mask with 4 command buttons: *Anonymized Exportation*, *All Exportation*, *Contact List* and *Statistics*. The *All Exportation* command is only available to system administrators, region managers, or managers of the cases. The data set scope restriction (so that for instance a region manager can only see cases from his region) is directly implemented in the XQuery scripts (`form.xql` to limit the options available, `filter.xql` and `export.xql` to limit the samples included into the set).

#### The `Chart` element

The `Chart` element defines a graphical chart. 

Actually the module only provide a frequency bar diagram definition, but this can be easily extended.

Example :

```xml
<Chart>
  <Layout>
    <Angle>75</Angle>
    <Bottom>120</Bottom>
    <Left>20</Left>
  </Layout>
  <Set>Cases</Set>
  <Title>Country</Title>
  <Variable Selector="Countries">Co</Variable>
</Chart>
```

The variable definition can use either a `Variable` or a `Vector` element.

The `Variable` defines a single value variable like the year of creation of a company, or the sex of a person. When doing frequency statistics a sample can only contribute to one value.

The `Vector` defines a multiple values variable like the *Sources of business innovation ideas* in the innovation tree model. When doing frequency statistics a sample can contribute to several values.

You must specify the variable domain. If the domain corresponds to a data type definition in the global information collection you can use the `@Selector` attribute. For other special domains you can use the `@Domain` attribute and use a specific domain key that can be used by the statistics module to interpret the potentially unbounded domain (e.g. *year* is a domain for a variable representing a year, *kam* is a domain representing all the KAM users of the application).

The example graph displays the frequency bar diagram by country for a set of cases. Each bar legend is rotated with a 75 degrees angle from the horizontal, is shifted 120 pixels to the bottom and 20 pixels to the left.

![Country bar diagram sample](../images/statistics/country.png "Country bar diagram sample")

### The `Table` element

The `Table` element defines a table export

Example : 

```xml
<Table Type="list" Page="cases">
  <Headers Lang="en">
    <Header BG="none">Case ID</Header>
    <Header BG="none">Project ID</Header>
    <Header BG="case">Call cut-off</Header>
    <Header BG="case">Phase</Header>
    <Header BG="case">Project acronym</Header>
    <Header BG="enterprise">SME beneficiary</Header>
    <Header BG="enterprise">Contact person</Header>
    <Header BG="enterprise">Country</Header>
    <Header BG="enterprise">Size</Header>
    <Header BG="enterprise">Planned life cycle</Header>
    <Header BG="case">Project officer</Header>
    <Header BG="case">EEN</Header>
    <Header BG="case">EEN KAM Coordinator</Header>
    <Header BG="case">KAM</Header>
    <Header BG="case">Case status</Header>
  </Headers>
</Table>
```
This example exports a table with 15 columns with different background colors as specified by the `@BG` attribute (note that actually colors are hard-coded into the `modules/stats/export.xql` controller). That table corresponds to the export command with the *list* type parameter as coded in the HTTP request query parameter *t* (see above in *A search mask definition* the *Contact list* command).

![Table export sample](../images/statistics/export.png "Table export sample")

## Graph rendering service

The graph rendering service is installed as a `stats` Javascript command generated from one of the `Command` elements in the `Filter` that defines the statistics page. 

### The `stats` command

> In: resources/lib/stats.js

The `stats` command uses several micro-format attributes to find the search mask into the page and to find the address of the JSON graph rendering service.

Example:
```html
<button id="c-stats-submit" class="btn btn-primary" 
  data-command="stats" data-target="editor" data-src="stats/filter"
  data-width="750">Statistics</button>
```

The mandatory `data-src` attribute specifies the URL of the graph rendering service.

The mandatory `data-target` attribute specifies the id of the search mask formular.

The optional `data-width` attribute specifies the width of the invidiual graphs that will be generated into the page.

The `stats` command plots graphs which are defined by specific micro-format instructions pre-generated into the page. These instructions are generated by the `stats.xsl` view which has been used to generate the page from the page model defined in `stats.xml`.

The `data-variable` attribute defines a frequency diagram for a given variable. The text value is the name of the variable to plot. That variable must be a key entry of the data set array.

The `data-set` variable gives the name of the data set array to use for plotting the variable. This implies the JSON response is a hash with an entry of the same name that contains an array of hash, each one representing a sample in the data set.

For instance with `data-set="Coaches"`, the `data-variable="Co"` will plot the Coaches[*i*].Co frequency diagram for every coach *i* in the data set.

Example:
```html
<div data-set="Coaches" class="chart" style="display:none" data-angle="75" data-bottom="120" data-left="20" data-variable="Co">
    <h3>Country</h3>
    <table class="stats">
        <thead>
            <th>Country</th>
            <th>Number</th>
            <th>%</th>
        </thead>
        <tbody/>
    </table>
    <div class="export noprint">Export <a class="export" href="#" download="case-tracker-countries.xls">excel</a>
        <a class="export" href="#" download="case-tracker-countries.csv">csv</a>
    </div>
</div>
```
### The  JSON response protocol

The JSON statistics response protocol defines the data format used by the graph rendering service to return a data set ready for rendering client-side. That reponse is generated in the `filter.xql` file.

The response can contain the following top level entries : 

* the **Sizes** entry must contain the number of data sampples
* one entry should contain an array of data samples, each data sample being a hash of multiple variables, if the data set contains only one hash it should be stored inside a singleton array, the case tracker uses a **Cases** entry (or an **Activities** entry), but this can be renamed to whatever data sample type you are plotting (see `stats.js`)
* the **Variables** is a hash containing one key for each variable to plot in a data sample, each variable is itself represented with a hash with two keys (e.g. `{"Sx":{"Labels":["Male","Female"],"Values":["M","F"]}}`):
   * the **Values** key must contain an array with all the values in the domain of the variable
   * the **Labels** key must contain an array of all the textual names values in the domain of the variable
   * of course both the *Values* and *Labels* arrays must have exactly the same length and be aligned, they are used to decode the variables and print their labels in the graphs legends
* the optional **Graphs** entry must be present if you are using C3 for rendering (i.e. `data-library="C3"`), it must contain a hash for each data sample variable to plot containing a JSON copy of the `Graph` element from the corresponding graph definition in the `stats.xml` file, this is used to generate the graph configuration passed to the C3 library

## Supergrid `SearchMask` extension

The statistics module extends Supergrid vocabulary with a `SearchMask` element. That extension is implemented by the `modules/formulars/search-mask.xsl` transformation.

The `SearchMask` element allows to present groups of search crieria fields into tables. It also allows to include definitions from `Component` elements defined in external formular definition files. This is useful to share criteria between several statistics pages.


