# 0010: React for paginated views

- Authors: Oliver Roick
- Status: Draft Proposal, 2017-08-31
- Backlog Issues: [#1465](https://github.com/Cadasta/cadasta-platform/issues/1465), [#1466](https://github.com/Cadasta/cadasta-platform/issues/1466)

## Functional Requirements

### Problem statement

Large lists of objects (organizations, projects, parties or resources) should be paginated. We currently do that by loading the whole list of objects into the web page and then using [DataTables](https://datatables.net/) to create paginated views on client-side. This approach affects the performance of these views since the whole data set needs to be loaded from the database, rendered into the page and then transferred to the client. 

To address this, we experimented with an unorthodox approach where we render HTML snippets on the server side, retrieve the content using asynchronous requests and add it to the page. This approach gave us the performance improvements we wanted, but it added a large amount of additional code. It adds complexity and duplicates functionality that we need to maintain (both for the website and the API). 

This decision record suggests a modern approach to achieve improved performance of the paginated pages while also keeping the amount of additional code to a minimum. 

## Context/Use cases

Use cases are unchanged from existing platform functionality. No additional use cases are covered by this Decision Record.

## Description

We aim to replace the pagination functionality of the following views using the approach discussed in this Decision Record. 

- `organization.views.default.OrganizationList `
- `organization.views.default.OrganizationDashboard`
- `organization.views.default.OrganizationMembers`
- `organization.views.default.UserList`
- `organization.views.default.ProjectList`
- `party.views.default.PartyList`
- `party.views.default.PartyDetail` (Relationships and Resources tabs)
- `party.views.default.PartyResourcesAdd`
- `party.views.default.PartyRelationshipDetail` (Resources area)
- `party.views.default.PartyRelationshipResourceAdd`
- `resources.views.default.ProjectResources`
- `resources.views.default.ProjectResourcesAdd`
- `resources.views.default.ProjectResourcesDetail` (Attached to area)
- `resources.views.default.LocationDetail` (Relationships and Resources tabs)
- `resources.views.default.LocationResourceAdd`

**Note:** This decision record focusses mostly on how React will be integrated into our current architecture and less on implementation details of paginated tables itself. I kept the examples in this DR simple on purpose to provide an higher-level overview of how we can use React to solve this problem. The concepts discussed in this DR can also be applied to other problems, where we're dealing with highly interactive scenarios. 

## Technical Specification

### UI Changes

No UI changes are foreseen as the objective is to transparently replace datatables.

### Overview of the suggested solution

Instead of building specific endpoints that serve rendered HTML snippets, we will leverage our existing JSON API for these use cases. Rendering will be implemented into the client-side using the JavaScript framework [React](https://facebook.github.io/react/). 

### Analysis of the alternatives

#### Alternative approaches

##### Rendering HTML snippets

Rendering HTML snippets on the server-side and adding them into the website using asynchronous HTTP requests, has proven itself too cumbersome. We have to add a significant amount of code to both the back-end and the front-end, which increases the complexity of the application and the amount of work we have to put in to maintain the software.

##### Using Django pagination

Paginated pages could also be implemented in a standard Django approach: Render static HTML pages with a subset of all records. We would store information on the current view (or page) as query parameters with the URL and would read that information when processing the request. While this approach is of very low technical risk and will increase performance, we would lose the interactivity that we gain from a JavaScript-based approach. Also for some pages (for example listing resources of locations or relationships), this would add a significant amount of additional page requests since we would have to load high-volume map pages over and over again. 

#### Alternative JavaScript frameworks

React is a state-of-the-art JavaScript framework that is used widely. There are various options (Angular, Ember, Vue), which all do not provide any significant advantages over React. React, however, only provides the rendering layer for web applications, so its footprint is limited. It further allows us to build client-side components, which we can reuse later, should we decide to move full single-page app. 

### Architecture description

#### API

The API does not require many changes. 

We already support pagination and we will be able to request specific pages at varying length using `limit` and `offset` query parameters. 

We also support basic search and sorting sorting, which is sufficient to implement client-side searchable and sortable lists. 

Aside from that, we might have to add smaller changes here and there to fully support the requirements for each of the paginated views.

#### Client-side extensions

We will implement a set of React components that will allow us to compose paginated tables that mirror the functionality currently achieved using DataTables.

The React component `PaginatedTable` will be use across all paginated views and will provide the core functionality including search, pagination navigation, setting the page size and column sorting. `PaginatedTable` will be the root component and will host the store and all event handlers. Functionality listed above will be implemented as separate React components. 

##### Configuring `PaginatedTable`

`PaginatedTable` is configurable. Each paginated view has a different API source (where we load the data from), number of columns and template for how each row in the table should be rendered. 

The following properties must be set when initialising `PaginatedTable`:

| Property         | Type              | Description   | 
| ---------------- | ----------------- | ------------- | 
| `url`            | `String`          | The API URL where data is loaded from. | 
| `columns `       | `Array`           | A list of column definitions. The number of elements defines the number of columns in the table. Each element must define the following keys: <br>`label` (the label used in the table head for the column)<br>`field` (the model field, used to apply sorting)<br>`sortable` (identifies if the column is sortable) keys. | 
| `row_component ` | `React.Component` | A React component that is used to render individual rows into the table. This is necessary because so tables (such as resources) require more complex rendering of individual cells. | 

##### Example use

```html
<!-- Load React libs -->
<script crossorigin src="https://unpkg.com/react@15/dist/react.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@15/dist/react-dom.min.js"></script>

<!-- Load the PaginateTable core component -->
<script src="/static/dist/PaginatedTable.min.js"></script>

<!-- Load the row component used in this view -->
<script src="/static/dist/PartyRow.min.js"></script>
<script type="text/javascript">
  var props = {
    "url": "/api/v1/resources",
    "columns": [
      {"label": "Party", "field": "name", "sortable": true},
      {"label": "Type", "field": "type", "sortable": true}
    ],
    "row_component": Row
  }
  ReactDOM.render(
    React.createElement(PaginatedTable, props),
    document.getElementById('paginated-table')
  );
</script>
```

A very basic implementation can be found in [branch `react-party`](https://github.com/Cadasta/cadasta-platform/compare/react-party).

### Database changes

N/A

### Security considerations

N/A

### Scalability

This approach will improve scalability of the platform because:

- Only data that is being displayed will be transferred. 
- JSON payload will have a smaller footprint than HTML.
- We will leverage database functions to slice the dataset into subsets for each page. These operations are highly optimised.

### Maintainability

#### Package management

We will use [yarn](https://yarnpkg.com) for package management. Yarn provides some advantages over npm:

- It is faster.
- Builds fail less often when a package can't be installed. It retries the failing request instead. 
- It is deterministic. The same libraries are installed across all systems; with npm you sometimes had different versions of the same library installed depending on the install order and the whether other libraries depend on the same library but a different version. 

#### Task runner

We will use [Grunt](https://gruntjs.com/) as our task runner. Grunt will be used to create builds, run tests and lint the code.  

#### Builds

The build compiles from `jsx` to JavaScript and minifies that code. The build script should be run when the test server is started in the dev VM (including a watch script so sources are re-built whenever a file is changed) and when deploying to production. 

We will run individual builds for each component to keep the size of resources small. For example, there will be one resource for paginated pages, another one for WMS, etc.

The command to create builds will be `grunt build`.

#### Tests

We will start to unit-test front-end components. We are going to use [Jest](https://facebook.github.io/jest/) for our test framework and [Enzyme](http://airbnb.io/enzyme/) to make it easier to assert and traverse React components.

The Travis build will have an additional test environment that runs all unit tests for client-side components. 

The command to run tests will be `grunt test`.

#### Code linting

We will the client-side code using [eslint](http://eslint.org/) to ensure a minimum standard code quality. Linting will be run as a separate Grunt task.

The Travis build will have an additional test environment that runs eslint on all client-side components.

The command to run code linting will be `grunt lint`.

#### Monitoring dependencies 

We use [Greenkeeper](https://greenkeeper.io/) to monitor the status of npm dependencies.  It works similar to requires.io, whenever dependencies are upgraded we recieve a pull request ([Here's an example](https://github.com/oliverroick/Leaflet.Deflate/pull/12/files)).

### Costs considerations

There should be no additional costs. Greenkeeper is free for open-source projects. 

### Pre-requisites

None. 

### Defintion of Done

- All paginated views are replace with the approach described in this decision record.
- All test pass. 
- The functionality is verified during QA.
- All references to datatables have been removed from the code. 