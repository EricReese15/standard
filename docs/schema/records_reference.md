# Record Reference

Whereas there can be multiple releases about a contracting process, there should be a single **record** per contracting process, aggregating all the releases available for the contracting process.

**Note: If any conflicts are found between this text, and the text within the schema, the schema takes precedence.**

```{eval-rst}
.. admonition:: Browsing the schema
   :class: note

   .. markdown::

      This page presents the record package schema as tables. You can also download the canonical version of the record package schema as {download}`JSON Schema <../../build/current_lang/record-package-schema.json>`, or view it in an [interactive browser](record_package).
```

## Package metadata

Records must be published within a [record package](record_package). The record package provides metadata about the record(s) that it contains.

```{eval-rst}
.. jsonschema:: ../../build/current_lang/record-package-schema.json
    :include:
    :collapse: records
```

See the [licensing guidance](../../guidance/publish/#license-your-data) for more details on selecting a license, and publishing license information.

See the [publication policy](../../guidance/publish/#finalize-your-publication-policy) guidance for more details on what to include in a publication policy.

The record package metadata has two differences from the release package metadata:

* Instead of a `releases` array, a record package has a `records` array containing one or more records.
* A record package has a `packages` array, to link to any release packages that were used to prepare the records.

The following example demonstrates all package metadata and record fields.

```{eval-rst}
.. jsoninclude:: ../examples/merging/versioned.json
   :jsonpointer:
   :expand: packages, records
   :title: package
```

## Record structure

A record **must** contain an [ocid](../identifiers/#ocid) and all [releases](#releases) about the contracting process. As such, a record functions as an index of all releases about a contracting process.

A record **should** contain a [compiledRelease](#compiled-release) object, which represents the state of the contracting process at the time of the record's publication.

A record **may** contain a [versionedRelease](#versioned-release) object, which aggregates, into a single object, all values of all fields from all releases. The versioned release is designed to make it easy to see how values change from one release to another, and will often be generated by data users, rather than by publishers.

### Releases

Each release in a record can be provided as either a linked release or an embedded release.

#### Linked releases

A linked release follows a simple schema:

```{eval-rst}
.. jsonschema:: ../../build/current_lang/record-package-schema.json
   :pointer: /definitions/record/properties/releases/oneOf/0/items
```

For each `url` value, it must be possible for a consuming application to retrieve the release package at the URL and identify the release within it. Since a release package can contain multiple releases, for a linked release to identify a specific release via its `url` field, the `id` of the release must be appended to the release package URL using a fragment identifier.

The following example demonstrates the use of linked releases.

```{eval-rst}
.. jsoninclude:: ../examples/merging/versioned.json
   :jsonpointer: /records/0
   :expand: releases, tag
   :title: releases
```

Above, the first linked release has a `url` value of <https://standard.open-contracting.org/examples/releases/ocds-213czf-000-00002-01-award1.json#ocds-213czf-000-00002-01-award1>. The first part (`https://standard.open-contracting.org/examples/releases/ocds-213czf-000-00002-01-award1.json`) is the URL of the release package, and the fragment identifier (`ocds-213czf-000-00002-01-award1`) is the `id` of the release.

Release `id` values are only required to be unique within the scope of a contracting process: that is, within the scope of an `ocid` value. As such, a consuming application needs to use that fragment identifier in combination with the `ocid` of the record in order to identify the matching release within the release package.

#### Embedded releases

An embedded release follows the [release schema](reference). In other words, instead of linking to a release within a release package as above, that release is entirely included in the record.

The following example demonstrates the use of embedded releases.

```{eval-rst}
.. jsoninclude:: ../examples/record-embedded-releases.json
   :jsonpointer: /records/0
   :expand: releases,tag
   :title: releases
```

#### Comparing options

The considerations are:

* Using linked releases yields smaller file sizes than using embedded releases.
* From a user's perspective, the contents of releases are easier to access as embedded releases than as linked releases (which involves the retrieval process described above).
* From a publisher's perspective, using linked releases involves publishing release packages and constructing `url` values, whereas using embedded releases doesn't.

### Compiled release

The compiled release is the latest version of all the data about a contracting process. In other words, it provides a snapshot of the current state of a contracting process.

The compiled release follows the [release schema](reference), with the exception of any fields on which `"omitWhenMerged": true` is declared.

The process for creating a compiled release from individual releases is described in the [merging](merging) reference.

### Versioned release

A versioned release aggregates all values of all fields from all releases. For each field, it describes the history of values from the initial value to the current value, including the `id`, `date` and `tag` of the release in which it was changed.

This versioned information is relevant to many use cases relating to contract monitoring. However, it can significantly increase file sizes. As such, publishers can consider publishing records both with and without the versioned release.

If the versioned release is not provided, third parties can generate it by processing the record's releases according to the [merging](merging) reference.

The following example displays a single field's [versioned values](../merging/#versioned-values). This shows that the amount changed between the planning stage and the tender stage, while the currency did not.

```{eval-rst}
.. jsoninclude:: ../examples/merging/versioned.json
   :jsonpointer: /records/0/versionedRelease/tender/value
   :expand: amount, releaseTag
   :title: versioned
```
