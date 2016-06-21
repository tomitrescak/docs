---
title: Custom Scalars
order: 209
description: Add custom scalars to a GraphQL schema.
---

## Custom scalars

The GraphQL language comes with following pre-defined scalar types `String`,
`Int`, `Float` and `Boolean`. While this covers most of the user cases, often
you need to support custom atomic data types (e.g. Date) or you need to add validations
to existing types. As a result, GraphQL allows you to define custom `scalars`,
that undertake this role.

To define a custom scalar you simply add it to the schema with the following notation:

```js
scalar MyCustomScalar
```

Consequently, you need to define the behaviour of the scalar, the way it will be serialised
when the value is sent to the client, or how the value is resolved when coming from the client.
For this purpose, each scalar needs to define three methods: `serialize`, `parseValue` and `parseLiteral`.
In Apollo, these functions are defined with the `__` prefix: `__serialize`, `__parseValue` and `__parseLiteral`.
Let's look at couple examples to demonstrate the potential of custom scalars.

### Date scalar

The goal is to define a Date data type, as we want to store the Date values in the database.
We will consider using the MongoDB that can use the native javascript `Date` data type. The `Date` data type
can be easily serialised as a number using the `getTime()` function. Therefore, we will assume that
user sends and receives a number to and from the grapqhl server. This number will be resolved to Date on the server
storing the date value. On client, user can simply create a new date from the received numeric value. Following is the
implementation of the Date data type. First, the schema:

```js
scalar Date

type MyType {
   created: Date
}
```

And here is the resolver (it is defined in Typescript, for javascript simply remove type definitions):

```js
Date: {
  __parseValue (value: number) {
    return new Date(value); // value from the client
  },
  __parseLiteral (ast: any) {
    return (parseInt(ast.value, 10)); // ast value is always in string format
  },
  __serialize(value: Date) {
    return value.getTime(); // value sent to the client
  }
}
```

### Validation of data types

Following example is taken out of the official GraphQL documentation for the scalar datatype.
Consider that you wan to store only odd values in your database field. First, the schema:

```js
scalar Odd

type MyType {
    oddValue: Odd
}
```

And now the resolver:

```js
import { Kind } from 'graphql/type'

Date: {
  __serialize: oddValue,
  __parseValue: oddValue,
  __parseLiteral (ast: any) {
    if (ast.kind === Kind.INT) {
      return oddValue(parseInt(ast.value, 10));
    }
    return null;
  }
}

function oddValue(value) {
  return value % 2 === 1 ? value : null;
}
```
