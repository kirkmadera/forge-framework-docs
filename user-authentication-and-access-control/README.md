# User Authentication and Access Control

Forge Acl facilitates access control management via an Acl Service, which internally uses an instance of
Zend\Permissions\Acl\Acl. An access control list is built via config for each area (frontend, admin). This
list is primarily based on Mvc routes, although it can contain other resource types as well.

## How It's Used

### Route Based Access Control Enforcement
The primary function of the Acl Service is route based access control enforcement. On the application's "route" event, the
Acl Service checks to see if the current user has access to the requested route. If so, the user is allowed to proceed. If
not, one of two things will happen: 1) If the user is not logged in, he will be presented with a login page and an error
message stating that he must log in to access the page. 2) If the user is logged in, but does not have access to the page,
he is presented with a "Not Authorized" error page.

### Navigation
The Acl Service is used directly by the navigation view helper. A navigation item will only be displayed if the user has
access to the resource defined for the item. If no resource is directly defined and the item is an Mvc page, the page's
route is used as the resource. This means that you do not need to explicitly define a resource for each navigation item
in your config.

Here is a quick code excerpt from the code that checks Acl for navigation items. This shows how a resource is
automatically determined for an Mvc page if not explicitly set.

```php
<?php
// If resource not explicitly set for page, use route as resource
if (is_null($resource) && $page instanceof MvcPage) {
    $resource = RouteResource::RESOURCE_PREFIX . $page->getRoute();
}
```

### isAllowed() controller plugin and view helper
Forge Acl defines an "isAllowed" controller plugin and view helper. This means that from a controller, block, or view
template, you can call ```$this->isAllowed($someResourceName[, $somePrivilege)]``` and it will reference the Acl Service and
return whether the current user has access to the given resource.

## Default Access Control Rules
The ForgeUser module defines the main default access control rules. By default, an unauthenticated user is allowed to
all routes, except for those under the "user", and "admin" routes. These routes assume that a user must be
logged in to do anything. Exceptions can be made if needed by defining more specific route rules. All login/logout
routes are ignored by the access control route based enforcer to avoid issues with being able to log in or out. All
logged in frontend users are granted access to all routes under "user". The same is true for the admin area.

The default ForgeUser rules look something like this:

```php
<?php
return [
    'acl_manager' => [
        'areas' => [
            'frontend' => [
                'roles' => [
                    'guest' => null,
                    'member' => 'guest',
                    'user' => 'member',
                ],
                'resources' => [
                    'Route\*'       => null,
                    'Route\signup'  => null,
                    'Route\login'   => null,
                    'Route\logout'  => null,
                    'Route\user'    => null,
                    'Route\user/*'  => null,
                ],
                'rules' => [
                    // By default, allow all roles to all routes. This is to prevent having to add a resource for every
                    // single page
                    'allow_all_roles_to_all_routes' => [
                        'type' => 'allow',
                        'roles' => null,
                        'resources' => [
                            'Route\*',
                        ],
                    ],
                    // Deny all roles from all user routes.
                    'deny_all_to_user_routes' => [
                        'type' => 'deny',
                        'roles' => null,
                        'resources' => [
                            'Route\user',
                            'Route\user/*',
                            'Route\logout',
                        ],
                    ],
                    // Allow users to user routes
                    'allow_user_to_user_routes' => [
                        'type'          => 'allow',
                        'roles'         => 'user',
                        'resources'     => [
                            'Route\user',
                            'Route\user/*',
                            'Route\logout',
                        ],
                    ],
                    // Allow guests to the login page, itself
                    'allow_guest_to_user_login' => [
                        'type' => 'allow',
                        'roles' => 'guest',
                        'resources' => [
                            'Route\login',
                        ],
                    ],
                    // Deny users from guest only pages
                    'deny_user_to_guest_routes' => [
                        'type' => 'deny',
                        'roles' => 'user',
                        'resources' => [
                            'Route\signup',
                            'Route\login',
                        ],
                    ],
                ],
            ],
        ],
    ],
];
```

Each module should define its own access control rules. If a module requires login, but is not within one of the
protected areas, it should define a new protected route. Here is an example from ForgeNotification:

```php
<?php
return [
    'acl_manager' => [
        'areas' => [
            'frontend' => [
                'resources' => [
                    'Route\notification' => null,
                    'Route\notification/*' => null,
                ],
                'rules' => [
                    'deny_all_from_notification_routes' => [
                        'type'          => 'deny',
                        'roles'         => null,
                        'resources'     => [
                            'Route\notification',
                            'Route\notification/*',
                        ],
                    ],
                    'allow_user_to_notification_routes' => [
                        'type'          => 'allow',
                        'roles'         => 'user',
                        'resources'     => [
                            'Route\notification',
                            'Route\notification/*',
                        ],
                    ],
                ],
            ],
        ],
    ],
];
```
