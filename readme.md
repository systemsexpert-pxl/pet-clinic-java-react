# Spring PetClinic Sample Application using spring-graphql

This PetClinic version uses the new [spring-graphql](https://github.com/spring-projects/spring-graphql) project, that has been [introduced](https://spring.io/blog/2021/07/06/hello-spring-graphql) in july 2021
and has been [finally release as 1.0.0 GA version](https://spring.io/blog/2022/05/19/spring-for-graphql-1-0-release).

This version uses **Spring Boot 2.7** with **GraphQL for Spring 1.0**.

It implements a [GraphQL API](http://graphql.org/) for the PetClinic and
provides an example Frontend for the API.

## Features

Some features that are built in:

* [Annotated Controllers](https://docs.spring.io/spring-graphql/docs/current-SNAPSHOT/reference/html/#controllers) (see `graphql/*Controller`-classes, e.g. `SpecialtyController` and `VetController`)
* Subscriptions via Websockets (see `VisitController#onNewVisit`)
* Own scalar types (See `PetClinicRuntimeWiringConfiguration` and `DateCoercing`)
* GraphQL Interfaces (GraphQL Type `Person`) and Unions (GraphQL Type `AddVetPayload`), see class `PetClinicRuntimeWiringConfiguration`
* Security: the `/graphql` http and WebSocket endpoints are secured and can only be accessed using a JWT token. More fine grained security is implemented using `@PreAuthorize` (see `VetService`)
  * Example: `addVet` mutation is only allowed for users with `ROLE_MANAGER`
* Pagination and Sorting of results: implemented with `spring-data`, see `OwnerController`
* Tests: See `test` folder for typical GraphQL endpoint tests, including tests for security

# Running the sample application

## Running locally

The server is implemented in the `backend` folder and can be started either from your IDE (`org.springframework.samples.petclinic.PetClinicApplication`) or
using maven from the root folder of the repository:

```
./mvnw spring-boot:run -pl backend
```

Note: the server runs on port **9977**, so make sure, this port is available.

(The server uses an in-memory database, so no external DB is needed)



## Running the frontend

While you can access the whole GraphQL API from GraphiQL this demo application also
contains a modified version of the classic PetClinic UI. Compared to the original
client this client is built as a Single-Page-Application using **React** and **Apollo GraphQL**
and has slightly different features to make it a more realistic use-case for GraphQL.

You can install and start the frontend by using npm:

```
cd ./frontend

npm install

npm run build:css

npm run generate

npm start
```

The running frontend can be accessed on [http://localhost:3000](http://localhost:3000).

For valid users to login, see list above.

![SpringBoot PetClinic, React Frontend](petclinic-ui.png)

# Accessing the GraphQL API

You can access the GraphQL API via the included customized version of GraphiQL.

The included GraphiQL adds support for login to the original GraphiQL.

You can use the following users for login:

* **joe/joe**: Regular user
* **susi/susi**: has Manager Role and is allowed to execute the `createVet` Mutation

After starting the server, GraphiQL runs on [http://localhost:9977](http://localhost:9977)


## Sample Queries

Here you can find some sample queries that you can copy+paste and run in GraphiQL. Feel free to explore and try more 😊.

**Query** all owners whose lastname starts with "K" and their pets:
```graphql
query {
  owners(filter: {lastName: "K"}) {
    pageInfo {
      totalCount
    }
    owners {
      id
      firstName
      lastName
      pets {
        id
        name
      }
    }
  }
}
```

Add a new Visit using a **mutation** (can be done with user `joe` and `susi`) and read id and pet of the
new created visit:

```graphql
mutation {
    addVisit(input:{
        petId:3,
        description:"Check teeth",
        date:"2022/03/30",
        vetId:1
    }) {
        newVisit:visit {
            id
            pet {
                id
                name
                birthDate
            }
        }
    }
}
```

Add a new veterinarian. This is only allowed for users with `ROLE_MANAGER` and that is `susi`:
```graphql
mutation {
  addVet(input: {
      firstName: "Dagmar",
      lastName: "Smith",
      specialtyIds: [1, 3]}) {

    ... on AddVetSuccessPayload {
      newVet: vet {
        id
        specialties {
          id
          name
        }
      }
    }

    ... on AddVetErrorPayload {
      error
    }
  }
}
```

Listen for new visits using a **Subscription**

Hint: open Graphiql in two browser tabs in parallel. In 1st window, run the following subscription,
in the 2nd tab create than a new Visit (see above for an example). The new Visit should automatically
be seen in 2nd tab, after the Mutation in 1st tab completes.

This mutation selects the treating veterinarian of the new created Visit and the pet that will be visiting.

```graphql

subscription {
    onNewVisit {
        description
        treatingVet {
            id
            firstName
            lastName
        }
        pet {
            id
            name
        }
    }

}
```

**Note**: The WebSocket/Subscription support in GraphiQL is far from being robust. Use with care!

![SpringBoot PetClinic, GraphiQL](graphiql.png)


# Contributing

Initial implementation of this GraphQL-based PetClinic example: https://github.com/spring-projects/spring-petclinic.git