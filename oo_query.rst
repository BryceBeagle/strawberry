Introduction
------------

The purpose of this document is to propose a syntax for using
``@strawberry.type`` objects to create GraphQL queries using object-oriented
patterns instead of strings.

Provided below is an example of a complicated GraphQL string-based query that
should be completely reproducible in this object-oriented system:

.. code-block:: graphql

    query QueryName {
        query1: functionField(argument: "value") {
            result1,
            result2
        }
        query2: books(author: "Daniel Kahneman", year: 2011) {
            author {
                name,
                age
            }
            title,
            isbn
        }
    }

This example has been designed to reflect a number of concepts

* Operation type (``query``)
* Operation name (``QueryName``)
* TODO: Operation arguments
* Fields with argument(s)
* Nested selection sets
* TODO: Fragments
* TODO: Directives
* TODO: Directive arguments
* TODO: Order-by
* Multiple queries in one request

Reference document from Apollo: https://www.apollographql.com/blog/the-anatomy-of-a-graphql-query-6dffa9e9e747/


Simple Query
------------

Using the schema example from the Strawberry docs:

.. code-block:: python

    import typing
    import strawberry

    @strawberry.type
    class Book:
      title: str
      author: 'Author'

    @strawberry.type
    class Author:
      name: str
      books: typing.List['Book']

    @strawberry.type
    class Query:
        books: typing.List['Book']
        authors: typing.List['Author']

A string-based query to retrieve information about all books and their others
would be:

.. code-block:: graphql

    query {
        books {
            title
            author {
                name
            }
        }
    }

A corresponding object-oriented query could be:
.. code-block:: python

    Query.books(
         [
            Book.title,
            Book.author[
                Author.name
            ]
         ]
    )
