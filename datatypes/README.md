What makes a great content management system? How can we create a next-generation reusible, flexible content repository that serves a large swath of web application needs?

This document aims to explore the properties which allow individuals to create well-structured content.

# System
## Properties we want
- Configuration at runtime, not compile time. This means configuration must not be defined in source code. Possible solutions to this are: configuration files, database configuration.

## Definitions
- Entity
  - A generic term for a storable piece of data.
  - Has zero or more fields.
- Data Type
  - A named entity
  - Composed out of data type components and custom fields
  - Primary abstraction for creating an application architecture
- Data Type Component
  - A predefined fieldset which, when added to a data type, satisfies various properties that a data type might require. See [#properties-we-want](Properties we want).
- Value
  - An instance of a data type or field
- Fields
  - A named attribute of a data type
  - Think "first name" on a "user" data type
  - Imposes some contstraint on the information which can be stored
    - At the most basic level, an "integer" or "string"
    - Can have complex contstraints like an email or address field
    - Can have default or custom validations
  - Can be basic or advanced
    - A basic type usually maps to a programming type primitive like string or int
    - An advanced type might make use of one of more basic types, but impart special meaning to them.
      - E.G. an email field is a simple string, but additional contstraints are implied
      - E.G. a reference field might be a simple string or int, but the meaning of the field is changed

# Data Types
## Properties we want
Not every data type needs to have all of these properties, but it must be possible to declare types which satisfy any and all of these properties, in any combination.
- Relatable
  - Data types must be able to relate to one another.
  - Types of relationships:
    - "Has a"
      - Used to embed entities in parent entities
      - Reference field
      - When the parent entity cannot exist or would be incomplete without the reference
    - "Like"
      - Used to express similarity
      - Reference to a shared taxonomy entity
    - "Related by"
      - Used to express complex relationships between entities.
      - When the entity can exists on its own or may be related to many different entities
      - When the relationship must be categorized
        - I.E. a "paid membership" vs. "free membership" or "President of Club A" vs. "Treasurer of Club A"
- Composable
  - Data types must be able to be combined to satisfy one or more interfaces.
- Auditable
  - One should be able to know when and how values were mutated and by whom.
- Revisionable
  - Closely related to auditable. Revisionable means that certain variations of a value can be declared canonical.
  - E.g. a data type value might have a draft, but it's previously published instance may still be "canonical" until the newer draft is published.
- Translatable
  - Every revision must be able to have internationalized field values.
- Observable
  - When a data type value is created, mutated, or removed, the system should be able to subscribe to these events
  - The system should be able to understand the previous state and the new state of the value
- Identifiable
  - Data should be able to be uniquely addressable
  - It's difficult to imagine a data type which would not be identifiable, perhaps for some kind of counter.
    - Maybe an application like a voting system. You might create an unidentifiable "vote" data type and a separate "vote receipt" data type. The vote receipt could capture things like who voted and when, but not _how_ they voted.
  - Types of identity:
    - Random
      - E.G. UUID
    - Serial
      - E.G. 1, 2, 3
    - Named
      - E.G. "system.application.domain"
      - If this type of identity is to be user-facing (i.e. used by anything other than code), care must be taken to ensure that this value can be internationalized.

# Data Type Components
Data type components represent predefined fieldsets which can bestow the properties we listed above upon a data type. They should be composable so that data types can use different combinations of these components together. This means that implementors should use conventional field names, constraints and validations where possible. E.g. if both the content and taxonomy components require a label, one should not be called "name" if the other is called "title".
- User facing
  - Content
    - By default, all content must be revisionable, translatable, and observable
  - Taxonomy
    - Taxonomy enables dynamic grouping and associativity between data type values
  - Relationships
    - Relationships enable complex and dynamic associativity between data types
  - Configuration
    - Configuration should be validated
    - Updates to configuration cannot create logical exceptions
    - Configuration must be statically addressable, i.e. identified prior to its instantiation
      - E.g. One must be able to load configuration by an ID like "system.application.domain". A UUID would not satisfy this requirement.

# Basic Field types
- boolean
- string (short)
  - E.g. used for a "title" field
- string (long)
  - E.g. used to a "description" field
- blob
  - Stores complex objects in binary form, can be backed by different storage than the containing data type
- number (integer)
- number (float)
- struct
  - Can itself hold multiple fields
  - Field set is unique to the containing data type
  - In a multivalue field, a struct allows on to not overload field delta with order
    - E.g. one can combine a string and integer field in a struct to create an ordered list

# Advanced Field Types
Implementors shoudl allow for advanced field types to be created and may define their own. The following types are required.
- reference
  - Embeds a foreign data type value in the containing data type
- label
  - A label is like a taxonomy term, except the possible values must be declared in configuration. It should be used to group data type values into sets.

# Cardinality
Cardinality is the number of values that a single field might hold. All fields must support multiple values for a given field. Robust content management systems must _never_ assume that cardinality of any property will be 1. Instead a configurable limit should be associated with any field instance and values above this maximum should be enforced by validation. This preserves the flexibility of a content architecture in the long term. Implementors should be mindful to use this configuration well. E.g. when serializing entities, one should serialize fields which have a cardinality of 1 as a single value field. If a field has a cardinality of 2 or more, it should be serialized as an array - even if only one value exists. This same prinipal should apply for internal APIs where feasible.

# Actor model
Within the system, CRUD events should be attributable to users or system agents. This allows the system to satisfy the "auditable" requirement. The person or thing which causes these CRUD events should be represented by an _actor_. An _actor_ is a defined, fieldable entity type which the ability to be authenticated.
An actor causes CRUD events to occur by providing _intentions_ to the system. By providing a layer of abstraction between actors and actual CRUD events, a separation of the program interface and data is enforced. By mapping all interactions through an intention model, it should be possible to easily implement various program interfaces. These interfaces might be RESTful drivers (e.g. JSON API, GraphQL, etc.) or even command line drivers.
Actors can also subscribe to observables and system events. This can be used to implement custom actions like "when foo, mutate bar" or implement real-time interfaces.
## Actors
  - Actors can be authenticated
  - An actor can be used to represent a user, but it may also be used to represent an internal or external system. This might be the "system" or another application integrating via an API.
  - An actor can be authenticated or unauthenticated. Implementations may treat actor entities differently based on this state.
  - Actors interact via the system the observations and intentions
## Intentions
Intentions are value structures which can cause CRUD operations to be performed within the system. Intentions may not always be satisfied, this may be due to access policies, failures, or other external factors. Intentions do not need to map directly to single entities. Intentions may be nested to create complex interactions that are processed by the system. This may allow the system to optimize the intentions into fewer database round-trips.
