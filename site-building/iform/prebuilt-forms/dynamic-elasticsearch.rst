Elasticsearch outputs (customisable)
====================================

This prebuilt page is designed to provide a flexible combination of outputs from an
Elasticsearch instance, if one is configured and linked to the warehouse according to the
instructions at https://github.com/Indicia-Team/support_files/tree/master/Elasticsearch.
This approach can provide an extremely high performance reporting option where a warehouse
holds multiple millions of records. It also contains controls for viewing record details
and performing verification actions, providing an alternative approach to record
verification.

If you already have configured access to Elasticsearch via the warehouse's REST API, you
should know or be able to find out the following settings:

  * Endpoint - the path within the REST API's address which refers to your Elasticsearch
    access alias.
  * User - a code for your user in the REST API configuration.
  * Secret - a secret given for your user's access to the REST API Elasticsearch endpoint.
  * Warehouse ID prefix - the prefix inserted before occurrence ID to make a globally
    unique ID on the Elasticsearch cluster.

If you do not have the above available and therefore need to set up the REST API access
on the warehouse, then follow the instructions at :doc:`../../../developing/rest-web-services/elasticsearch`.

It would be beneficial to have a basic understanding of the Elasticsearch Query API before
proceeding.

Form configuration
------------------

The page offers standard configuration options for all Indicia pages powered by the IForm
module. The additional specific settings are described below.

Elasticsearch settings
^^^^^^^^^^^^^^^^^^^^^^

Start by filling in your Elasticsearch options as described above on the *Elasticsearch
settings* section of the Edit tab.

Filter settings
^^^^^^^^^^^^^^^

Not yet implemented. Currently, you can control page filtering by adding hidden inputs to
the Form Structure.

Permission settings
^^^^^^^^^^^^^^^^^^^

Here you can defined the name of the Drupal permission which a user must have in order to
be able to access the following kinds of data:

* My records download permission (records are limited to the records input by the logged
  in user).
* All records download permission (any records exposed by the Elasticsearch index alias
  configured on this page are available).
* Records in collated locality download permission (if a field called
  `field_location_collation` is in the user profile which contains a numeric ID of an
  indexed location, then limits the available records to those which intersect that
  location. Can be used to allow users with regional permissions to access the records in
  their area such as a Local Environmental Records Centre.)

Note that the default behaviour of an Elasticsearch page will be to request all records.
Therefore if your user does not have the permission you set for all records they will
not be able to access any data. If you want to use one of the permissions options other
than all records you can make the page request data with the appropriate permissions
filter by adding the `[permissionFilters]` control to the page. This outputs a drop down
control that allows the user to select from whichever options they have permission to use.
If there is only one option then the control is hidden.

User Interface
~~~~~~~~~~~~~~

The content of the generated page is entirely driven by the *User interface > Form
structure* configuration. Like other Indicia customisable page types, the *Form Structure*
is a text area in which you can place tokens which are replaced by controls when the page
is viewed. You can intersperse the control tokens with HTML to build a custom layout if
required.

Controls are represented by their name in square brackets and must be the only thing on
their line. They are followed by any number of parameters on the subsequent lines which
start with @, are followed by the property name then equals then the value. The list of
properties associated with a control stops as soon as something else is found which is not
a property definition. Property names and values are normally on one line but if a large
property value is required you can wrap the value in an XML comment (<!-- ... -->). For
example::

  [myControl]
  @myProperty=foo
  @myLongProperty=<!--
    bar
    baz
  -->

The controls available for addition to the page are as follows:

[source]
""""""""

The `source` control acts as a link from other controls on the page to a set of data from
Elasticsearch. See :ref:`elasticsearch-report-helper-source`.

[indiciaSource]
"""""""""""""""

.. todo::

  Implement an indiciaSource control to make this code data source independent.

[dataGrid]
""""""""""

Generates a table containing Elasticsearch data. See
:ref:`elasticsearch-report-helper-dataGrid`.

[download]
""""""""""

A button with associated progress display for generating downloadable zip files of CSV
data from an associated [source] control. See
:ref:`elasticsearch-report-helper-download`.

[higherGeographySelect]
"""""""""""""""""""""""

A select box for choosing from a list of higher geography boundaries. See
:ref:`elasticsearch-report-helper-higherGeographySelect`.

[leafletMap]
""""""""""""

A map panel which uses the leaflet library that can display occurrence data from
Elasticsearch in a variety of ways. See
:ref:`elasticsearch-report-helper-leafletMap`.

[permissionFilters]

Output a selector for various high level permissions filtering options. See
:ref:`elasticsearch-report-helper-permissionFilters`.

[recordDetails]
"""""""""""""""

A tabbed panel showing key details of the record. See
:ref:`elasticsearch-report-helper-recordDetails`.

[standardParams]
""""""""""""""""

A toolbar allowing filters to be applied to the page's report data. See
:ref:`elasticsearch-report-helper-standardParams`.

[templatedOutput]
"""""""""""""""""

A flexible output of ES data which uses templates to build the HTML. See
:ref:`elasticsearch-report-helper-templatedOutput`.

[userFilters]
"""""""""""""

Provides a drop down populated with the user's saved report filters. Selecting a filter
applies that filter to the current page's outputs. See
:ref:`elasticsearch-report-helper-userFilters`.

[urlParams]
"""""""""""

This control allows you to configure how the page uses parameters in the URL to filter the
output shown on the page. See
:ref:`elasticsearch-report-helper-urlParams`.

[verificationButtons]
"""""""""""""""""""""
Outputs a panel containing action buttons for verification tasks. See
:ref:`elasticsearch-report-helper-verificationButtons`.

Adding filters to the page using HTML hidden inputs
---------------------------------------------------

It is possible to define filters for the entire page by adding hidden inputs to the form
structure which define each filter value. These filters can be hidden with a fixed value
that is always applied, or visible, e.g. a text input that the user can use to search. We
use HTML5 data attributes to define the filter behaviour and set the class to
`es-filter-param` so the code detects changes to the value and applied the filter to the
report output.

The data attributes you can specify are:

  * data-es-bool-clause - set to `must`, `must_not`, `should` or `filter`. Normally you
    will set this to `must` or `must_not` but see `Elasticsearch Bool Query documentation
    <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html>`_
    for more info.
  * data-es-field - when this filter applies to a specific field, set to the Elasticsearch
    field name you want to filter against.
  * data-es-query-type - set to one of the following:

      match_all
      match_none
      term
      terms
      match
      match_phrase
      match_phrase_prefix
      query_string
      simple_query_string

    All of the above map to the query with the same name in the `Elasticsearch Query DSL
    documentation <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html>`_.

Some examples follow:

.. code-block:: HTML

  <!--A text input for search-->
  <input type="text"
    id="filter-search"
    class="es-filter-param"
    data-es-bool-clause="must"
    data-es-query-type="query_string" />

  <!--Record status filter select-->
  <label for="filter-status">Status:</label>
  <select id="filter-status" class="es-filter-param" data-es-bool-clause="must" data-es-query-type="query_string">
    <option value="NOT identification.verification_status:R">All records excluding not accepted</option>
    <option value="">All records</option>
    <option value="identification.verification_status:C AND identification.verification_substatus:0 AND NOT identification.query:Q">Pending</option>
    <option value="identification.verification_status:V">Accepted</option>
    <option value="identification.verification_status:R">Not accepted</option>
    <option value="identification.verification_status:C AND identification.verification_substatus:3">Plausible</option>
    <option value="identification.verification_status:C AND identification.query:Q">Queried</option>
    <option value="identification.verification_status:C AND identification.query:A">Answered</option>
  </select>


*Filter controls*

*HTML inputs*

Attributes, diff query types
    $fieldQueryTypes = ['term', 'match', 'match_phrase', 'match_phrase_prefix'];
    $stringQueryTypes = ['query_string', 'simple_query_string'];

* data-es-nested for nested fields.
* data-es-query
* data-es-bool-clause



