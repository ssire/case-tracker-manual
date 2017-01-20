# Statistics module

The statistics module allows to query the database with a search mask and to generate graphs and export tables. It offers 3 kinds of services : 

* a statistics page construction service that returns a page containing a search mask and a result view area
* a graph generation service that returns a data set and generates statistical graphs in the result view area
* a table exportation service that returns a data set directly as an HTML table

The search masks are declared using Supergrid language with a specific SearchMask extension. 

The graph generation service queries the database and returns the data set using a simple JSON protocol. The data set is then processed client-side to generate the graphs and some short HTML summary tables in the result view area. Charts are pre-embedded into a statistics page (Case, Activity or KPI statistics for instance) using HTML microformat attributes pre-generated in the HTML result view area. The SVG graphs and the tables are dynamically generated in Javascript on reception of the JSON data set. The final rendering uses the d3.js library.

The table exports are directly generated as HTML tables server-side. 

The statistics specification language defines the graphs and the export tables in a `stats.xml` file. That file must be deployed to the application configuration collection.

## Oppidum mapping

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

This defines 3 resources with their corresponding pipelines :

* the `GET stats/cases` pipeline generates the statistics page that must contain the pre-generated graph micro-format instructions for each graph and include the Javascript client-side part of the graph generation service 
* the `POST stats/filter` pipeline is the server-side part of the graph generation service, it returns a JSON data set
* the `POST stats/export` pipeline is the table exportation service

Both the graph generation and the table exportation services take XML submitted data. They submitted root element tag name serves as a key to identify the type of data set request (e.g. Cases or Activities or KPI).

The search masks are available in the `formulars` folder of the application, together with the other document XML formular specifications since they use the same Supergrid generator. Note that the formular root element name must be matched with the graph generation and graph exportation services.

The statistics configuration file is available in `modules/stats/stats.xml`.

The client-side graph generation is implemented in `resources/lib/stats.js` (and calls the d3.js third party library). 

The table export currently directly generates HTML tables with a short embedded Javascript function to expand the labels to lower the bandwidth required. It should be replaced with a JSON table export in the future.

## Statistics markup language

This section describes syntax used in the `modules/stats/stats.xml` file. 

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

* a `Filters` section that contains one `Filter` element for each statistics page the application offers
* a `Tables` section that contains one `Table` element for each export table the application offers

### The `Filter` element

The `Filter` elements defines a statistics page rendering.

The mandatory `@Page` attribute is the key used to find the definition associated with the resource name in `modules/stats/stats.xql` that constructs the view.

The mandatory `Formular` element defines the search mask. The mandatory unique `Template` element contains the URL of the search mask formular to load. The `Command` elements define the available commands.

The `Charts` element contains one `Chart` element per chart to draw in the result view area.

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

This example loads the *../templates/stats-cases* formular into the search mask with 4 command buttons: *Anonymized Exportation*, *All Exportation*, *Contact List* and *Statistics*. The *All Exportation* command is only available to system administrators, region managers, or managers of the cases.

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

### The graph micro-format language 

This section describes the micro-format instructions generated by the `modules/stats/stats.xsl` when rendering a statistics page. These instructions are then interpreted client-side by the `resources/lib/stats.js` library to compute statistics and render the graphs from a JSON response containing a data-set.

TBD

### The  JSON response protocol

The JSON statistics response protocol defines the data format used by the graph rendering service to return a data set ready for rendering client-side. That reponse is generated in the `modules/stats/filter.xql` file.

TBD

## Supergrid `SearchMask` extension

The statistics module extends Supergrid vocabulary with a `SearchMask` element. That extension is implemented by the `modules/formulars/search-mask.xsl` transformation.

The `SearchMask` element allows to present groups of search crieria fields into tables. It also allows to include definitions from `Component` elements defined in external formular definition files. This is useful to share criteria between several statistics pages.


