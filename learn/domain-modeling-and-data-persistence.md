# Domain Modeling and Data Persistence

Forge Model is an object relational mapping system intended to allow application code to query and manipulate objects in
terms of the domain objects themselves without any knowledge of the persistence layer. There are three primary concepts
that need to be understood in order to use Forge Model.

1. Entity - An entity is a domain object with business data and logic. The domain layer is fully unaware of the
   persistence layer.
2. Mapper - A mapper class is used to query for data or alter data in a persistent storage layer
3. Criteria - The Criteria class is an implementation of the [Repository Pattern][1]. It allows you to build a query in terms
   of domain property names. This query can be executed against a mapper or directly used to filter an array of data.

## Quick Start
### Finding Data
The criteria class is very flexible for querying data, but has conventions that can be used to reduce the amount of code
needed to accomplish a simple task. All methods below accept a string, array or CriteriaInterface object for
specification of the criteria to act on. If a string is specified, it is treated as an id field. If an array is specified,
each key/value pair is treated as an equals operator, where all values given must match. The Criteria object offers more
fine grained specification of criteria if more control is needed.

```php
<?php
$userMapper = $this->getServiceLocator('ForgeUser\Mapper\User');

// Find a user with id 123 and return as an instance of ForgeUser\Model\User
$user = $userMapper->find(123);

// Do the same, but explicitly use a Criteria object
$criteria = new Criteria();
$criteria->addFilter('id', CriteriaInterface::EQUAL_TO, 123);
$user = $userMapper->find($criteria);

// Find all users with enabled set to true and return as a result set of entities
$enabledUsers = $userMapper->find(['enabled' => true], MapperInterface::RETURN_AS_RESULT_SET);

// Do the same, but explicitly use a Criteria object
$criteria = new Criteria();
$criteria->addFilter('enabled', CriteriaInterface::EQUAL_TO, true);
$enabledUsers = $userMapper->find($criteria);

// Find all enabled users with their role name, sorted by last name, then first name. Role is a related entity to user
$criteria = new Criteria();
$criteria->addDataKey('role.name')
         ->addFilter('enabled', CriteriaInterface::EQUAL_TO, true)
         ->addSort('last_name', SORT_ASC)
         ->addSort('first_name', SORT_ASC);
$users = $userMapper->find($criteria, MapperInterface::RETURN_AS_RESULT_SET);
```

### Saving data
```php
<?php
$userMapper = $this->getServiceLocator('ForgeUser\Mapper\User');

// Create a new user
$user = $userMapper->getEntityPrototype();
$user->setFirstName('John')
     ->setLastName('Smith')
     ->setUsername('john.smith')
     ->setEmail('john.smith@example.com')
     ->setPassword($user->encryptPassword('SomePassword'))
     ->setEnabled(true);
$userId = $userMapper->saveEntity($user);

// Update an existing user
$user = $userMapper->find(123);
$user->setUsername('john.smith123');
$rowWasUpdated = $userMapper->saveEntity($user);

// Import many users at once
$user = $userMapper->getEntityPrototype();
$userData = [
    [
        'first_name' => 'John',
        'last_name' => 'Smith',
        'username' => 'john.smith',
        'email' => 'john.smith@example.com',
        'password' => $user->encryptPassword('SomePassword')
        'enabled' => true,
    ],
    [
        'first_name' => 'Bob',
        'last_name' => 'Smith',
        'username' => 'bob.smith',
        'email' => 'bob.smith@example.com',
        'password' => $user->encryptPassword('SomePassword')
        'enabled' => true,
    ],
];
$numUsersCreated = $userMapper->batchCreate($userData);

// Import many user updates at once. All rows must have the same keys
$user = $userMapper->getEntityPrototype();
$userData = [
    [
        'id' => 123,
        'username' => 'john.smith123',
        'email' => 'john.smith123@example.com',
        'enabled' => true,
    ],
    [
        'id' => 456,
        'username' => 'bob.smith123',
        'email' => 'bob.smith123@example.com',
        'enabled' => true,
    ],
];
$numUsersUpdated = $userMapper->batchUpdate($userData);

// The same update array can also be used for a batchSave() method. All rows must still have the same keys. New rows
// should have an "id" key, but it should be set to null
$numUsersUpdatedOrCreated = $userMapper->batchSave($userData);

// Update many users at the same time with the same data
// Specifically, set users with ids 123, 456, or 789 to disabled
$userMapper->update([
    'enabled' => false,
], [123, 456, 789]);

// Delete many users at once
$userMapper->delete([123, 456, 789]);
```

## Entity
An entity is a domain object with business properties and logic. It has no awareness of the persistence layer or mapper
classes. This means that lazy loading of data is not possible, which can make for a slightly awkward workflow for those
used to active record. For example, $user->getBillingAddress() won't work unless you've already loaded the
billing address record from the database into the user entity. This does mean, however, that your domain objects have
a clear objective and only do what they do best.

All entities clearly define the properties they work with. The getProvidedData() method on all entities returns an array
of all data keys relevant directly to this entity, not including related entities' data. The
getProvidedData() method is also used help facilitate the getArrayCopy() and populate() methods to implement the
ArraySerializeable interface. This allows entities to be used directly with forms using ZF2's bind() functionality. The
populate method is also used by the database mapper to create the entity populated with all data returned from the
persistence layer. Entities also can store additional data in an arbitrary data array. The populate() method uses the
ClassMethods hydrator to populate all recognized data points from getProvidedData(), then adds to the generic data array
for unrecognized properties.

All entities also provide a general input filter which can be used for validation. This is done by implementing the
InputFilterAwareInterface interface.

Internally, entities store an $isNew flag, to help mappers determine whether the entity is new or exists already in
storage.

## Mapper
All mapper classes implement these methods per the MapperInterface: find(), createEntity(), updateEntity(), deleteEntity(), batchSave(),
batchCreate(), batchUpdate(), update(), and delete(). These are not specific to the database mapper implementation. The
data source for these mappers could be anything. The primary job of the mapper is to interpret the criteria passed to it
in domain logic terms and interact with the persistence layer accordingly.

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

## Criteria
The Criteria class is the main querying mechanism used in Forge. Very complex queries can be built completely in terms
of entity properties with no knowledge of the underlying database structure. Only the relationship types between entities
need to be understood. In the case where the Criteria class is not sufficient for the complexity, a mapper specific
method should be created.

The criteria constructor accepts criteria data in a number of formats and converts it into properties of the Criteria
object. In all mapper methods which accept criteria data, a Criteria object may be passed or criteria data
may be passed which is in turn passed to the Criteria constructor.

#### Formats
* ID String - If a string is passed, it is interpreted as an "id" field. This is in terms of the entity; the persistence
  layer field name could be anything. This only works for entities that have the concept of an id. This will query for
  the entity with the given ID
* Array of IDs - If an array is passed without string keys, the values are interpreted as id values. This will query
  for all entities which have any of the given ids.
* Key/Value array - If an array of key/value pairs is passed, they are interpreted as entity keys which must all match
  the given values in order to match entities.
* Full criteria array - Criteria data can be passed as a more complete array describing all aspects of Criteria. See the
  Criteria constructor for specifics

## To Do
There are a number of incomplete items in Forge Model. These will be implemented when the need arises.

* Use of the $mergeData parameter when returning as a result set or paginator is not yet supported.
* Use of the $mergeData parameter without an "id" field from the database is not yet supported.
* batchSave(), batchCreate(), batchUpdate(), and update() do not yet support saving of entities with multiple data stores.

[1]: http://martinfowler.com/eaaCatalog/repository.html
