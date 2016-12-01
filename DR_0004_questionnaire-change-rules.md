# 0004: Rules for changing existing XLSForm questionnaires

- **Author:** David Palomino 
- **Status:** Accepted, 2016-12-01

## Context

Project managers define the forms (_questionnaires_) to collect information (locations, parties, tenure relationships, and resources) by uploading Excel spreadsheets that follow [XLSForm](http://xlsform.org/) specification.

As it was described in [DR_0001_blocking_questionnaire-upload](https://github.com/Cadasta/decision-records/pull/1), these quesitonnaires can be updated arbitrarily without any restriction, and these changes could be not consistent with existing data in the current project schema. That is the reason why it was decided to disable the upload of new versions of these questionnaires once that there is some collected data (records) into that project.

With this being totally correct, it is also true that sometimes there is a need to do some simple modifications in these quesitonnaires that are not affecting the project schema (with maybe the easiest example to fix a typo in the form). So we need to define clear rules describing the changes that are going to be allowed in those questionnaires once the project has data, so there would be no need to execute any kind of migration of the existing data.

## Decision

The following is an exhaustive list of the allowed changes when updating a questionnaire (i.e. uploading a new version of it) once that there is some data in the project. Anything that is not listed in the following list is not allowed. If there is no data yet in the project, then any change in the questionnaire will be allowed.

List of allowed changes: 

1. Changes in `form_id` and `title` (in the `settings` tab)
2. Change in the default language to use (`default_language` in `settings` tab)
3. Add new languages for the labels (i.e. adding `label:xx` column in `survey` and `choices` tab)
4. Change in language used in labels
5. Removal of a language used in labels (i.e. deleting `label:xx` column in `survey`and `choices` tab)
6. Any change in any label in either `survey` or `choices`
7. Adding new choices for the multiple-choice questions in `choices` tab, with the exception of `respondent`, `land_type`, `tenure_type` and `geo_type`. These will not accept any changes for now (we could revisit this when adding support for additional `respondent`, `land_type` and `tenure_type`). 
8. Add new fields in `survey` of any type
9. Former mandatory/required fields could be made optional, with the exemption of `party_type`, `party_name`, `tenure_type` and `location_type`. 

By definition, anything not contained in the list above will not be allowed. Just showing here some examples of changes that will not be allowed:

1. Change of field names
2. Change of field types
3. Removing existing fields
4. Removing choices in the multiple-choice fields
5. Former optional fields in the questionnaire cannot be made required (`required` column `yes`). 

## Consequences

By limiting the scope of changes in forms we are preventing errors to occur with existing data schemas and, at the same time, allowing some flexibility to do some minor modifications to the questionnaires once the project has been created.

Eventually XLSForms will be deprecated and replaced by a GUI to create the questionnaires.

