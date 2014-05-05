# A Basic Module Example

We will build a HelloWorld module. This is the minimum required to create a Forge module.

## Inform the Application

First, we add the module to application.global.php to make the application aware of it.

```php
<?php
// configs/application.global.php

return [
    'modules' => [
        // Forge modules first
        'ForgeApplication',
        'ForgeAcl',
        // More Forge modules...

        // Third party modules second
        'AcmeWidgets',

        // Then, local modules
        'HelloWorld',

        // Then, Application module
        'Application',
    ],
    //...
];
```

## Describe the Module

Then, we create a Module.php file which describes the module. For the sake of simplicity, all
configuration has been included directly within the getConfig() method. When configuration gets
more complex, it is recommended to create config files under a "config" directory within the
module and include them in the getConfig() method.

```php
<?php
// modules/local/HelloWorld/Module.php

namespace HelloWorld;

use Forge\ModuleManager\StandardModule;

class Module extends StandardModule
{
    public function getConfig()
    {
        return [
            /**
             * version, base_path, and namespace are the minimum required nodes
             */
            'version' => '0.1.0',
            'base_path' => __DIR__,
            'namespace' => __NAMESPACE__,

            /**
             * Tell the service manager how to create the controller.
             */
            'controllers' => [
                'invokables' => [
                    'HelloWorld\Controller\HelloWorldController' =>
                        'HelloWorld\Controller\HelloWorldController',
                ],
            ],
        ];
    }
}
```

@todo Remove the requirement to include the "controllers" node and fall back to the DI
      container.

## Create the Controller

Now, we can create a controller. It will do nothing other than return a composite view.

```php
<?php
// modules/local/HelloWorld/src/Controller/Index.php

namespace HelloWorld\Controller;

use Forge\Mvc\Controller\ActionController;

class Index extends ActionController
{
    public function indexAction()
    {
        return $this->getCompositeView();
    }
}
```

## Create the View

The view is made up of two pieces. A view template and view configuration. First, we will create the template.

```php
<?php
// modules/local/HelloWorld/design/frontend/abstract/view/helloworld/index/index.phtml
?>
Hello World!
```

Then we create the view configuration to include the template within the view and updated the page title.

```php
<?php
// modules/local/HelloWorld/design/frontend/abstract/config/view.config.php

return [
    'helloworld_index_index' => [
        'priority' => 10,
        'conditions' => [
            'module' => 'hello-world',
            'controller' => 'index',
            'action' => 'index',
        ],
        'updates' => [
            'viewHelpers' => [
                'headTitle' => [
                    'actions' => [
                        'addPageTitle' => [
                            'setType' => 'PREPEND',
                            'title' => 'Hello World!',
                        ],
                    ],
                ],
            ],
            'blocks' => [
                'merge' => [
                    'main_content_children' => [
                        'children' => [
                            'content' => [
                                'template' => 'helloworld/index/index',
                            ],
                        ],
                    ],
                ],
            ],
        ],
    ],
];
```

## View the Hello World Page

If you have done everything correctly, you should be able to browse to
http://yourdomain.com/hello-world. You should see the content of the page showing "Hello
World!". You should also see that the page meta title has been changed to "Hello World!".

