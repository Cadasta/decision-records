# 0008: Roles-based permissions

- **Authors:** Chandra Lash, Brian O'Hare
- **Status:** Draft Proposal, 2017-07-26
- **Backlog Issue:** https://github.com/Cadasta/cadasta-platform/issues/1467

## Functional Requirements

Currently the Cadasta platform uses [django-tutelary](http://django-tutelary.readthedocs.io/en/latest/) as the platform's permissions system. We have decided to replace *tutelary* for the following reasons:

1. **Overly complex permissions:**
Initially we believed there would be a need for very granular-level permissioning. The need for such a complex system has never come to fruition.
2. **Performance limitations:**
`django-tutelary` is not optimised for large data sets, resulting in expensive database queries, which eventually slow down the whole system.
3. **Template Context inconsistencies:** The use of flags to indicate what UI elements to display based on permissions is not consistent.

The replacement for `django-tutelary` needs to address the following requirements:

* For each view, we should be able to decide whether the user can access the view or not, based on the users role in the system.
* For views that allow both GET and POST requests, we should be able to determine the required role permissions based on the request method.  For example, in the API you can list projects for an organisation. The same endpoint is used to create projects via a POST request. Any user can list projects, but only organisation admins have permission to create projects as well.
* Views that list objects should be filterable. For example, when listing projects of an organisation, only organisation admins and members of the respective projects should see private projects in the list of projects.
* Roles should be available in the view context so that UI elements can be displayed based on roles in a consistent way. Instead of using flags like `is_allow_add_location` we should settle for a set of flags indicating the roles the user has in a project. For example: `is_org_admin`, `is_project_manager`, `is_data_collector`, `is_project_member`. This set of flags should be used consistently throughout all templates. All existing flags that don't fit into this set should be replaced and removed from the corresponding template contexts.

The following user roles should be defined:

* **Anonymous:**
Any user that visits the platform without registering
* **Public User:**
A registered user that doesn’t belong to a project
* **Project Viewer:**
A registered user with limited project permissions, mostly for viewing project data
* **Mobile Data Collector:**
A registered user with limited permissions, for adding data through mobile devices
* **Data Collector:**
A registered user that will be making data contributions to a project, mostly doing field work
* **Project Manager:**
A registered user that will manage the details, data, and users for a project
* **Organization Administrator:**
A registered user that manages projects for an organization and the organization itself
* **Super User:**
Internal Cadasta user that has all platform permissions

The proposed roles-based system has been detailed in the [Cadasta Platform Permissions](https://drive.google.com/open?id=1RnjuWzjxkbnntJQHxo2Rbg8AiigkydtG7dLvFWZtAnU) document.

These roles are further detailed in the [User Profiles](https://drive.google.com/open?id=0BzpiEtMtHC3rZldVSXgtWl9UOFk) document.

The permissions for each role increase from the left to right in the permissions table, meaning that the permissions are only expanded and none are lost. For example, a Data Collector has all the permissions of a Project User plus more specific to the role. The Project Manager has all the permissions of the Data Collector plus more. The new permissions code should reflect this hierarchy.

The only exception to this is the Mobile Data Collector, whose main purpose is to add project data through mobile devices with no additional project permissions.

An example of how the different roles would view a public project are detailed in the [Sample Project Page based on Roles](https://docs.google.com/document/d/1P4oQWZkjYUjQPzLNQK6y9mdyzUbte9Xc9-n7HkMXiWQ/edit?usp=sharing) document.

## Description of implemented use cases

Use cases are unchanged from existing platform functionality. No additional use cases are covered by this Decision Record.

## UI Changes

No UI changes are foreseen as the objective is to transparently replace `django-tutelary`.

## Solution Overview

The proposed solution aims to avoid a major refactor of existing model, view and mixin code while reducing the complexity introduced by `django-tutelary`. The existing `OrganizationRole` and `ProjectRole` models will be retained. These are core platform models that allow membership relations between users and organizations and projects. To remove these or alter these significantly would require a major refactor of the platform. Instead of removing these models they will be extended by adding a foreign key relationship to the existing django `Group` model. A fixed set of platform-wide groups will be created so that a users `OrganizationRole` or `ProjectRole` will be associated with a single `Group`. Membership of a `Group` effectively determines the nature of the role. A user who is an *organization admin* will have an `OrganizationRole` which belongs to the *OrgAdmin* `Group`. Each `Group` will have an associated set of custom `Permissions` which will determine what actions the user is allowed to perform in the system. When access to a view is requested, the users `OrganizationRole` or `ProjectRole` can be retrieved and its permissions checked against the `permission_required` attribute on the view. This allows much of the existing code to be retained while swapping out tutelary's expensive permissions checking.

## Analysis of alternatives

The proposed solution aims to achieve a trade-off between reducing complexity and maintaining investment in existing platform code that would be difficult or time consuming to refactor. The proposed solution keeps the existing `OrganizationRole` and `ProjectRole` models and their associated view mixins and avoids the introduction of additional third-party permissions libraries.

Some investigation of alternative solutions has been carried out. Both `django-role-permissions` and `django-guardian` were investigated. Both of these libraries appear to require that the users are added directly to a `Group`. Since our proposed solution maintains a level of indirection between `user` and `group`, significant refactoring would be required to adapt existing code to these third-party approaches.

## Architecture description

This solution outlined below is based on a prototype implementation at https://github.com/Cadasta/cadasta-platform/pull/1667

##### Groups and Permissions

The new permissions system leverages `django`'s existing `Group` and `Permission` models. A set of platform wide `Groups` and `Permissions` are created during the `loadstatic` phase of platform deployment. These `Groups` correspond to the platform user `Roles` outlined [here](https://docs.google.com/document/d/1RnjuWzjxkbnntJQHxo2Rbg8AiigkydtG7dLvFWZtAnU/edit). Permissions correspond to [actions](https://github.com/Cadasta/cadasta-platform/wiki/History-Log#detailed-feature-description) the user can perform in the system and are determined by the role(s) the user has. The roles and permissions are defined in `json` in `cadasta/config/permissions` and are loaded using a custom management command at `accounts/management/commands/loadpermissions.py`.

##### Roles

An abstract base `Role` model has been added. The platform's existing `OrganizationRole` and `ProjectRole` models are retained but now implement `Role` and define a  `FK` relation to `auth.Group`. Each `Group` has an associated set of `Permissions`. An `OrganizationRole` for a user with an organization *admin role* will be associated to the `OrgAdmin` `Group` and will have a corresponding set of permissions. Two further class-based roles have been defined for *anonymous* and *public* users respectively.

##### Core Mixins

A  number of additional mixins have been added to `cadasta/core/mixins.py`. The `BaseRolePermissionMixin` checks the permissions required to access a view.  It holds the users current set of roles and provides a `permissions` property which composes a set of permissions based on the users currently assigned roles. The `RolePermissionRequiredMixin` extends `BaseRolePermissionsMixin` and the default django `auth.PermissionRequiredMixin`. The `RoleLoginPermissionRequiredMixin` checks if the user is authenticated and redirects accordingly. The `APIPermissionRequiredMixin` performs permission checking for `API` views.

##### Organization View Mixins

The `ProjectQuerySetMixin` has been removed and replaced with `ProjectListMixin` to provide role-based filtering of projects for `UI` and `API` project list views.  Other than this there are minimal changes to organization mixins.

##### Views

The existing `django-tutelary` approach to determining the required permissions for a view has been replaced with a more conventional django approach. Views can set permissions in one of three ways:
1. Set the permission using the `permission_required` class attribute.
2. Set `permission_required` to a dict which maps a single permission to a request method.
3. Override `get_permission_required` if a more complex approach to determining permissions is needed, e.g. if different permissions are required based on the request method and/or organization or project visibility.

##### Code examples

The following code examples are indicative of the approach and subject to change:

##### Groups and Permissions

We define a set of platform-wide Permissions, eg.,

```python
p1 = Permission(
		name=’View Project’, content_type=’Project’,
		codename=’project.view’
	 )
```

We create the following platform-wide Groups:

`PublicUser`, `ProjectUser`, `DataCollector`, `ProjectManager`, `OrgAdmin`, `SuperUser` eg:

```python
project_user = Group(name=’ProjectUser’, permissions=[p1,...,pn])
```

##### Roles

We define an abstract base model Role with the following attributes:

```python
class Role (model.Model):
	class Meta:
	    abstract = True

	@cached_property
	def permissions(self):
		return [perm.codename
				for perm in self.group.permissions.all()]
```

We then modify `OrganizationRole` and `ProjectRole` to extend `Role`:

```python
class OrganizationRole(Role):
	organization = ForeignKey(Organization)
	group = models.ForeignKey(
        'auth.Group', related_name='organization_roles')

class ProjectRole(Role):
	project = ForeignKey(Project)
	role = CharField(choices=ROLE_CHOICES, default=’PU’)
	group = models.ForeignKey(
        'auth.Group', related_name='project_roles')
```


When a Role is created it is associated with its corresponding Group, eg:

```python
pu = Group.objects.get(name='ProjectUser')
role = ProjectRole(user=user, group=pu, project=project, role=’PU’)
```
When a permissions check need to be made to see if a user can perform a particular action the action will be allowed if the permission is in `role.permissions`.

For the original discussions around this approach see https://docs.google.com/document/d/1CfuaHQ8Sg-EeNp-JeGB_YcFpei7G3aBkpNy4yYB2YHk/edit#


## Database changes

The proposed solution will introduce a number of database changes. Both `OrganizationRole` and `ProjectRole` will need to be updated with a `Group` attribute which will hold the appropriate role group and the associated set of `Permissions`. In addition, the `admin` field on the `OrganizationRole` model will be removed from the database and replaced with a boolean property. The property will be `True` if the `OrganizationRole` belongs to the `OrgAdmin` group. Migrations will be required to add the `group` field to the appropriate models, to set this field to `not-null`, to update existing `OrganizationRole` and `ProjectRole` instances with the corresponding groups.  Ideally, test cases will be provided to verify the correctness of these proposed changes.

## Security Considerations

Since this proposed solution will determine access to platform data in future, it will need to be thoroughly tested.

## Scalablity

The current permissions system implementation based on `django-tutelary` is not scalable, see analysis in the backlog issue [here](https://github.com/Cadasta/cadasta-platform/issues/1467). The lack of bulk test data makes it difficult to test scalability limits. The implementation should be tested against mirrors of the production and demo databases to judge whether there is an improvement in performance v's tutelary.

## Maintainablity

The proposed solution should be more maintainable than the existing `django-tutelary` implementation. It will be less complex, will sit within the platform code, rather than in an external library and will make use of the existing django permissions system. It does not introduce any additional OS or python dependencies.

## Cost implications

None foreseen.

## Prerequisites

None.

## Definition of Done

The implementation will be considered done when it meets the functional requirements specified above. All existing unit tests should pass once tutelary has been removed. In addition, existing user roles and permissions should remain unchanged, ie the switch away from `tutelary` should be completely transparent to the end user, they should maintain their current roles and permissions. User interface elements which depend on user permissions should continue to behave as expected. Finally, there should be a measurable improvement in performance verses the existing solution.
