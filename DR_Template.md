# DR Template: This is the template to be used for writing DRs

- **Author:** David Palomino
- **Status:** Submitted, 2017-06-26
- **Backlog issue:** Here we'll include a link to the github issue included in the backlog with the initial requirements

## Functional Specification 

In addition to the information included in the backlog issue, we will provide a broader description of all the functionality needed. Here we will include things like: 
- Expected user behavior for all cases
- Error messages
- How to handle corner cases

In the GitHub issue the following information should have been provided before the creation of the DR: 
- **Problem Statement:** Describing the use case/need this try to cover, and the background information behind it.
- **Context/Use Case:** Providing some examples
- **User Stories:** Description of the feature following the syntax “As a… I want to…”. Sometimes is not only a single User Story but a few of the marked with checkboxes. For the features that involved like 3-4 USs we will handle them using a single GitHub Issue in the backlog. For the big topics, we will be creating epics. 
- **Description:** Detailed description including technical details related to the feature. 
- **Link to DR:** Link to the associated DR (when applicable).

## Technical Specification 

### UI Changes

- **UI Changes involved:** Yes/No
- Link to last updated wireframes when necessary. Note that wireframes could be made for a bigger context. If that is the case, we will highlight the areas of the wireframes (pages for instance) that apply to the feature. 


### Overview of the suggested solution 

High level overview of the selected solution/implementation. 


### Analysis of the alternatives

Summary of the different alternatives that have been considered during the research phase, including the reasons for selecting or not selecting each of them. Providing links to mockups, examples, etc could be useful. 


### Architecture description 

Architecture design details for the implementation selected. 


### Database changes

Description of database changes needed (if any), including migrations. 


### Security considerations

Analysis of the security implications, when needed. 


### Scalability 

Please, provide an analysis regarding the scalability of the proposed solution. Try to answer questions like: 
- Is the proposed solution efficient for projects with lots of records or resources? How many? 
- Is this working with a large number of users? 
- Latency or throughput considerations? 


### Maintainability 

Include here all maintainability aspects that need to be considered. 


### Costs considerations

If the suggested solutions has some costs implications. For instance, AWS costs, licensing, etc. 


### Pre-requisites

Are there any tasks that should be implemented before? If so, please provide links to appropriate DRs and/or GitHub issues. 


### Defintion of Done

Detailed description of tests to be successfully executed for the feature being considered done. This information should be also moved to the [Manual Test Cases spreadsheet](https://docs.google.com/spreadsheets/d/1JmKVAmdQgjdzg8YhrSDtfBtWckkv7aq4-ZIZCmwbbwU/edit#gid=0). 
