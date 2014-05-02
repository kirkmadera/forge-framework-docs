# Core Framework Principles

The goal of Forge Framework is to allow for development of reusable complex application
subsystems like Product Catalog Managment, Content Management, or Inventory Management
which work together when paired, but also work standalone. Any subsystem should provide
a clear interface and be easily replaceable with another implementation. The architecture
of Forge Framework allows for full customization of any piece of functionality or design,
while retaining the ability to upgrade the base framework without refactoring.

##Additional Guiding Principles

* Must allow for completely self-contained modules
* Must allow modules to affect any part of the view using a composite view design pattern
* Should keep module dependencies to a minimum
* Must explicitly define module dependencies
* Should leverage Zend Framework when possible
* Must use Zend Framework coding standards
* Must handle errors and exceptions well from both a developer perspective and an end-user
  perspective
* Must allow for unintrusive extension of existing functionality using design patterns
  like event/observer and service locator/dependency injection
* Must allow for layers of core, third party, then local functionality
* Must allow for multiple cascading themes
* Must make security a high priority
* Must make performance a high priority
* Must be capable of importing/exporting data for use in or from other systems
* Must define clear integration points and interfaces which allow for complete
  substitution of parts of the system
* Must be well documented
* Must provide unit tests for all core functionality
* Should quarantine files that should not be versioned into discrete directories
* Must be open-source
