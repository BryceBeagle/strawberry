Introduction
------------

The purpose of this document is to propose a syntax for using
``@strawberry.type`` objects to construct GraphQL queries using object-oriented
patterns instead of strings.

Provided below is an example of a complicated GraphQL string-based query that
should be completely reproducible in this object-oriented system:

.. code-block:: graphql

    query DanLibrary($author: Author, $extraInfo: Boolean!) {
        BooksByAuthor: books(author: $author, includePoems: true) {
            author {
                fullName: name,
                age
            }
            title,
            isbn @skip(if: $extraInfo)
        }
        MagazinesByAuthor: magazines(author: $author) {
            publisher
        }
    }

This example has been designed to reflect a number of concepts

* Explict operation type (``query``)
* Operation name (``QueryName``)
* Operation parameters
* Fields with parameter(s)
* Nested selection sets
* Multiple queries in one request
* Directives
* Field aliases
* TODO: Fragments
* TODO: Unions

This document will introduce these concepts gradually and demonstrate how an
object-oriented approach could represent all of them.

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

    @strawberry.type
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
passed - a set [1] of the fields that we want returned. Other, optional, arguments
will be discussed later. The fields will be the attributes that are defined in
our schema, as opposed to simple strings.

For now, interpret the callable signature of a query object, in this case
``Query.books`` as

.. code-block:: python

    def books(fields: set, **kwargs) -> strawberry.CompiledQuery:
        ...

The ``fields`` parameter is always required. If the resolver takes arguments,
they will be provided within ``**kwargs``.  The next section will describe such
a case.

``strawberry.CompiledQuery`` is used here as a placeholder for the query
construction's returned object. It will store the query and can be used using
its ``.execute()`` method. If the query itself takes arguments [2], they will be
provided to ``.execute()``.


[1] I'm still not sure if sets or lists would be better. Sets use the same
symbols as the string-based queries (``{}``), and fields must be unique, but are
not ordered. Are GraphQL results guaranteed to be returned in the same order
they're requested?

[2] I have yet to work out how adding parameters will be described with the
OO-based patterns.


Resolvers with Parameters
+++++++++++++++++++++++++

In this section, the schema used for reference will be:

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

``Query.books`` is now a resolver that has a couple of parameters. Note that
while ``Query.books`` is now given a signature here, ``fields`` and the other
arguments are added behind the scenes when we actually want to build a query.

A GraphQL string query that gets the information of all books from the author
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
now uses an explicit keyword [1]. Everything else remains the same.


[1] I was torn with leaving the ``fields`` argument at the beginning of the
arglist *without* a keyword or forcing a keyword argument so it can appear last.
I decided to go with the latter as I found it much harder to understand what's
happening if the query's arguments come *after* the ``fields`` set. A user could
still technically use the other order if they desire.
