# 0001: Blocking update of questionnaires

- **Author:** Oliver Roick
- **Status: ** Draft proposal

## Context

The Cadasta platform allows creating custom, project-specific forms to collect information for records (locations, tenure relationships, and parties); we call these custom forms _questionnaires_. Project managers create these custom forms by uploading an Excel spreadsheet that follows the [XLSForm](http://xlsform.org/) specification. 

Currently, questionnaires can be updated arbitrarily. We do not, however, have the necessary mechanism in place to migrate existing data in a project, created using one questionnaire, to the structure of a new questionnaire. 

This poses several problems in regards to the integrity of the data set in a project (this list provides selected examples and is not exhaustive):

1. When a field is removed from a questionnaire, there is no way of validating data that was provided for the field in an older questionnaire. Eventually, we will lose that information because the field doesn't exist in the data structure any longer.
2. When the data type of a field is changed in a way that we can not automatically migrate that information (e.g., the field is changed from a string to number) we will receive exceptions from the system that we can not handle in a meaningful way, unless we define rules for these migrations. 
3. Removing a value from a select field leads to a similar issue: We are not able to automatically re-assign the value of the field without breaking the integrity of the data. 

## Decision

Updating a questionnaire for a project will be disabled as soon as data collectors add data to the project; both the platform website and the API.

## Consequences

By disabling questionnaire update, we ensure that project managers do not introduce breaking changes to a questionnaire that affect the integrity of existing data. 

A negative effect of this approach is that project managers also won't be able to provide smaller updates to a questionnaire; for instance, fixing a typo in a field label. 

Eventually, we should think about deprecating XLSForms altogether and replace the current procedure with a user interface to create custom questionnaires along with rules that define how existing data is migrated after a questionnaire update. 
