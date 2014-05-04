# Saving data

@todo Document saving data. This was copied over from other existing documentation and needs to
be cleaned.

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
