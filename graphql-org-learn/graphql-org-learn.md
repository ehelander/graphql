# [GraphQL.org/learn](https://graphql.org/learn/)

## [Introduction to GraphQL](https://graphql.org/learn/)

- See also
  - How to GraphQL [fullstack tutorial](https://www.howtographql.com/)
  - Free online course, [Exploring GraphQA: A Query Language for APIs](https://www.edx.org/course/exploring-graphql-a-query-language-for-apis)
- A GraphQL service is created by:
  - Defining types and fields on those types
  - Providing functions for each field on each type
- A received query is checked to ensure it only contains defined fields.

## [Queries and Mutations](https://graphql.org/learn/queries/)

### Fields

- Essence of GraphQL:
  - Asking for specific fields on objects.
- Queries have the same shape as the result.
  - You always get back what you expect.
  - The server knows what fields the client is looking for.

    ```gql
    {
      hero {
        name
      }
    }
    ```

    ```json
    {
    "data": {
      "hero": {
      "name": "R2-D2"
      }
    }
    }
    ```

- Queries allow *sub-selection*: selecting fields from nested objects.
- Queries allow comments (`#`).
- A large amount of data can be fetched in a single request (vs. the numerous roundtrips necessary with REST).
- A query looks the same for a single item as for a list of items.
  - The schema indicates which is expected.

  ```gql
  {
    hero {
      name
      # Queries can have comments!
      friends {
        name
      }
    }
  }
  ```

  ```json
  {
    "data": {
      "hero": {
        "name": "R2-D2",
        "friends": [
          {
            "name": "Luke Skywalker"
          },
          {
            "name": "Han Solo"
          },
          {
            "name": "Leia Organa"
          }
        ]
      }
    }
  }
  ```

### Arguments

- Each field and nested object can receive a separate set of arguments.
  - Arguments can be passed to scalar fields, allowing data transformations on the server (instead of on each client)

  ```gql
  {
    human(id: "1000") {
      name
      height
    }
  }
  ```

  ```json
  {
    "data": {
      "human": {
        "name": "Luke Skywalker",
        "height": 1.72
      }
    }
  }
  ```

- Example with enums

  ```gql
  {
    human(id: "1000") {
      name
      height(unit: FOOT)
    }
  }
  ```

  ```json
  {
    "data": {
      "human": {
        "name": "Luke Skywalker",
        "height": 5.6430448
      }
    }
  }
  ```

### Aliases

- Aliases allow renaming the result of a field to anything.
  - This facilitates querying for the same field with different arguments.

  ```gql
  {
    empireHero: hero(episode: EMPIRE) {
      name
    }
    jediHero: hero(episode: JEDI) {
      name
    }
  }
  ```

  ```json
  {
    "data": {
      "empireHero": {
        "name": "Luke Skywalker"
      },
      "jediHero": {
        "name": "R2-D2"
      }
    }
  }
  ```

### Fragments

- Reusable query units: *fragments*
  - A fragment can represent a set of fields and be reused across queries.

  ```gql
  {
    leftComparison: hero(episode: EMPIRE) {
      ...comparisonFields
    }
    rightComparison: hero(episode: JEDI) {
      ...comparisonFields
    }
  }

  fragment comparisonFields on Character {
    name
    appearsIn
    friends {
      name
    }
  }
  ```

  ```json
  {
    "data": {
      "leftComparison": {
        "name": "Luke Skywalker",
        "appearsIn": [
          "NEWHOPE",
          "EMPIRE",
          "JEDI"
        ],
        "friends": [
          {
            "name": "Han Solo"
          },
          {
            "name": "Leia Organa"
          },
          {
            "name": "C-3PO"
          },
          {
            "name": "R2-D2"
          }
        ]
      },
      "rightComparison": {
        "name": "R2-D2",
        "appearsIn": [
          "NEWHOPE",
          "EMPIRE",
          "JEDI"
        ],
        "friends": [
          {
            "name": "Luke Skywalker"
          },
          {
            "name": "Han Solo"
          },
          {
            "name": "Leia Organa"
          }
        ]
      }
    }
  }
  ```

- Fragments can access variables declared in the query or mutation.

  ```gql
  query HeroComparison($first: Int = 3) {
    leftComparison: hero(episode: EMPIRE) {
      ...comparisonFields
    }
    rightComparison: hero(episode: JEDI) {
      ...comparisonFields
    }
  }

  fragment comparisonFields on Character {
    name
    friendsConnection(first: $first) {
      totalCount
      edges {
        node {
          name
        }
      }
    }
  }
  ```

  ```json
  {
    "data": {
      "leftComparison": {
        "name": "Luke Skywalker",
        "friendsConnection": {
          "totalCount": 4,
          "edges": [
            {
              "node": {
                "name": "Han Solo"
              }
            },
            {
              "node": {
                "name": "Leia Organa"
              }
            },
            {
              "node": {
                "name": "C-3PO"
              }
            }
          ]
        }
      },
      "rightComparison": {
        "name": "R2-D2",
        "friendsConnection": {
          "totalCount": 3,
          "edges": [
            {
              "node": {
                "name": "Luke Skywalker"
              }
            },
            {
              "node": {
                "name": "Han Solo"
              }
            },
            {
              "node": {
                "name": "Leia Organa"
              }
            }
          ]
        }
      }
    }
  }
  ```

### Operation name

- Shorthand: Can omit `query` (operation type) keyword and query name.
  - Generally, preferable to include both the *operation type* and *operation name*.
    - Operation
      - Types
        - `query`
        - `mutation`
        - `subscription`

  ```gql
  query HeroNameAndFriends {
    hero {
      name
      friends {
        name
      }
    }
  }
  ```

  ```json
  {
    "data": {
      "hero": {
        "name": "R2-D2",
        "friends": [
          {
            "name": "Luke Skywalker"
          },
          {
            "name": "Han Solo"
          },
          {
            "name": "Leia Organa"
          }
        ]
      }
    }
  }
  ```

### Variables

- To include a variable in a query:
  1. Replace the static value in the query with `$variableName`
  2. Declare `$variableName` as one of the variables accepted by the query
  3. Pass `variableName: value` in the separate, transport-specific (usually JSON) variables dictionary
     - Avoid using string interpolation to construct queries from user-supplied values.

  ```gql
  query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
      friends {
        name
      }
    }
  }
  ```

  Variables:

  ```txt
  {
    "episode": "JEDI"
  }
  ```

  ```json
  {
    "data": {
      "hero": {
        "name": "R2-D2",
        "friends": [
          {
          "name": "Luke Skywalker"
          },
          {
          "name": "Han Solo"
          },
          {
          "name": "Leia Organa"
          }
        ]
      }
    }
  }
  ```

- Variable definition: `($episode: Episode)`
  - A list of variables, prefixed with `$` and followed by their type.
  - Possible variable types:
    - Scalar
    - Enum
    - Input object
  - Declared variables are required *unless* the type contains a `!`.
    - If the variable is being passed to a field requiring a non-null argument, it's required regardless.
- Variables can be assigned default values in the query: `($episode: Episode = JEDI)`

### Directives

- Directives allow dynamically changing the structure and shape of a query using variables.
- When `withFriends` is `false`:

```gql
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

```txt
{
  "episode": "JEDI",
  "withFriends": false
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

```gql
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

```txt
{
  "episode": "JEDI",
  "withFriends": true
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

- A directive can be attached to a field or fragment inclusion.
- The core GraphQL specification includes 2 directives (which must be supported by a server implementation to comply with specs):
  - `@include(if: Boolean)`
    - Include the field if the argument is `true`
  - `@skip(if: Boolean)`
    - Skip the field if the argument is `true`.
- Server implementations can define new directives.

### Mutations

- Convention: Any operations that cause data change are sent explicitly via a mutation.


```gql
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

```txt
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

```json
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

- When a mutation field returns an object type (e.g., `createReview` above), nexted fields can be returned (e.g., `starts`, `commentary`).
  - Useful when mutating existing data: Can mutate and query the new value of the field in a single request.
  - Note that `review` is an *input object type* (not a scalar).
- Query fields are executed in parallel. Mutation fields run in series.
  - This prevents race conditions when a mutation includes a field more than once.

### Inline Fragments

- Example below:
  - `hero` returns (interface) type `Character`, which may be either a `Human` or a `Droid` (concrete) type.
    - The direct selection (for `hero`) can only request fields that exist on the `Character` interface (e.g., `name`).
    - To request fields that only exist on the concrete types (`Droid` or `Human`, not on `Character`), use an *inline fragment* with a type query.
    - Named fragments can be used similarly (since a named fragment always has a type attached).

  ```gql
  query HeroForEpisode($ep: Episode!) {
    hero(episode: $ep) {
      name
      ... on Droid {
        primaryFunction
      }
      ... on Human {
        height
      }
    }
  }
  ```

  ```json
  {
    "ep": "JEDI"
  }
  ```

  ```json
  {
    "data": {
      "hero": {
        "name": "R2-D2",
        "primaryFunction": "Astromech"
      }
    }
  }
  ```

### Meta fields

- In some cases, it is not possible to know what type will be received.
- The `__typename` meta field returns the object type at that location.
- In the example below, `search` is a union type that can return 3 different types. The `__typename` field allows the client to determine the type of a specific object that's returned.

  ```gql
  {
    search(text: "an") {
      __typename
      ... on Human {
        name
      }
      ... on Droid {
        name
      }
      ... on Starship {
        name
      }
    }
  }
  ```

  ```json
  {
    "data": {
      "search": [
        {
          "__typename": "Human",
          "name": "Han Solo"
        },
        {
          "__typename": "Human",
          "name": "Leia Organa"
        },
        {
          "__typename": "Starship",
          "name": "TIE Advanced x1"
        }
      ]
    }
  }
  ```

## [Schemas and Types](https://graphql.org/learn/schema/)

- A query starts with a 'root' object.
- Language-agnostic 'GraphQL schema language'

### Object types and fields

- Most basic part of GraphQL schema: object types

```gql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- `Character`
  - An object type
    - A type with fields
- `name` & `appearsIn`
  - Fields
- `String`
  - Built-in scalar type
    - Resolve to simple scalar values
    - No sub-types
- `String!`
  - Non-nullable String field
    - The GraphQL service will always return a value.
- `[Episode]!`
  - Non-nullable array of `Episode` objects.

### Arguments

- Every field on a GraphQL object type can have 0 or more arguments.

```gql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

- All arguments are named (passed by name, not position: `unit: `).
- Arguments can be required or optional.
  - Optional arguments can include a default value (`= METER`).

### Query & Mutation types

```gql
schema {
  query: Query
  mutation: Mutation
}
```

- 2 special object types in a schema that define the *entry point* of a GraphQL query.
  - `query`
    - Every GraphQL service has a `query` type.
  - `mutation`
    - Not every GraphQL service has a `mutation` type.

### Scalar types

- Scalar types represent the leaves of the query (no sub-leaves).
- Default GraphQL scalars:
  - `Int`
    - 32-bit signed integer
  - `Float`
    - Double-precision floating-point value
  - `String`
    - UTF-8 character sequence
  - `Boolean`
    - `true` or `false`
  - `ID`
    - Unique identifer.
    - Serialized as a String (but not intended to be human-readable).
- Can also specify custom scalar types (e.g., `Date`).
  - Up to implementer to decide how it should be serialized, deserialized, and validated.

### Enumeration types

- Scalars that are restricted to a specified set of allowed values.
- Faciliates:
  - Validating that arguments are one of the allowed values.
  - Communicating the set of allowed values for the field via the type system.
- GraphQL service implementations will make use of enums if languages support them; otherwise, they will be implemented some other way.

```gql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

### Lists and Non-Null

- Types that can be defined in GraphQL:
  - Object
  - Scalar
  - Enum
- *Type modifiers* can be used for these types.
  - `!` after the type name: Non-null
    - The server always expects to return a non-null value.
      - If it gets a null, it will trigger a GraphQL execution error.
  - `[]` around the type: List
    - `[String!]`: List can be null, but can't have null members.
    - `[String]!`: List cannot be null, but can contain null members.

### Interfaces

- Interface: An abstract type that includes a certain set of fields the type must include to implement the interface.

```gql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}

type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

- To query for `Character` and get the `primaryFunction` if the returned item is a `Droid`, use an inline fragment.

  ```gql
  query HeroForEpisode($ep: Episode!) {
    hero(episode: $ep) {
      name
      ... on Droid {
        primaryFunction
      }
    }
  }
  ```

  ```json
  {
    "ep": "JEDI"
  }
  ```

  ```json
  {
    "data": {
      "hero": {
        "name": "R2-D2",
        "primaryFunction": "Astromech"
      }
    }
  }
  ```

### Union types

- Union types don't specify common fields between the types.
  - Members of a union type must be concrete objects (not interfaces or other unions).

```gql
union SearchResult = Human | Droid | Starship
```

```gql
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

```gql
{
  search(text: "an") {
    __typename
    ... on Character {
      name
    }
    ... on Human {
      height
    }
    ... on Droid {
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

```json
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
```

### Input types

- A complex object can be passed as an argument.
  - This is particularly useful when working with mutations (e.g., creating an object).
- It's defined with the `input` keyword instead of `type`.

```gql
input ReviewInput {
  stars: Int!
  commentary: String
}
```

- In a mutation:

  ```gql
  mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
    createReview(episode: $ep, review: $review) {
      stars
      commentary
    }
  }
  ```

  ```json
  {
    "ep": "JEDI",
    "review": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
  ```

  ```json
  {
    "data": {
      "createReview": {
        "stars": 5,
        "commentary": "This is a great movie!"
      }
    }
  }
  ```
