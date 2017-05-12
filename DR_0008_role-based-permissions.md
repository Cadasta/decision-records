# 0008: Roles-based permissions

- **Author:** Chandra Lash
- **Status:** Submitted, 2017-05-12

## Context

Currently the Cadasta platform uses [django-tutelary](http://django-tutelary.readthedocs.io/en/latest/) for permissioning.

We have decided to replace this permission system based on:

1. **Overly complex permissions:** 
Initially we believed there would be a need for very granular-level permissioning. The need for such a complex system has never come to fruition. 
2. **Performance limitations:** 
We have found django-tutelary to have a negative effect when iterating over large permission sets. 

## Decision

The platform permissioning would be better served by removing tutulary and replacing it with a well-documented roles-based system.

The roles-based system has been detailed in the [Cadasta Platform Permissions](https://drive.google.com/open?id=1RnjuWzjxkbnntJQHxo2Rbg8AiigkydtG7dLvFWZtAnU) document. 

### User Roles

* **Anonymous:**
Any user that visits the platform without registering
* **Public User:**
A registered user that doesn’t belong to a project
* **Project User:**
A registered user with limited project permissions, mostly for viewing project data
* **Data Collector:**
A registered user that will be making data contributions to a project, mostly doing field work
* **Project Manager:**
A registered user that will manage the details, data, and users for a project
* **Organization Manager:**
A registered user that manages projects for an organization and the organization itself
* **Super User:**
Internal Cadasta user that has all platform permissions

These roles are further detailed in the [User Profiles](https://drive.google.com/open?id=0BzpiEtMtHC3rZldVSXgtWl9UOFk) document.

### Role Hierarchy

The permissions for each role increase from the left to right in the permissions table, meaning that the permissions are only expanded and none are lost. For example, a Data Collector has all the permissions of a Project User plus more specific to the role. The Project Manager has all the permissions of the Data Collector plus more. The new permissions code should reflect this hierarchy. 

An example of how the different roles would view a public project are detailed in the [Sample Project Page based on Roles](https://docs.google.com/document/d/1P4oQWZkjYUjQPzLNQK6y9mdyzUbte9Xc9-n7HkMXiWQ/edit?usp=sharing) document.

## Consequences

Replacing tutulary with solely a roles-based system should simplify the platform code as well as improve performance. This roles-based system should be easier to document and communicate to partners.

The only downside I see at this time is removing the ability to have extremely granular-level details. I can see in the future the possibility of having to add additional roles based upon evolving needs. Overall I believe the pros far outweigh the cons.

