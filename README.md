Forge Framework
===============

> Forge Framework is currently under development. We are internally using gitbook
> and github.io to generate documentation only. This is intended to be an open source
> project, but will not be released it until it has reached a certain level of stability.
> No specific release date has been set, but it will likely be sometime in early 2015.

Forge Framework is an application level framework that runs on Zend Framework 2.
The purpose of Forge Framework is to provide prebuilt application components, fully
functional out of the box, with flexible customization strategies that offer full
control to the user but do not break the upgrade path.

Zend Framework was chosen because of it's maturity, documentation, coding standards,
loosely coupled libraries, and event based MVC architecture. The event based architecture
of ZF2 made it easy to introduce Forge Framework functionality in a non intrusive way.

A concerted effort was made to use Zend Framework 2 functionality as is where possible.
All ZF2 documentation can be relied upon to work with Forge Framework except where
specifically noted. ZF2 community modules may be used directly with Forge Framework.
This is extremely useful for quickly testing ZF2 modules. However, to fully utilize
Forge Framework's view building capabilities, some customization is required.

Forge Framework also shares many similarities with Magento. For example, the composite
view strategy based in layout configuration files is based off of Magento's view strategy.
