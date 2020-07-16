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

* Explict operation type (``query``)
* Operation name (``QueryName``)
* TODO: Operation arguments
* Fields with argument(s)
* Nested selection sets
* TODO: Fragments
* TODO: Directives
* TODO: Directive arguments
* TODO: Order-by
* TODO: Multiple queries in one request
* TODO: Query aliases
* TODO: Unions

Reference document from Apollo: https://www.apollographql.com/blog/the-anatomy-of-a-graphql-query-6dffa9e9e747/

Queries
-------

Simple Query
++++++++++++

Using the schema example from the Strawberry docs:

.. code-block:: python

    from typing import List
    import strawberry

    @strawberry.type
    class Book:
      title: str
      author: 'Author'

    @strawberry.type
    class Author:
      name: str
      books: List['Book']

    @strawberry.query
    class Query:
        books: List['Book'] = strawberry.field(resolver=...)
        authors: List['Author'] = strawberry.field(resolver=...)

A string-based query to retrieve information about all books and their authors
would be:

.. code-block:: graphql

    {
        books {
            title
            author {
                name
            }
        }
    }

A corresponding object-oriented query could be:

.. code-block:: python

    query = Query.books({
        Book.title,
        Book.author({
            Author.name
        })
    })

    result = query.execute()

We execute the query ``books`` as a function. For now, only one argument will be
passed - a list of the fields that we want returned. Other, optional, arguments
will be discussed later. The fields will be the attributes that are defined in
our schema, as opposed to simple strings.

For now, interpret the callable signature of a query object, in this case
``Query.books`` as

.. code-block:: python

    def books(query: set, **kwargs) -> strawberry.Query:
        ...

The query parameter is always required. If the query takes arguments, they will
be within ``**kwargs``.  The next section will describe such a case.


Query with Arguments
++++++++++++++++++++

In section, the schema used will be:

.. code-block:: python

    from typing import List, Optional
    import strawberry

    @strawberry.type
    class Book:
      title: str
      author: 'Author'

    @strawberry.type
    class Author:
      name: str
      books: List['Book']

    @strawberry.query
    class Query:
        @strawberry.field
        def books(author: Optional[str], isbn: Optional[str]) -> List['Book']:
            return ...()
        authors: typing.List['Author']

``Query.books` is now a query that takes a couple of arguments.

A graphql string query that gets the information of all books from the author
``John Cena`` would be:

.. code-block:: graphql

     {
         books(author: "John Cena") {
             title
             author {
                 name
             }
         }
    }

The corresponding

.. code-block:: python

    query = Query.books(author="John Cena",
        fields={
            Book.title,
            Book.author({
                Author.name
            })
        }
    )

    result = query.execute()

Here, the ``author`` argument is provided before the ``fields`` argument, which
now uses an explicit keyword. [1]


[1] I was torn with leaving the ``fields`` argument at the beginning of the
arglist *without* a keyword or forcing a keyword argument so it can appear last.
I decided to go with the latter as I found it much harder to understand what's
happening if the query's arguments come *after* the ``fields`` set. A user could
still technically use the other order if they desire.
