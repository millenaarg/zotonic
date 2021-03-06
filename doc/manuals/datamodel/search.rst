.. highlight:: django
.. _manual-datamodel-query-model:

The Query search-model
======================

Using the query search API you can retrieve lists of resources in
various ways. The query search-model is a powerful search model that
accepts a lot of options. This page documents those options.

You access the query model in the following way::

  {% for id in m.search[{query (options go here...) }] %}

For instance, to select all news items, ordered by their modification
date, newest first::

  {% for id in m.search[{query cat='news' sort='-rsc.modified'}] %}
      {{ id }}
  {% endfor %}

Trying it out
-------------

Of course you can create your own ``for``-loop in a template, but
there are easier ways to check out the inner workings of the
query-model: through your browser.

The query-model is exposed to the browser in (currently) 2 URLs: the
Atom feed module for creating a customized update feed, and the API
for receiving lists of ids in JSON.

Get all resource of the "documentation" category on zotonic.com::

  http://zotonic.com/api/search?cat=documentation

Get a feed of most recent documentation containing the word "filter"::

  http://zotonic.com/feed/search?cat=documentation&text=filter

.. note::
   
   ``mod_atom_feed`` automatically sorts on last-modified date,
   ``api/search`` doesn't.


Query-model arguments
---------------------

**authoritative**

  Boolean, filters whether a resource is considered "authoritative"
  (belonging on this site) or not.

  ``authoritative=1``

**cat**

  Filter resources on a specific category. Specifying multiple 'cat'
  arguments will do an "or" on the categories.

  ``cat='news'``

**cat_exact**

  Filter resources to include the given category, but exclude any subcategory

  ``cat_exact='news'``

**cat_exclude**

  Filter resources to exclude the given category.

  ``cat_exclude='meta'``

**id_exclude**

  Filter resources to exclude the ones with the given ids.

  ``id_exclude=123``

**filter**

  Filtering on columns.

  ``filter=['pivot_title', 'Hello']``

  In its most simple form, this does an 'equals' compare filter. The
  "filter" keywords expects a list. If the list is two elements long,
  we expect the first column to be the filter column name from the
  database table, and the second column name to be the filter value.

  ``filter=['numeric_value', `gt`, 10]``
  
  If the filter is a three-column list, the second column is the
  operator. This must be an atom (surround it in backquotes!) and must
  be one of the following: ``eq``, ``ne``, ``gt``, ``gte``, ``lt``,
  ``lte``; or one of ``=``, ``<>``, ``>``, ``>=``, ``<``, ``<=``:

  ``filter=['numeric_value', `>`, 10]``
  
**hassubject**

  Select all resources that have an outgoing connection to the given
  page, which is specified by the argument (the page id 123 in the
  example, or the unique page name 'tag_gift'). Optionally, you can
  pass the name of a predicate as the second argument, to specify that
  the connection should have this predicate. Specifying this multiple
  times does an "and" of the conditions.

  ``hassubject=123``
  ``hassubject='tag_gift'``
  ``hassubject=[123,'author']``

**hasobject**

  Like hassubject, but selects all pages that have an incoming
  connection to the given page, which is specified by the
  argument. Optionally, you can pass the name of a predicate as the
  second argument, to specify that the connection should have this
  predicate.

  ``hasobject=123``
  ``hasobject='tag_gift'``
  ``hasobject=[123,'hasdocument']``

**hasanyobject**

  Like hasobject, but allows to define an "or" operation on the edge.
  You can define multiple combinations of predicates and objects, any
  resource having such an outgoing edge will be matched.

  ``hasanyobject=1,2,3``
  ``hasanyobject=[1,author],2,3``
  ``hasanyobject=[*,author],2,3``

  The term ``[*,author],2,3`` selects any resource with an *author*,
  or a connection to the ids 2 or 3 (with any predicate).

**is_featured**

  A boolean option that specifies if a page should be featured or not.

  ``is_featured``

**is_published**

  Select published, unpublished or omit the publish check. Legal
  values are true, false or all.

  ``is_published='all'``

**is_public**

  Filter on whether an item is publicly visible or not. Valid values
  are 'true', 'false', 'all'.

  ``is_public='false'``

**upcoming**

  Specifying 'upcoming' means that you only want to select things that
  have a start date which lies in the future. Like the name says,
  useful to select upcoming events.

  ``upcoming``

**ongoing**

  Specifying 'ongoing' means that you only want to select things that
  are happening now: that have a start date which lies in the past,
  and an end date which lies in the future.

  ``ongoing``

**finished**

  Specifying 'finished' means that you only want to select things that
  have a start date which lies in the past. 

  ``finished``

**sort**

  Sort the result on a field. The name of the field is a string which
  directly refers to the sql-join that is being used. If you specify a
  dash ("-") in front of the field, the order is descending. Leaving
  this out or specifying a "+" means ascending.

  Some sort fields:

  - ``rsc.modified`` - date of last modification
  - ``rsc.pivot_date_start`` - the start date specified in the admin
  - ``rsc.pivot_date_end`` - the end date specified in the admin
  - ``rsc.pivot_title`` - the title of the page. When making
    multilingual sites, the behavior of sorting on title is undefined.

  For all the sort fields, you will have to consult Zotonic’s data
  model. Example sorting on modification date, newest first:

  ``sort='-rsc.modified'``

**custompivot**

  Add a join on the given custom pivot table. The table is joined to
  the primary ``rsc`` table.

  ``custompivot=foo``
  (joins the ``pivot_foo`` table into the query)

  The pivot tables are aliassed with a number in order of their
  occurrence, with the first pivot table aliassed as ``pivot1``. This
  allows you to do filtering on custom fields like this:

  ``{query custompivot="pivotname" filter=["pivot1.fieldname", `=`, "hello"]}``


**hasobjectpredicate**

  Filter on all things which have any outgoing edge with the given
  predicate.

  ``hasobjectpredicate='hasdocument'``

**hassubjectpredicate**

  Filter on all things which have any incoming edge with the given
  predicate.

  ``hassubjectpredicate='author'``

**text**

  Perform a fulltext search on the primary "rsc" table. The result
  will automatically be ordered on the relevancy (rank) of the result.

  ``text="test"``

**query_id**

  Load the query arguments from the saved ``query`` resource. 

  ``query_id=331``

  .. seealso:: :ref:`manual-query-resources`

**publication_month**

  Filter on month of publication date

  ``publication_month=9``

**publication_year**

  Filter on year of publication date

  ``publication_year=2012``

**date_start_after**

  Select items with a start date greater than given value

  ``date_start_after="2010-01-15"``

  It also possible to use relative times:

   * ``now``
   * ``+0 sunday``  (last sunday or the current sunday)
   * ``+0 monday``  (last monday or the current monday)
   * ``+1 minute``
   * ``+1 hour``
   * ``+1 day``
   * ``+1 week``
   * ``+1 month``
   * ``+1 year``

  Negative offsets are allowed as well. There //must// be a ``+`` or ``-`` sign.

**date_start_before**

  Select items with a start date smaller than given value

  ``date_start_before="2010-01-15"``

**date_start_year**

  Select items with an "event start date" in the given year.

  ``date_start_year=2012``

**date_end_after**

  Select items with a end date greater than given value

  ``date_end_after="2010-01-15"``

**date_end_before**

  Select items with a end date smaller than given value

  ``date_end_before="2010-01-15"``

**date_end_year**

  Select items with an "event end date" in the given year.

  ``date_end_year=2012``

**content_group**

  Select items which are member of the given content group (or one of its children).

  ``content_group=public``

  
Filter behaviour
----------------

All of the filters works as ``AND`` filter. The only exception to this
is the ``cat=`` filter: if you specify multiple categories, those
categories are "OR"'ed together, to allow to search in multiple
distinct categories with a single search query.
  


.. _manual-query-resources:

Query resources
---------------

Query resources are, as the name implies,
:ref:`manual-datamodel-resources` of the special category `query`. In
the admin this category is called "search query". it is basically a
stored (and thus content manageable) search query. You create an
editable search query in an admin page that then is invoked from a
template.

When creating such a resource in the page, you will see on the admin
edit page an extra text field in which you can add search terms. Each
search term goes on its own line, and the possible search terms are
equal to the ones described on this page (the `Query-model
arguments`).
