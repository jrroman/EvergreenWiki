## Evergreen Project and Distro Settings

The Project and Distro settings pages can be found at the right side dropdown on the navigation menu. All users can access the distro settings page and Project Admins and Superusers can access the project settings page. 

[[images/sidebar.png]]


#### Types of Special Users 

##### Superusers
Superusers can be set in the Evergreen settings file and have access to all Evergreen projects and Evergreen distro settings. 

##### Project Administrators
Project Administrators have access to specific projects that they maintain and can be set by an Evergreen Superuser in the Project Settings page for that specific project. 
After adding the user's Evergreen username to the list of Administrators, that user will be able to access the Project Settings page for that project only and modify repository information, access settings, alerts, and keys.


#### Project Settings

The Project Settings file displays information about the Project itself, as well as configurable fields for managing your project

There are two types of users that can view and edit these settings: Superusers and Project Admins. 
##### General Project Settings
If a Project Administrator wants Evergreen to discontinue or start tracking a project, it can be changed via the Enabled and Disabled radio buttons. 
The display name field  corresponds to what users will see in the project dropdown on the navigation bar. 
Admins can change the location or name of the config file in the repository if they would like to have Evergreen run tests using a different project file located elsewhere, or if they move the config file. 
The batch time corresponds to the interval of time (in minutes) that Evergreen should wait in between activating the latest version. 

[[images/settings.png]]

##### Repository Info
Admins can modify which GitHub repository the project points to and change the owner, repository name, or branch that is to be tracked by Evergreen. 

[[images/repo.png]]

##### Access and Admin Settings
Admins can set a project as private or public. 
A private project can only be seen by logged in users.
A public project is viewable to those who are not logged in as well. 
To set a Project Administrator edit the Admin list (which can only be viewable by Superusers or Project Admins for the project) for the probject by adding the Evergreen username to the list and saving the project settings. 

[[images/admins.png]] 

##### Scheduling Settings 
Admins can enable the ability to unschedule old tasks if a more recent commit passes. 

[[images/schedule.png]]

##### Alerts
Admins can set up email alerts for different events by adding an action and inputting the email address to send the email alert to. 

[[images/alerts.png]]

##### Variables
Admins can store project variables that can be referenced in the config file via an expansion. 

[[images/vars.png]]
 
#### Distro Settings

The Distros page allows all users to see all available distros a project can run tasks on. 
As a superuser or admin, one can also add new distro configurations to be used by other users. 

[[images/distros.png]]
