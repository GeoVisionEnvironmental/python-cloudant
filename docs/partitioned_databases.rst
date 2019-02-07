=====================
Partitioned Databases
=====================

Partitioned databases introduce the ability for a user to create logical groups
of documents called partitions by providing a partition key with each document.

.. warning:: Your Cloudant cluster must have the ``partitions`` feature enabled.
             A full list of enabled features can be retrieved by calling the
             client :func:`~cloudant.client.CouchDB.metadata` method.

Creating a partitioned database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    db = client.create_database('mydb', partitioned=True)

Handling documents
^^^^^^^^^^^^^^^^^^

The document ID contains both the partition key and document key in the form
``<partitionkey>:<documentkey>`` where:

- Partition Key *(string)*. Must be non-empty. Must not contain colons (as this
  is the partition key delimiter) or begin with an underscore.
- Document Key *(string)*. Must be non-empty. Must not begin with an underscore.

Be aware that ``_design`` documents and ``_local`` documents must not contain a
partition key as they are global definitions.

**Create a document**

.. code-block:: python

    partition_key = 'Year2'
    document_key = 'julia30'
    db.create_document({
        '_id': ':'.join((partition_key, document_key)),
        'name': 'Jules',
        'age': 6
    })

**Get a document**

.. code-block:: python

    doc = db[':'.join((partition_key, document_key))]

Creating design documents
^^^^^^^^^^^^^^^^^^^^^^^^^

To define partitioned indexes you must set the ``partitioned=True`` optional
when constructing the new ``DesignDocument`` class.

.. code-block:: python

    ddoc = DesignDocument(db, document_id='view', partitioned=True)
    ddoc.add_view('myview','function(doc) { emit(doc.foo, doc.bar); }')
    ddoc.save()

Similarly, to define a partitioned Cloudant Query index you must set the
``partitioned=True`` optional.

.. code-block:: python

    index = db.create_query_index(
        design_document_id='query',
        index_name='foo-index',
        fields=['foo'],
        partitioned=True
    )
    index.create()

Querying data
^^^^^^^^^^^^^

A partition key can be specified when querying data so that results can be
constrained to a specific database partition.

.. warning:: To run partitioned queries the database itself must be partitioned.

**Query**

.. code-block:: python

    results = self.db.get_partitioned_query_result(
        partition_key, selector={'foo': {'$eq': 'bar'}})

    for result in results:
        ...

See :func:`~cloudant.database.CouchDatabase.get_partitioned_query_result` for a
full list of supported parameters.

**Search**

.. code-block:: python

    results = self.db.get_partitioned_search_result(
        partition_key, search_ddoc['_id'], 'search1', query='*:*')

    for result in results['rows']:
        ....

See :func:`~cloudant.database.CloudantDatabase.get_partitioned_search_result`
for a full list of supported parameters.

**Views (MapReduce)**

.. code-block:: python

    results = self.db.get_partitioned_view_result(
        partition_key, view_ddoc['_id'], 'view1')

    for result in results:
        ....

See :func:`~cloudant.database.CouchDatabase.get_partitioned_view_result` for a
full list of supported parameters.