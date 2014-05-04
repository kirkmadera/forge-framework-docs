# Domain Modeling and Data Persistence

Domain models in Forge Framework are true domain models focused on business data, business
logic, and nothing more. Data modeling is complicated enough without the included complexity
of data persistence. Forge Framework implements the
[Data Mapper](http://martinfowler.com/eaaCatalog/dataMapper.html) pattern; the task of data
persistence is offloaded to a class dedicated to this purpose.

To abstract the data storage even further, the
[Repository Pattern](http://martinfowler.com/eaaCatalog/repository.html) is also utilized via
the *Criteria* class. This class allows all data to be queried in direct terms of domain data
and relationships with no knowledge of the underlying data storage structure.

Both of these patterns make it easier to substitute data storage mechanisms. Forge Framework
intends to the best storage mechanism for each specific task.

Other existing ORM systems like [Doctrine](http://www.doctrine-project.org/projects/orm.html)
were also considered but, in the end, were not a good fit.

## Terms

The following PHP class types are used in Forge Framework:

1. **Entity** - An entity is a domain model with business data and logic. *(Note: We may want
   to rename this at some point as domain models may not represent anything tangeable. For
   example, you might write a Tax model to store tax calculation logic, but there is no real
   tax "entity")*
2. **Mapper** - A mapper class retreives or stores entities from persistent storage.
3. **Criteria** - The Criteria class is a communication layer between entities and mappers
   that allow data to be queried in terms of entities.

## Entity
An entity is a domain object with business properties and logic. It has no awareness of the
persistence layer or mapper classes.

All entities clearly define the properties they work with. The getProvidedData() method on all
entities returns an array of all data keys relevant directly to this entity, not including
related entities' data. The getProvidedData() method is also used help facilitate the
getArrayCopy() and populate() methods to implement the ArraySerializeable interface. This
allows entities to be used directly with forms using ZF2's bind() functionality. The populate
method is also used by the database mapper to create the entity populated with all data
returned from the persistence layer. Entities also can store additional data in an arbitrary
data array. The populate() method uses the ClassMethods hydrator to populate all recognized
data points from getProvidedData(), then adds to the generic data array for unrecognized
properties.

All entities also provide a general input filter which can be used for validation. This is done
by implementing the InputFilterAwareInterface interface.

Internally, entities store an $isNew flag, to help mappers determine whether the entity is new
or exists already in storage.

## Mapper
All mapper classes implement these methods per the MapperInterface: find(), createEntity(),
updateEntity(), deleteEntity(), batchSave(), batchCreate(), batchUpdate(), update(), and
delete(). These are not specific to the database mapper implementation. The data source for
these mappers could be anything. The primary job of the mapper is to interpret the criteria
passed to it in domain logic terms and interact with the persistence layer accordingly.

## Criteria

The Criteria class is used for querying data in terms of entities. No knowledge is required of
the underlying persistence mechanism or data structure; only entity data and relationships.
Criteria can be used for common simple use cases with very minimal code or can be used for
complex queries.

The Criteria class cannot handle every possible scenario. In instances where use of the
Criteria class becomes too cumbersome, create a specific method on the Mapper class instead.
Never directly access or write to the persistence layer from any class other than a Mapper.
This will break down the barriers between the domain and persistence layers and cause issues in
the future.
