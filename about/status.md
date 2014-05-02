# Status

##Completed

* Composite view using layout configuration files and block classes, with a shared data
  layer
* Strategy for completely self-contained modules
* Module dependency checking
* Error handling, exception handling, and logging to various outputs
* Design strategy with a minimal functional abstract theme and a more styled default theme. * Base of abstract theme built on HTML 5 Boilerplate
* Javascript and CSS file merging and minification strategy, utilizing named groups
* Admin area login and basic functionality
* Access control strategy for enforcing rules based on roles, privileges, and resource
  accessed
* Flexible data modeling and persistence strategy using the data mapper and repository
  patterns
* User login, forgot password, profile, edit profile functionality
* OAuth login adapter interface. Currently only LinkedIn adapter built
* A developer tools area in the admin where view configuration can be tested and pages can
  be previewed.
* Invite Only functionality where site registration may be closed and invites sent to
  specific email addresses via the admin

##In Progress

* Abstract theme adjustments
* Remove custom module dependency management in favor of ZF2's. This did not exist at the
  module dependency management was built for Forge Framework
* Document current functionality
* Notification functionality with the ability to display notifications to a specific user, a
  specific role, all frontend users, all admin users, or all users
* Inbox system for sending messages back and forth. Not planned to be robust as there is no
  specific business need for this at the moment.

##Planned

* Refactor current view code
* Review any ZF2 functionality overrides and document them
* Cron job scheduler and view in the admin
* Process manager with corresponding admin view
* Queuing system for any process that can be deferred to improve user perceived system
  performance
* Full page caching system based on HTTP protocol similar to that of Symfony
* Ability to set module config settings via the admin
* Module self-installation and self-uninstallation of database structure & data changes
* Facebook OAuth login
* Twitter OAuth login
* Google OAuth login
* More functionality under developer tools

##Major Future Milestones

* Content Management System
* Blog
* eCommerce Functionality (Will split this milestone up later once it becomes feasible)
* Social sharing
* Newsletter signup and integration to marketing platforms like MailChimp and CheetahMail
* Review stability and benefits of Hack Language and potentially implement
* Make fully compatible with HHVM and recommend as the default setup
