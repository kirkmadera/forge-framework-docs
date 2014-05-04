# Software Requirements

Forge Framework is still in very early development stages, so we are using somewhat
bleeding edge technologies with the expectation that they will be stable and widely
available at the time that Forge Framework is released.

## Current Requirements

* **PHP >=5.5.3** - Forge uses password_hash() from 5.5. Not sure if any 5.6
  features/functions will be needed.
* **ZF2 >=2.3**
* **MySQL >=5.5**
* **PHP intl extension** - This is currently required because it made coding some of the
  translation stuff easier. This will be removed as a requirement on release and made a
  recommendation instead.
* **PHP json extension** - Currently required to perform json tasks. May use ZF2's Json
  classes in the future to avoid forcing a user to install a PHP extension

## Planned Requirements (Depending on stability and availability at release time)

* **PHP 5.6 recommended** - Will probably support 5.5
* **MySQL 5.6 required** - InnoDb Fulltext search is expected to be used
* **PHP intl extension recommended**
* **PHP json extension recommended**
