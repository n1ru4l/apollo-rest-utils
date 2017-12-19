# Rest GraphQL Schema Generator

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
[![npm version](https://badge.fury.io/js/%40n1ru4l%2Fgraphql-schema-generator-rest.svg)](https://badge.fury.io/js/%40n1ru4l%2Fgraphql-schema-generator-rest)
[![CircleCI](https://circleci.com/gh/n1ru4l/graphql-schema-generator-rest.svg?style=svg)](https://circleci.com/gh/n1ru4l/graphql-schema-generator-rest)

This package provides the functionality of generating a GraphQL schema from type definitions annotated with `@rest` directives.

## Install

```shell
yarn add @n1ru4l/graphql-schema-generator-rest
```

## Usage

[Check out the examples!](https://github.com/n1ru4l/graphql-schema-generator-rest/tree/master/examples)

### Creating a schema

```javascript
import { generateRestSchema } from '@n1ru4l/graphql-schema-generator-rest'
import { graphql } from 'graphql'
import gql from 'graphql-tag'
import fetch from 'node-fetch'

const typeDefs = gql`
  type User {
    id: ID!
    login: String!
    friends: [User]!
      @rest(
        route: "/users/:userId/friends"
        provides: { userId: "id" } # map id from parent object to :userId route param
      )
  }

  type Query {
    user(id: ID!): User @rest(route: "/users/:id")
  }
`

const schema = generateRestSchema({
  typeDefs,
  fetcher: fetch,
})

const query = `
  query user {
    user(id: "2") {
      id
      login
      friends {
        id
        login
      }
    }
  }
`

graphql(schema, query)
  .then(console.log)
  .catch(console.log)
```

Available options for `generateRestSchema`:

|                       |                                                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------- |
| **`typeDefs`**        | AST object for GraphQL type definitions generated by [`graphql-tag`](https://www.npmjs.com/package/graphql-tag) |
| **`fetcher`**         | WHATWG Fetch Compatible fetch implementation                                                                    |
| **`queryMappers`**    | Object of queryMappers that manipulate the query params before a request is sent                                |
| **`requestMappers`**  | Object of requestMappers that manipulate the request body object before a request is sent                       |
| **`responseMappers`** | Object of responseMappers that manipulate the response returned by a request                                    |

### Type Definitions

```graphql
type User {
  id: ID!
  login: String!
  friends: [User]!
    @rest(
      route: "/users/:userId/friends"
      provides: { userId: "id" } # map id from parent object to :userId route param
    )
}

type Query {
  user(id: ID!): User @rest(route: "/users/:id")
}
```

Available options for the `rest` directive:

|                     |                                                                                 |
| ------------------- | ------------------------------------------------------------------------------- |
| **`route`**         | The route which is called                                                       |
| **`provides`**      | An object that maps fields from the parent object to the scope of the directive |
| **`method`**        | The HTTP method that will be used (`PUT`, `GET`, `POST`, `PATCH`)               |
| **`query`**         | An object that maps fields to the query params                                  |
| **`queryMapper`**   | The identifier of a a queryMapper that maniplates the query mappings            |
| **`body`**          | An object that maps fields to the request body                                  |
| **`requestMapper`** | The identifier of a requestMapper that manipulates the request body             |

## Recipies

### [apollo-link-schema](https://www.npmjs.com/package/apollo-link-schema)

```javascript
import { generateRestSchema } from '@n1ru4l/graphql-schema-generator-rest'
import { SchemaLink } from 'apollo-link-schema'
import { graphql, print } from 'graphql'
import gql from 'graphql-tag'
import fetch from 'node-fetch'

const typeDefs = gql`
  type User {
    id: ID!
    login: String!
    friends: [User]!
      @rest(
        route: "/users/:userId/friends"
        provides: { userId: "id" } # map id from parent object to :userId route param
      )
  }

  type Query {
    user(id: ID!): User @rest(route: "/users/:id")
  }
`

const schema = generateRestSchema({
  typeDefs,
  fetcher: fetch,
})

const link = new SchemaLink({ schema })

const query = gql`
  query user {
    user(id: "2") {
      id
      login
      friends {
        id
        login
      }
    }
  }
`

makePromise(execute(link, { operationName: `userProfile`, query }))
  .then(console.log)
  .catch(console.log)
```

### [apollo-server-express](https://www.npmjs.com/package/apollo-server-express)

```javascript
import express from 'express'
import bodyParser from 'body-parser'
import { generateRestSchema } from '@n1ru4l/graphql-schema-generator-rest'
import { graphqlExpress, graphiqlExpress } from 'apollo-server-express'
import gql from 'graphql-tag'
import fetch from 'node-fetch'

const typeDefs = gql`
  type User {
    id: ID!
    login: String!
    friends: [User]!
      @rest(
        route: "/users/:userId/friends"
        provides: { userId: "id" } # map id from parent object to :userId route param
      )
  }

  type Query {
    user(id: ID!): User @rest(route: "/users/:id")
  }
`

const schema = generateRestSchema({
  typeDefs,
  fetcher: fetch,
})

const PORT = 3000

const app = express()

app.use('/graphql', bodyParser.json(), graphqlExpress({ schema }))
app.listen(PORT)
```

## Tests

```shell
yarn test
```

## Contribute

### Checkout project

For contributions please fork this repository.

```bash
git clone https://github.com/<your-login>/graphql-schema-generator-rest.git
cd graphql-schema-generator-rest
yarn install
```

### Commiting Changes

Please use `yarn cm` for commiting changes to git.
