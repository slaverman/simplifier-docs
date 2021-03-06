.. _restful:

FHIR RESTful API
================

Vonk supports most of the features in the `FHIR RESTful API <http://www.hl7.org/implement/standards/fhir/http.html>`_.

.. _restful_crud:

Create, read, update, delete
----------------------------

These four operations to manage the contents of the Vonk FHIR Server, commonly referenced by the acronym CRUD, are implemented as per the specification.
This includes version-read and the conditional variations. 
Only a few limitations apply.

Vonk enables create-on-update: If you request an update and no resource exists for the given id, the provided resource will be created under the provided id.

Vonk can reject a resource based on :ref:`feature_prevalidation`.

.. _restful_crud_limitations:

Limitations on CRUD
^^^^^^^^^^^^^^^^^^^

#. ``_summary`` is not yet supported.
#. Simultaneous conditional creates and updates are not entirely transactionally safe:
   
   * Two conditional updates may both result in a ``create``, although the result of one may be a match to the other.
   * Two conditional creates may both succeed, although the result of one may be a match to the other.
   * A conditional create and a simultaneous conditional update may both result in a ``create``, although the result of one may be a match to the other.

.. _restful_versioning:

Versioning
----------

Vonk keeps a full version history of every resource, including the resources on the :ref:`administration_api`.

.. _restful_search:

Search
------

Search is supported as per the specification, with a few :ref:`restful_search_limitations`.

In the default configuration the SearchParameters from the `FHIR specification <http://www.hl7.org/implement/standards/fhir/searchparameter-registry.html>`_ 
are available. But Vonk also allows :ref:`feature_customsp`. 

Chaining and reverse chaining is fully supported.

Quantity search on UCUM quantities automatically converts units to a canonical form. This means you can have kg in an Observation and search by lbs, or vice versa.

.. _restful_search_limitations:

Limitations on search
^^^^^^^^^^^^^^^^^^^^^

The following parameters and options are not yet supported:

#. ``_text``
#. ``_content``
#. ``_query``
#. ``_containedType``
#. ``_filter``
#. ``:approx`` modifier on a quantity SearchParameter
#. ``:text`` modifier on a string SearchParameter
#. ``:above``, ``:below``, ``:in``, ``:not-in`` modifiers on a token SearchParameter
#. ``:above`` on a uri SearchParameter (``:below`` *is* supported)
#. ``:recurse`` modifier on ``_include`` and ``_revinclude``.

Furthermore:

#. ``_sort`` is only implemented for the parameter ``_lastUpdated`` in order to support History.
#. Paging is supported, but it is not isolated from intermediate changes to resources.

.. _restful_history:

History
-------

History is supported as described in the specification, on the system, type and instance level.
The ``_since`` and ``_count`` parameters are also supported.

.. _restful_history_limitations:

Limitations on history
^^^^^^^^^^^^^^^^^^^^^^

#. ``_at`` parameter is not yet supported.
#. Paging is supported, but it is not isolated from intermediate changes to resources.

.. _restful_batch:

Batch
-----

Batch is fully supported. Although the specification does not allow references between resources in a Batch, Vonk can handle them.

.. _restful_transaction:

Transaction
-----------

Transactions are supported, with a single limitation:

#. Of the three storage implementations, only SQL Server truly supports transactions. On :ref:`MongoDB<configure_mongodb>` and :ref:`Memory<configure_memory>`, transaction support can be simulated at the FHIR level, but not be enforced on the database level.
#. References between resources in the transaction can only point backwards. So if resource B references A, A must be created or updated before B. This implies that circular references are also not supported. 

.. _restful_capabilities:

Capabilities
------------

On the Capabilities interaction (``<vonk-endpoint>/metadata``) Vonk returns a CapabilityStatement that is built dynamically from the 
supported ResourceTypes, SearchParameters and interactions. E.g. if you :ref:`feature_customsp_configure`, the SearchParameters that are actually loaded appear in the CapabilityStatement.

.. _restful_notsupported:

Not supported interactions
--------------------------

These interactions are not yet supported by Vonk:

#. patch
