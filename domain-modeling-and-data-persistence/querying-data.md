# Querying Data

The *Criteria* class is used for querying data in terms of entities. No knowledge is required
of the underlying persistence mechanism or data structure; only entity data and relationships.
*Criteria* can be used for common simple use cases with very minimal code or can be used for
complex queries.

The *Criteria* class cannot handle every possible scenario. In instances where use of the
Criteria class becomes too cumbersome, create a specific method on the Mapper class instead.
**Never directly access or write to the persistence layer from any class other than a Mapper.**
This will break down the barriers between the domain and persistence layers and cause issues in
the future.

## Creating a Criteria Object

The *Criteria* constructor is flexible and will respond to the following arguments as the first argument:

* **Scalar Value** - The given value is treated as an "id" value
* **Numerically Indexed Array** - The given values are treated as "id" values. Data returned
  must match one of the provided values in its id field
* **Key/Value Array** - Data returned must match all provided key/value pairs
* **Array with Reserved Keys** - Reserved keys "_data_keys", "_filters", "_sort", and "_limit"
  may also be used. These were really created for internal purposes of copying criteria data.
  It is recommented to use the explicit **Criteria** methods when the query is more complicated
  than the first three signatures account for.

### Learning Through Examples:

```php
<?php
// Retrieve user with the id 123 and return as a User entity
$criteria = new Criteria();
$criteria->addFilter('id', '==', 123);
$userMapper = $this->getServiceLocator('ForgeUser\Mapper\User');
$user = $userMapper->find($criteria);
```

The "id" here is not a reference to an "id" field in a database. It is a reference to an "id"
field on the entity. The ID is tied to the user, not to the database. The *Mapper* reads the
*Criteria* object and translates "id" to whatever it knows as the "id" field. This could be
"user_id" for example. This could be something other than a MySQL database also. It is the
*Mapper's* job to read and translate the request from the *Criteria* object.

The *Criteria* object is merely a set of instructions for a *Mapper* to execute.

*Criteria* also has shorthand notations. Typing out that much just to grab a user by ID is
extremely verbose.

```php
<?php
$criteria = new Criteria(['id' => 123]);
$user = $userMapper->find($criteria);
```

Which can be reduced to:

```php
<?php
$criteria = new Criteria(123);
$user = $userMapper->find($criteria);
```

...which can further be reduced to...

```php
<?php
$user = $userMapper->find(123);
```

The find() method of a *Mapper* expects a *Criteria* object. If a *Criteria* object is not
passed, a *Criteria* object is created and the first argument is passed to *Criteria's*
constructor().


## Mapper Return Types

*Mappers* have the ability to return data in many ways. The default is to return an entity object like a $user entity. The following options are available:

* **MapperInterface::RETURN_AS_VALUE** - Returns a single value from one row, one column
* **MapperInterface::RETURN_AS_COLUMN** - Returns an array of values from a single column
* **MapperInterface::RETURN_AS_ARRAY** - Returns an array of the first row
* **MapperInterface::RETURN_AS_ENTITY** - Returns an Entity object created from the first row
* **MapperInterface::RETURN_AS_MULTIDIMENSIONAL_ARRAY** - Returns an array of all rows, all
  columns
* **MapperInterface::RETURN_AS_RESULT_SET** - Returns an instance of
  Zend\Db\ResultSet\ResultSetInterface. Think of this as an array of entity objects.
* **MapperInterface::RETURN_AS_PAGINATOR** - Returns an instance of Zend\Paginator\Paginator.
  This is a ResultSet wrapped in a paginator. It can be passed directly to a pagination
  control. It also behaves identitcally to a ResultSet in most other respects.

The default return type is an entity object. The return types are smart and only load what is needed to return the requested data.

The result set return type is interesting in that it does not create entity objects until they
are needed, greatly reducing its memory footprint. When a ResultSet is iterated over, an entity
is created by reading from an internal data array. This entity is also cloned from an entity
prototype which greatly reduces the overhead of constructing a new entity each time.

### Learning Through Examples:

These are the most common use cases. Finding a single entity or finding multiple entities.

```php
<?php
// Find user with ID 123. $user is a user entity object
$user = $userMapper->find(123);

// Find all enabled users and return them as a result set
$enabledUsers = $userMapper->find(['enabled' => true], MapperInterface::RETURN_AS_RESULT_SET);

foreach ($enabledUsers as $user) {
    // $user is instance of a user entity class
    echo $user->getFirstName() . "\n";
}

```

Sometimes, the entity class is unnecessary overhead.

```php
<?php
// Find all enabled users and return them as a multidimensional array
$enabledUsers = $userMapper->find(['enabled' => true],
MapperInterface::RETURN_AS_MULTI_DIMENSIONAL_ARRAY);

foreach ($enabledUsers as $user) {
    // $user is an array
    echo $user['first_name'] . "\n";
}
```

Another common use case is returning ids only for a data set.

@todo Create a better example for column return type

```php
<?php
// Find all enabled users and return their IDs as an array
$criteria = new Criteria();
$criteria->setDataKeys(['id'])
         ->addFilter('enabled', '==', true);
$enabledUserIds = $userMapper->find($criteria, MapperInterface::RETURN_AS_COLUMN);

if (in_array($user->getId(), $enabledUserIds)) {
    echo 'This user is enabled!';
} else {
    echo 'This user is not enabled!';
}
```

## Related entities

Entities can be related to each other. This is done with mapping metadata in configuration files. The result is that related data can be retrieved using a dot notation to separate entity
from property.

```php
<?php
/**
 * Find all enabled users with their role name, sorted by last name, then first name. Role is
 * a related entity to user.
 */
$criteria = new Criteria();
$criteria->addDataKey('role.name')
         ->addFilter('enabled', '==', true)
         ->addSort('last_name', SORT_ASC)
         ->addSort('first_name', SORT_ASC);
$users = $userMapper->find($criteria, MapperInterface::RETURN_AS_RESULT_SET);

/**
 * Find a user and his related default billing address record. This is a one to one
 * relationship
 */
$criteria = new Criteria();
$criteria->addDataKey('default_billing_address.*')
         ->addFilter('id', '==', 123);
$user = $userMapper->find($criteria);

echo $user->getFirstName();
echo $user->getDefaultBillingAddress->getStreet();

/**
 * Find a user and all of his related addresses. This is a one to many relationship
 */
$criteria = new Criteria();
$criteria->addDataKey('billing_addresses.*')
         ->addFilter('id', '==', 123);
$user = $userMapper->find($criteria);

echo $user->getFirstName();
foreach ($user->getBillingAddresses() as $billingAddress) {
    echo $billingAddress->getStreet();
}
```

@todo Some additional magic is also performed to merge subqueries, where the relationship is
one to many. For example, if a customer may have many addresses, the addresses are set to an
addresses key on the customer row rather than returning multiple customer rows, one for each
address as is returned by the query. Need to document this behavior. Also, not entirely sure
of the naming convention for plural names. Need to review.

@todo Organize the documentation below. This was all copied as is from existing documentation.

### Find

The find method has the following signature:

```php
<?php
public function find($criteria = null, $returnFormat = MapperInterface::RETURN_AS_ENTITY, $mergeData = false);
```

The $returnFormat parameter specifies the format in which the method should return data. The possible values are:

* MapperInterface::RETURN_AS_VALUE
* MapperInterface::RETURN_AS_COLUMN
* MapperInterface::RETURN_AS_ENTITY
* MapperInterface::RETURN_AS_ARRAY
* MapperInterface::RETURN_AS_RESULT_SET
* MapperInterface::RETURN_AS_PAGINATOR
* MapperInterface::RETURN_AS_MULTI_DIMENSIONAL_ARRAY

The $mergeData parameter specifies whether to combine related matching data into a more hierarchical format which
makes more sense from a php perspective. The reason for this is that one to many type relationships will provide the same
row data over and over for each joined row.

Aliases may also be used in criteria when selecting data keys. This can be useful when querying for a one to many type of
relationship;

```php
<?php
$userMapper = $this->getServiceLocator('ForgeUser\Mapper\User');
$criteria = new Criteria();
$criteria->addDataKey(['addresses' => 'address.*']);
$users = $userMapper->find($criteria, MapperInterface::RETURN_AS_MULTI_DIMENSIONAL_ARRAY, true);

/**
Array(
    0 => Array(
        first_name => 'John',
        last_name => 'Smith',
        //...
        'addresses' => Array(
            0 => Array(
                street => '123 Test St.',
                city => 'Dallas',
                state => 'TX',
                zip => '75001'
            ),
            1 => Array(
                street => '456 Test St.',
                city => 'Dallas',
                state => 'TX',
                zip => '75001'
            )
        )
    )
)
*/
```

### Relationships & Data Stores

Database mappers have extra functionality built into them for managing relationships between entities. This logic allows
for querying of related data in a single request. Database mappers can also handle entities with data stored across
multiple tables (think EAV). This is done through "Data Stores" which are a connection to another table or set of tables
when additional data is stored.

In order for this to work, there is a small amount of meta data mapping required. This is done via the main application
config in the node "entity_persistence_map". By default, in Forge, this configuration is broken out into a separate
config file per module called "entity-persistence-map.config.php". This config can be spread across many modules and
is merged into one. This means that a module can easily inject its relationship definitions into another module.

Below is an example of the file format along with annotations.

```php
<?php

return [
    'entity_persistence_map' => [
        // Entity code for the user entity
        'user' => [
            // Main database table used to store the user entity
            'table' => 'user',
            // Mapping of entity properties to database columns
            'map' => [
                'id' => 'id',
                'role_id' => 'role_id',
                'username' => 'username',
                'password' => 'password',
                'email' => 'email',
                'first_name' => 'first_name',
                'last_name' => 'last_name',
                'enabled' => 'enabled',
                'profile_image_filename' => 'profile_image_filename',
            ],
            'relationships' => [
                // Entity code of the role entity
                'role' => [
                    // Relationship type. Can be a fully qualified class name or the last part of a class name under
                    // Forge\Model\Mapper\Database\Relationship
                    'type' => 'Simple',
                    // Relationship options
                    'options' => [
                        // The "Simple" relationship types assumes a direct connection to the target table and requires
                        // a list of column pairs that connect the two. The entity code is used for the table here.
                        'column_pairs' => [
                            'user.role_id' => 'role.id',
                        ],
                    ],
                ],
            ],
        ],
        'role' => [
            'table' => 'role',
            'map' => [
                'id' => 'id',
                'code' => 'code',
                'name' => 'name',
            ],
        ],
        'some_entity' => [
            'table' => 'some_entity',
            // Data stores are used to connect to other tables in which this entity stores its data.
            'data_stores' => [
                'some_entity_additional_info' => [ // Data store alias
                    'type' => 'Simple', // Relationship type
                    'table' => 'some_entity_additional_info', // Data store db table name
                    'options' => [
                        'column_pairs' => [
                            'some_entity.some_id' => 'some_entity_additional_info.id',
                        ],
                    ],
                ],
            ],
            'map' => [
                'id' => 'id',
                // The data store prefix is used in the mapping for entities stored in separate data stores
                'code' => 'some_entity_part.code',
                'name' => 'name',
            ],
        ],
    ],
];
```


## To Do

* Use of the $mergeData parameter when returning as a result set or paginator is not yet supported.
* Use of the $mergeData parameter without an "id" field from the database is not yet supported.
* batchSave(), batchCreate(), batchUpdate(), and update() do not yet support saving of entities with multiple data stores.
