.. highlight:: text

.. _version-specifiers:

==================
Version specifiers
==================


This specification describes a scheme for identifying versions of Python software
distributions, and declaring dependencies on particular versions.


Definitions
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this
document are to be interpreted as described in :rfc:`2119`.

"Build tools" are automated tools intended to run on development systems,
producing source and binary distribution archives. Build tools may also be
invoked by integration tools in order to build software distributed as
sdists rather than prebuilt binary archives.

"Index servers" are active distribution registries which publish version and
dependency metadata and place constraints on the permitted metadata.

"Publication tools" are automated tools intended to run on development
systems and upload source and binary distribution archives to index servers.

"Installation tools" are integration tools specifically intended to run on
deployment targets, consuming source and binary distribution archives from
an index server or other designated location and deploying them to the target
system.

"Automated tools" is a collective term covering build tools, index servers,
publication tools, integration tools and any other software that produces
or consumes distribution version and dependency metadata.


Version scheme
==============

Distributions are identified by a public version identifier which
supports all defined version comparison operations

The version scheme is used both to describe the distribution version
provided by a particular distribution archive, as well as to place
constraints on the version of dependencies needed in order to build or
run the software.


.. _public-version-identifiers:

Public version identifiers
--------------------------

The canonical public version identifiers MUST comply with the following
scheme::

    [N!]N(.N)*[{a|b|rc}N][.postN][.devN]

Public version identifiers MUST NOT include leading or trailing whitespace.

Public version identifiers MUST be unique within a given distribution.

Installation tools SHOULD ignore any public versions which do not comply with
this scheme but MUST also include the normalizations specified below.
Installation tools MAY warn the user when non-compliant or ambiguous versions
are detected.

See also :ref:`version-specifiers-regex` which provides a regular
expression to check strict conformance with the canonical format, as
well as a more permissive regular expression accepting inputs that may
require subsequent normalization.

Public version identifiers are separated into up to five segments:

* Epoch segment: ``N!``
* Release segment: ``N(.N)*``
* Pre-release segment: ``{a|b|rc}N``
* Post-release segment: ``.postN``
* Development release segment: ``.devN``

Any given release will be a "final release", "pre-release", "post-release" or
"developmental release" as defined in the following sections.

All numeric components MUST be non-negative integers represented as sequences
of ASCII digits.

All numeric components MUST be interpreted and ordered according to their
numeric value, not as text strings.

All numeric components MAY be zero. Except as described below for the
release segment, a numeric component of zero has no special significance
aside from always being the lowest possible value in the version ordering.

.. note::

   Some hard to read version identifiers are permitted by this scheme in
   order to better accommodate the wide range of versioning practices
   across existing public and private Python projects.

   Accordingly, some of the versioning practices which are technically
   permitted by the specification are strongly discouraged for new projects. Where
   this is the case, the relevant details are noted in the following
   sections.


.. _local-version-identifiers:

Local version identifiers
-------------------------

Local version identifiers MUST comply with the following scheme::

    <public version identifier>[+<local version label>]

They consist of a normal public version identifier (as defined in the
previous section), along with an arbitrary "local version label", separated
from the public version identifier by a plus. Local version labels have
no specific semantics assigned, but some syntactic restrictions are imposed.

Local version identifiers are used to denote fully API (and, if applicable,
ABI) compatible patched versions of upstream projects. For example, these
may be created by application developers and system integrators by applying
specific backported bug fixes when upgrading to a new upstream release would
be too disruptive to the application or other integrated system (such as a
Linux distribution).

The inclusion of the local version label makes it possible to differentiate
upstream releases from potentially altered rebuilds by downstream
integrators. The use of a local version identifier does not affect the kind
of a release but, when applied to a source distribution, does indicate that
it may not contain the exact same code as the corresponding upstream release.

To ensure local version identifiers can be readily incorporated as part of
filenames and URLs, and to avoid formatting inconsistencies in hexadecimal
hash representations, local version labels MUST be limited to the following
set of permitted characters:

* ASCII letters (``[a-zA-Z]``)
* ASCII digits (``[0-9]``)
* periods (``.``)

Local version labels MUST start and end with an ASCII letter or digit.

Comparison and ordering of local versions considers each segment of the local
version (divided by a ``.``) separately. If a segment consists entirely of
ASCII digits then that section should be considered an integer for comparison
purposes and if a segment contains any ASCII letters then that segment is
compared lexicographically with case insensitivity. When comparing a numeric
and lexicographic segment, the numeric section always compares as greater than
the lexicographic segment. Additionally a local version with a great number of
segments will always compare as greater than a local version with fewer
segments, as long as the shorter local version's segments match the beginning
of the longer local version's segments exactly.

An "upstream project" is a project that defines its own public versions. A
"downstream project" is one which tracks and redistributes an upstream project,
potentially backporting security and bug fixes from later versions of the
upstream project.

Local version identifiers SHOULD NOT be used when publishing upstream
projects to a public index server, but MAY be used to identify private
builds created directly from the project source. Local
version identifiers SHOULD be used by downstream projects when releasing a
version that is API compatible with the version of the upstream project
identified by the public version identifier, but contains additional changes
(such as bug fixes). As the Python Package Index is intended solely for
indexing and hosting upstream projects, it MUST NOT allow the use of local
version identifiers.

Source distributions using a local version identifier SHOULD provide the
``python.integrator`` extension metadata (as defined in :pep:`459`).


Final releases
--------------

A version identifier that consists solely of a release segment and optionally
an epoch identifier is termed a "final release".

The release segment consists of one or more non-negative integer
values, separated by dots::

    N(.N)*

Final releases within a project MUST be numbered in a consistently
increasing fashion, otherwise automated tools will not be able to upgrade
them correctly.

Comparison and ordering of release segments considers the numeric value
of each component of the release segment in turn. When comparing release
segments with different numbers of components, the shorter segment is
padded out with additional zeros as necessary.

While any number of additional components after the first are permitted
under this scheme, the most common variants are to use two components
("major.minor") or three components ("major.minor.micro").

For example::

    0.9
    0.9.1
    0.9.2
    ...
    0.9.10
    0.9.11
    1.0
    1.0.1
    1.1
    2.0
    2.0.1
    ...

A release series is any set of final release numbers that start with a
common prefix. For example, ``3.3.1``, ``3.3.5`` and ``3.3.9.45`` are all
part of the ``3.3`` release series.

.. note::

   ``X.Y`` and ``X.Y.0`` are not considered distinct release numbers, as
   the release segment comparison rules implicit expand the two component
   form to ``X.Y.0`` when comparing it to any release segment that includes
   three components.

Date-based release segments are also permitted. An example of a date-based
release scheme using the year and month of the release::

    2012.4
    2012.7
    2012.10
    2013.1
    2013.6
    ...


.. _pre-release-versions:

Pre-releases
------------

Some projects use an "alpha, beta, release candidate" pre-release cycle to
support testing by their users prior to a final release.

If used as part of a project's development cycle, these pre-releases are
indicated by including a pre-release segment in the version identifier::

    X.YaN   # Alpha release
    X.YbN   # Beta release
    X.YrcN  # Release Candidate
    X.Y     # Final release

A version identifier that consists solely of a release segment and a
pre-release segment is termed a "pre-release".

The pre-release segment consists of an alphabetical identifier for the
pre-release phase, along with a non-negative integer value. Pre-releases for
a given release are ordered first by phase (alpha, beta, release candidate)
and then by the numerical component within that phase.

Installation tools MAY accept both ``c`` and ``rc`` releases for a common
release segment in order to handle some existing legacy distributions.

Installation tools SHOULD interpret ``c`` versions as being equivalent to
``rc`` versions (that is, ``c1`` indicates the same version as ``rc1``).

Build tools, publication tools and index servers SHOULD disallow the creation
of both ``rc`` and ``c`` releases for a common release segment.


Post-releases
-------------

Some projects use post-releases to address minor errors in a final release
that do not affect the distributed software (for example, correcting an error
in the release notes).

If used as part of a project's development cycle, these post-releases are
indicated by including a post-release segment in the version identifier::

    X.Y.postN    # Post-release

A version identifier that includes a post-release segment without a
developmental release segment is termed a "post-release".

The post-release segment consists of the string ``.post``, followed by a
non-negative integer value. Post-releases are ordered by their
numerical component, immediately following the corresponding release,
and ahead of any subsequent release.

.. note::

   The use of post-releases to publish maintenance releases containing
   actual bug fixes is strongly discouraged. In general, it is better
   to use a longer release number and increment the final component
   for each maintenance release.

Post-releases are also permitted for pre-releases::

    X.YaN.postM   # Post-release of an alpha release
    X.YbN.postM   # Post-release of a beta release
    X.YrcN.postM  # Post-release of a release candidate

.. note::

   Creating post-releases of pre-releases is strongly discouraged, as
   it makes the version identifier difficult to parse for human readers.
   In general, it is substantially clearer to simply create a new
   pre-release by incrementing the numeric component.


Developmental releases
----------------------

Some projects make regular developmental releases, and system packagers
(especially for Linux distributions) may wish to create early releases
directly from source control which do not conflict with later project
releases.

If used as part of a project's development cycle, these developmental
releases are indicated by including a developmental release segment in the
version identifier::

    X.Y.devN    # Developmental release

A version identifier that includes a developmental release segment is
termed a "developmental release".

The developmental release segment consists of the string ``.dev``,
followed by a non-negative integer value. Developmental releases are ordered
by their numerical component, immediately before the corresponding release
(and before any pre-releases with the same release segment), and following
any previous release (including any post-releases).

Developmental releases are also permitted for pre-releases and
post-releases::

    X.YaN.devM       # Developmental release of an alpha release
    X.YbN.devM       # Developmental release of a beta release
    X.YrcN.devM      # Developmental release of a release candidate
    X.Y.postN.devM   # Developmental release of a post-release

Do note that development releases are considered a type of pre-release when
handling them.

.. note::

   While they may be useful for continuous integration purposes, publishing
   developmental releases of pre-releases to general purpose public index
   servers is strongly discouraged, as it makes the version identifier
   difficult to parse for human readers. If such a release needs to be
   published, it is substantially clearer to instead create a new
   pre-release by incrementing the numeric component.

   Developmental releases of post-releases are also strongly discouraged,
   but they may be appropriate for projects which use the post-release
   notation for full maintenance releases which may include code changes.


Version epochs
--------------

If included in a version identifier, the epoch appears before all other
components, separated from the release segment by an exclamation mark::

    E!X.Y  # Version identifier with epoch

If no explicit epoch is given, the implicit epoch is ``0``.

Most version identifiers will not include an epoch, as an explicit epoch is
only needed if a project *changes* the way it handles version numbering in
a way that means the normal version ordering rules will give the wrong
answer. For example, if a project is using date based versions like
``2014.04`` and would like to switch to semantic versions like ``1.0``, then
the new releases would be identified as *older* than the date based releases
when using the normal sorting scheme::

    1.0
    1.1
    2.0
    2013.10
    2014.04

However, by specifying an explicit epoch, the sort order can be changed
appropriately, as all versions from a later epoch are sorted after versions
from an earlier epoch::

    2013.10
    2014.04
    1!1.0
    1!1.1
    1!2.0


.. _version-specifiers-normalization:

Normalization
-------------

In order to maintain better compatibility with existing versions there are a
number of "alternative" syntaxes that MUST be taken into account when parsing
versions. These syntaxes MUST be considered when parsing a version, however
they should be "normalized" to the standard syntax defined above.


Case sensitivity
~~~~~~~~~~~~~~~~

All ascii letters should be interpreted case insensitively within a version and
the normal form is lowercase. This allows versions such as ``1.1RC1`` which
would be normalized to ``1.1rc1``.


Integer Normalization
~~~~~~~~~~~~~~~~~~~~~

All integers are interpreted via the ``int()`` built in and normalize to the
string form of the output. This means that an integer version of ``00`` would
normalize to ``0`` while ``09000`` would normalize to ``9000``. This does not
hold true for integers inside of an alphanumeric segment of a local version
such as ``1.0+foo0100`` which is already in its normalized form.


Pre-release separators
~~~~~~~~~~~~~~~~~~~~~~

Pre-releases should allow a ``.``, ``-``, or ``_`` separator between the
release segment and the pre-release segment. The normal form for this is
without a separator. This allows versions such as ``1.1.a1`` or ``1.1-a1``
which would be normalized to ``1.1a1``. It should also allow a separator to
be used between the pre-release signifier and the numeral. This allows versions
such as ``1.0a.1`` which would be normalized to ``1.0a1``.


Pre-release spelling
~~~~~~~~~~~~~~~~~~~~

Pre-releases allow the additional spellings of ``alpha``, ``beta``, ``c``,
``pre``, and ``preview`` for ``a``, ``b``, ``rc``, ``rc``, and ``rc``
respectively. This allows versions such as ``1.1alpha1``, ``1.1beta2``, or
``1.1c3`` which normalize to ``1.1a1``, ``1.1b2``, and ``1.1rc3``. In every
case the additional spelling should be considered equivalent to their normal
forms.


Implicit pre-release number
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pre releases allow omitting the numeral in which case it is implicitly assumed
to be ``0``. The normal form for this is to include the ``0`` explicitly. This
allows versions such as ``1.2a`` which is normalized to ``1.2a0``.


Post release separators
~~~~~~~~~~~~~~~~~~~~~~~

Post releases allow a ``.``, ``-``, or ``_`` separator as well as omitting the
separator all together. The normal form of this is with the ``.`` separator.
This allows versions such as ``1.2-post2`` or ``1.2post2`` which normalize to
``1.2.post2``. Like the pre-release separator this also allows an optional
separator between the post release signifier and the numeral. This allows
versions like ``1.2.post-2`` which would normalize to ``1.2.post2``.


Post release spelling
~~~~~~~~~~~~~~~~~~~~~

Post-releases allow the additional spellings of ``rev`` and ``r``. This allows
versions such as ``1.0-r4`` which normalizes to ``1.0.post4``. As with the
pre-releases the additional spellings should be considered equivalent to their
normal forms.


Implicit post release number
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Post releases allow omitting the numeral in which case it is implicitly assumed
to be ``0``. The normal form for this is to include the ``0`` explicitly. This
allows versions such as ``1.2.post`` which is normalized to ``1.2.post0``.


Implicit post releases
~~~~~~~~~~~~~~~~~~~~~~

Post releases allow omitting the ``post`` signifier all together. When using
this form the separator MUST be ``-`` and no other form is allowed. This allows
versions such as ``1.0-1`` to be normalized to ``1.0.post1``. This particular
normalization MUST NOT be used in conjunction with the implicit post release
number rule. In other words, ``1.0-`` is *not* a valid version and it does *not*
normalize to ``1.0.post0``.


Development release separators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Development releases allow a ``.``, ``-``, or a ``_`` separator as well as
omitting the separator all together. The normal form of this is with the ``.``
separator. This allows versions such as ``1.2-dev2`` or ``1.2dev2`` which
normalize to ``1.2.dev2``.


Implicit development release number
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Development releases allow omitting the numeral in which case it is implicitly
assumed to be ``0``. The normal form for this is to include the ``0``
explicitly. This allows versions such as ``1.2.dev`` which is normalized to
``1.2.dev0``.


Local version segments
~~~~~~~~~~~~~~~~~~~~~~

With a local version, in addition to the use of ``.`` as a separator of
segments, the use of ``-`` and ``_`` is also acceptable. The normal form is
using the ``.`` character. This allows versions such as ``1.0+ubuntu-1`` to be
normalized to ``1.0+ubuntu.1``.


Preceding v character
~~~~~~~~~~~~~~~~~~~~~

In order to support the common version notation of ``v1.0`` versions may be
preceded by a single literal ``v`` character. This character MUST be ignored
for all purposes and should be omitted from all normalized forms of the
version. The same version with and without the ``v`` is considered equivalent.


Leading and Trailing Whitespace
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Leading and trailing whitespace must be silently ignored and removed from all
normalized forms of a version. This includes ``" "``, ``\t``, ``\n``, ``\r``,
``\f``, and ``\v``. This allows accidental whitespace to be handled sensibly,
such as a version like ``1.0\n`` which normalizes to ``1.0``.


Examples of compliant version schemes
-------------------------------------

The standard version scheme is designed to encompass a wide range of
identification practices across public and private Python projects. In
practice, a single project attempting to use the full flexibility offered
by the scheme would create a situation where human users had difficulty
figuring out the relative order of versions, even though the rules above
ensure all compliant tools will order them consistently.

The following examples illustrate a small selection of the different
approaches projects may choose to identify their releases, while still
ensuring that the "latest release" and the "latest stable release" can
be easily determined, both by human users and automated tools.

Simple "major.minor" versioning::

    0.1
    0.2
    0.3
    1.0
    1.1
    ...

Simple "major.minor.micro" versioning::

    1.1.0
    1.1.1
    1.1.2
    1.2.0
    ...

"major.minor" versioning with alpha, beta and candidate
pre-releases::

    0.9
    1.0a1
    1.0a2
    1.0b1
    1.0rc1
    1.0
    1.1a1
    ...

"major.minor" versioning with developmental releases, release candidates
and post-releases for minor corrections::

    0.9
    1.0.dev1
    1.0.dev2
    1.0.dev3
    1.0.dev4
    1.0c1
    1.0c2
    1.0
    1.0.post1
    1.1.dev1
    ...

Date based releases, using an incrementing serial within each year, skipping
zero::

    2012.1
    2012.2
    2012.3
    ...
    2012.15
    2013.1
    2013.2
    ...


Summary of permitted suffixes and relative ordering
---------------------------------------------------

.. note::

   This section is intended primarily for authors of tools that
   automatically process distribution metadata, rather than developers
   of Python distributions deciding on a versioning scheme.

The epoch segment of version identifiers MUST be sorted according to the
numeric value of the given epoch. If no epoch segment is present, the
implicit numeric value is ``0``.

The release segment of version identifiers MUST be sorted in
the same order as Python's tuple sorting when the normalized release segment is
parsed as follows::

    tuple(map(int, release_segment.split(".")))

All release segments involved in the comparison MUST be converted to a
consistent length by padding shorter segments with zeros as needed.

Within a numeric release (``1.0``, ``2.7.3``), the following suffixes
are permitted and MUST be ordered as shown::

   .devN, aN, bN, rcN, <no suffix>, .postN

Note that ``c`` is considered to be semantically equivalent to ``rc`` and must
be sorted as if it were ``rc``. Tools MAY reject the case of having the same
``N`` for both a ``c`` and a ``rc`` in the same release segment as ambiguous
and remain in compliance with the specification.

Within an alpha (``1.0a1``), beta (``1.0b1``), or release candidate
(``1.0rc1``, ``1.0c1``), the following suffixes are permitted and MUST be
ordered as shown::

   .devN, <no suffix>, .postN

Within a post-release (``1.0.post1``), the following suffixes are permitted
and MUST be ordered as shown::

    .devN, <no suffix>

Note that ``devN`` and ``postN`` MUST always be preceded by a dot, even
when used immediately following a numeric version (e.g. ``1.0.dev456``,
``1.0.post1``).

Within a pre-release, post-release or development release segment with a
shared prefix, ordering MUST be by the value of the numeric component.

The following example covers many of the possible combinations::

    1.dev0
    1.0.dev456
    1.0a1
    1.0a2.dev456
    1.0a12.dev456
    1.0a12
    1.0b1.dev456
    1.0b2
    1.0b2.post345.dev456
    1.0b2.post345
    1.0rc1.dev456
    1.0rc1
    1.0
    1.0+abc.5
    1.0+abc.7
    1.0+5
    1.0.post456.dev34
    1.0.post456
    1.0.15
    1.1.dev1


Version ordering across different metadata versions
---------------------------------------------------

Metadata v1.0 (:pep:`241`) and metadata v1.1 (:pep:`314`) do not specify a standard
version identification or ordering scheme. However metadata v1.2 (:pep:`345`)
does specify a scheme which is defined in :pep:`386`.

Due to the nature of the simple installer API it is not possible for an
installer to be aware of which metadata version a particular distribution was
using. Additionally installers required the ability to create a reasonably
prioritized list that includes all, or as many as possible, versions of
a project to determine which versions it should install. These requirements
necessitate a standardization across one parsing mechanism to be used for all
versions of a project.

Due to the above, this specification MUST be used for all versions of metadata and
supersedes :pep:`386` even for metadata v1.2. Tools SHOULD ignore any versions
which cannot be parsed by the rules in this specification, but MAY fall back to
implementation defined version parsing and ordering schemes if no versions
complying with this specification are available.

Distribution users may wish to explicitly remove non-compliant versions from
any private package indexes they control.


Compatibility with other version schemes
----------------------------------------

Some projects may choose to use a version scheme which requires
translation in order to comply with the public version scheme defined in
this specification. In such cases, the project specific version can be stored in the
metadata while the translated public version is published in the version field.

This allows automated distribution tools to provide consistently correct
ordering of published releases, while still allowing developers to use
the internal versioning scheme they prefer for their projects.


Semantic versioning
~~~~~~~~~~~~~~~~~~~

`Semantic versioning`_ is a popular version identification scheme that is
more prescriptive than this specification regarding the significance of different
elements of a release number. Even if a project chooses not to abide by
the details of semantic versioning, the scheme is worth understanding as
it covers many of the issues that can arise when depending on other
distributions, and when publishing a distribution that others rely on.

The "Major.Minor.Patch" (described in this specification as "major.minor.micro")
aspects of semantic versioning (clauses 1-8 in the 2.0.0 specification)
are fully compatible with the version scheme defined in this specification, and abiding
by these aspects is encouraged.

Semantic versions containing a hyphen (pre-releases - clause 10) or a
plus sign (builds - clause 11) are *not* compatible with this specification
and are not permitted in the public version field.

One possible mechanism to translate such semantic versioning based source
labels to compatible public versions is to use the ``.devN`` suffix to
specify the appropriate version order.

Specific build information may also be included in local version labels.

.. _Semantic versioning: https://semver.org/


DVCS based version labels
~~~~~~~~~~~~~~~~~~~~~~~~~

Many build tools integrate with distributed version control systems like
Git and Mercurial in order to add an identifying hash to the version
identifier. As hashes cannot be ordered reliably such versions are not
permitted in the public version field.

As with semantic versioning, the public ``.devN`` suffix may be used to
uniquely identify such releases for publication, while the original DVCS based
label can be stored in the project metadata.

Identifying hash information may also be included in local version labels.


Olson database versioning
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``pytz`` project inherits its versioning scheme from the corresponding
Olson timezone database versioning scheme: the year followed by a lowercase
character indicating the version of the database within that year.

This can be translated to a compliant public version identifier as
``<year>.<serial>``, where the serial starts at zero or one (for the
'<year>a' release) and is incremented with each subsequent database
update within the year.

As with other translated version identifiers, the corresponding Olson
database version could be recorded in the project metadata.


Version specifiers
==================

A version specifier consists of a series of version clauses, separated by
commas. For example::

   ~= 0.9, >= 1.0, != 1.3.4.*, < 2.0

The comparison operator determines the kind of version clause:

* ``~=``: `Compatible release`_ clause
* ``==``: `Version matching`_ clause
* ``!=``: `Version exclusion`_ clause
* ``<=``, ``>=``: `Inclusive ordered comparison`_ clause
* ``<``, ``>``: `Exclusive ordered comparison`_ clause
* ``===``: `Arbitrary equality`_ clause.

The comma (",") is equivalent to a logical **and** operator: a candidate
version must match all given version clauses in order to match the
specifier as a whole.

Whitespace between a conditional operator and the following version
identifier is optional, as is the whitespace around the commas.

When multiple candidate versions match a version specifier, the preferred
version SHOULD be the latest version as determined by the consistent
ordering defined by the standard `Version scheme`_. Whether or not
pre-releases are considered as candidate versions SHOULD be handled as
described in `Handling of pre-releases`_.

Except where specifically noted below, local version identifiers MUST NOT be
permitted in version specifiers, and local version labels MUST be ignored
entirely when checking if candidate versions match a given version
specifier.


.. _version-specifiers-compatible-release:

Compatible release
------------------

A compatible release clause consists of the compatible release operator ``~=``
and a version identifier. It matches any candidate version that is expected
to be compatible with the specified version.

The specified version identifier must be in the standard format described in
`Version scheme`_. Local version identifiers are NOT permitted in this
version specifier.

For a given release identifier ``V.N``, the compatible release clause is
approximately equivalent to the pair of comparison clauses::

    >= V.N, == V.*

This operator MUST NOT be used with a single segment version number such as
``~=1``.

For example, the following groups of version clauses are equivalent::

    ~= 2.2
    >= 2.2, == 2.*

    ~= 1.4.5
    >= 1.4.5, == 1.4.*

If a pre-release, post-release or developmental release is named in a
compatible release clause as ``V.N.suffix``, then the suffix is ignored
when determining the required prefix match::

    ~= 2.2.post3
    >= 2.2.post3, == 2.*

    ~= 1.4.5a4
    >= 1.4.5a4, == 1.4.*

The padding rules for release segment comparisons means that the assumed
degree of forward compatibility in a compatible release clause can be
controlled by appending additional zeros to the version specifier::

    ~= 2.2.0
    >= 2.2.0, == 2.2.*

    ~= 1.4.5.0
    >= 1.4.5.0, == 1.4.5.*


Version matching
----------------

A version matching clause includes the version matching operator ``==``
and a version identifier.

The specified version identifier must be in the standard format described in
`Version scheme`_, but a trailing ``.*`` is permitted on public version
identifiers as described below.

By default, the version matching operator is based on a strict equality
comparison: the specified version must be exactly the same as the requested
version. The *only* substitution performed is the zero padding of the
release segment to ensure the release segments are compared with the same
length.

Whether or not strict version matching is appropriate depends on the specific
use case for the version specifier. Automated tools SHOULD at least issue
warnings and MAY reject them entirely when strict version matches are used
inappropriately.

Prefix matching may be requested instead of strict comparison, by appending
a trailing ``.*`` to the version identifier in the version matching clause.
This means that additional trailing segments will be ignored when
determining whether or not a version identifier matches the clause. If the
specified version includes only a release segment, then trailing components
(or the lack thereof) in the release segment are also ignored.

For example, given the version ``1.1.post1``, the following clauses would
match or not as shown::

    == 1.1        # Not equal, so 1.1.post1 does not match clause
    == 1.1.post1  # Equal, so 1.1.post1 matches clause
    == 1.1.*      # Same prefix, so 1.1.post1 matches clause

For purposes of prefix matching, the pre-release segment is considered to
have an implied preceding ``.``, so given the version ``1.1a1``, the
following clauses would match or not as shown::

    == 1.1        # Not equal, so 1.1a1 does not match clause
    == 1.1a1      # Equal, so 1.1a1 matches clause
    == 1.1.*      # Same prefix, so 1.1a1 matches clause if pre-releases are requested

An exact match is also considered a prefix match (this interpretation is
implied by the usual zero padding rules for the release segment of version
identifiers). Given the version ``1.1``, the following clauses would
match or not as shown::

    == 1.1        # Equal, so 1.1 matches clause
    == 1.1.0      # Zero padding expands 1.1 to 1.1.0, so it matches clause
    == 1.1.dev1   # Not equal (dev-release), so 1.1 does not match clause
    == 1.1a1      # Not equal (pre-release), so 1.1 does not match clause
    == 1.1.post1  # Not equal (post-release), so 1.1 does not match clause
    == 1.1.*      # Same prefix, so 1.1 matches clause

It is invalid to have a prefix match containing a development or local release
such as ``1.0.dev1.*`` or ``1.0+foo1.*``. If present, the development release
segment is always the final segment in the public version, and the local version
is ignored for comparison purposes, so using either in a prefix match wouldn't
make any sense.

The use of ``==`` (without at least the wildcard suffix) when defining
dependencies for published distributions is strongly discouraged as it
greatly complicates the deployment of security fixes. The strict version
comparison operator is intended primarily for use when defining
dependencies for repeatable *deployments of applications* while using
a shared distribution index.

If the specified version identifier is a public version identifier (no
local version label), then the local version label of any candidate versions
MUST be ignored when matching versions.

If the specified version identifier is a local version identifier, then the
local version labels of candidate versions MUST be considered when matching
versions, with the public version identifier being matched as described
above, and the local version label being checked for equivalence using a
strict string equality comparison.


Version exclusion
-----------------

A version exclusion clause includes the version exclusion operator ``!=``
and a version identifier.

The allowed version identifiers and comparison semantics are the same as
those of the `Version matching`_ operator, except that the sense of any
match is inverted.

For example, given the version ``1.1.post1``, the following clauses would
match or not as shown::

    != 1.1        # Not equal, so 1.1.post1 matches clause
    != 1.1.post1  # Equal, so 1.1.post1 does not match clause
    != 1.1.*      # Same prefix, so 1.1.post1 does not match clause


Inclusive ordered comparison
----------------------------

An inclusive ordered comparison clause includes a comparison operator and a
version identifier, and will match any version where the comparison is correct
based on the relative position of the candidate version and the specified
version given the consistent ordering defined by the standard
`Version scheme`_.

The inclusive ordered comparison operators are ``<=`` and ``>=``.

As with version matching, the release segment is zero padded as necessary to
ensure the release segments are compared with the same length.

Local version identifiers are NOT permitted in this version specifier.


Exclusive ordered comparison
----------------------------

The exclusive ordered comparisons ``>`` and ``<`` are similar to the inclusive
ordered comparisons in that they rely on the relative position of the candidate
version and the specified version given the consistent ordering defined by the
standard `Version scheme`_. However, they specifically exclude pre-releases,
post-releases, and local versions of the specified version.

The exclusive ordered comparison ``>V`` **MUST NOT** allow a post-release
of the given version unless ``V`` itself is a post release. You may mandate
that releases are later than a particular post release, including additional
post releases, by using ``>V.postN``. For example, ``>1.7`` will allow
``1.7.1`` but not ``1.7.0.post1`` and ``>1.7.post2`` will allow ``1.7.1``
and ``1.7.0.post3`` but not ``1.7.0``.

The exclusive ordered comparison ``>V`` **MUST NOT** match a local version of
the specified version.

The exclusive ordered comparison ``<V`` **MUST NOT** allow a pre-release of
the specified version unless the specified version is itself a pre-release.
Allowing pre-releases that are earlier than, but not equal to a specific
pre-release may be accomplished by using ``<V.rc1`` or similar.

As with version matching, the release segment is zero padded as necessary to
ensure the release segments are compared with the same length.

Local version identifiers are NOT permitted in this version specifier.


Arbitrary equality
------------------

Arbitrary equality comparisons are simple string equality operations which do
not take into account any of the semantic information such as zero padding or
local versions. This operator also does not support prefix matching as the
``==`` operator does.

The primary use case for arbitrary equality is to allow for specifying a
version which cannot otherwise be represented by this specification. This operator is
special and acts as an escape hatch to allow someone using a tool which
implements this specification to still install a legacy version which is otherwise
incompatible with this specification.

An example would be ``===foobar`` which would match a version of ``foobar``.

This operator may also be used to explicitly require an unpatched version
of a project such as ``===1.0`` which would not match for a version
``1.0+downstream1``.

Use of this operator is heavily discouraged and tooling MAY display a warning
when it is used.


Handling of pre-releases
------------------------

Pre-releases of any kind, including developmental releases, are implicitly
excluded from all version specifiers, *unless* they are already present
on the system, explicitly requested by the user, or if the only available
version that satisfies the version specifier is a pre-release.

By default, dependency resolution tools SHOULD:

* accept already installed pre-releases for all version specifiers
* accept remotely available pre-releases for version specifiers where
  there is no final or post release that satisfies the version specifier
* exclude all other pre-releases from consideration

Dependency resolution tools MAY issue a warning if a pre-release is needed
to satisfy a version specifier.

Dependency resolution tools SHOULD also allow users to request the
following alternative behaviours:

* accepting pre-releases for all version specifiers
* excluding pre-releases for all version specifiers (reporting an error or
  warning if a pre-release is already installed locally, or if a
  pre-release is the only way to satisfy a particular specifier)

Dependency resolution tools MAY also allow the above behaviour to be
controlled on a per-distribution basis.

Post-releases and final releases receive no special treatment in version
specifiers - they are always included unless explicitly excluded.


Examples
--------

* ``~=3.1``: version 3.1 or later, but not version 4.0 or later.
* ``~=3.1.2``: version 3.1.2 or later, but not version 3.2.0 or later.
* ``~=3.1a1``: version 3.1a1 or later, but not version 4.0 or later.
* ``== 3.1``: specifically version 3.1 (or 3.1.0), excludes all pre-releases,
  post releases, developmental releases and any 3.1.x maintenance releases.
* ``== 3.1.*``: any version that starts with 3.1. Equivalent to the
  ``~=3.1.0`` compatible release clause.
* ``~=3.1.0, != 3.1.3``: version 3.1.0 or later, but not version 3.1.3 and
  not version 3.2.0 or later.


Direct references
=================

Some automated tools may permit the use of a direct reference as an
alternative to a normal version specifier. A direct reference consists of
the specifier ``@`` and an explicit URL.

Whether or not direct references are appropriate depends on the specific
use case for the version specifier. Automated tools SHOULD at least issue
warnings and MAY reject them entirely when direct references are used
inappropriately.

Public index servers SHOULD NOT allow the use of direct references in
uploaded distributions. Direct references are intended as a tool for
software integrators rather than publishers.

Depending on the use case, some appropriate targets for a direct URL
reference may be an sdist or a wheel binary archive. The exact URLs and
targets supported will be tool dependent.

For example, a local source archive may be referenced directly::

    pip @ file:///localbuilds/pip-1.3.1.zip

Alternatively, a prebuilt archive may also be referenced::

    pip @ file:///localbuilds/pip-1.3.1-py33-none-any.whl

All direct references that do not refer to a local file URL SHOULD specify
a secure transport mechanism (such as ``https``) AND include an expected
hash value in the URL for verification purposes. If a direct reference is
specified without any hash information, with hash information that the
tool doesn't understand, or with a selected hash algorithm that the tool
considers too weak to trust, automated tools SHOULD at least emit a warning
and MAY refuse to rely on the URL. If such a direct reference also uses an
insecure transport, automated tools SHOULD NOT rely on the URL.

It is RECOMMENDED that only hashes which are unconditionally provided by
the latest version of the standard library's :py:mod:`hashlib` module be used
for source archive hashes. At time of writing, that list consists of
``'md5'``, ``'sha1'``, ``'sha224'``, ``'sha256'``, ``'sha384'``, and
``'sha512'``.

For source archive and wheel references, an expected hash value may be
specified by including a ``<hash-algorithm>=<expected-hash>`` entry as
part of the URL fragment.

For version control references, the ``VCS+protocol`` scheme SHOULD be
used to identify both the version control system and the secure transport,
and a version control system with hash based commit identifiers SHOULD be
used. Automated tools MAY omit warnings about missing hashes for version
control systems that do not provide hash based commit identifiers.

To handle version control systems that do not support including commit or
tag references directly in the URL, that information may be appended to the
end of the URL using the ``@<commit-hash>`` or the ``@<tag>#<commit-hash>``
notation.

.. note::

   This isn't *quite* the same as the existing VCS reference notation
   supported by pip. Firstly, the distribution name is moved in front rather
   than embedded as part of the URL. Secondly, the commit hash is included
   even when retrieving based on a tag, in order to meet the requirement
   above that *every* link should include a hash to make things harder to
   forge (creating a malicious repo with a particular tag is easy, creating
   one with a specific *hash*, less so).

Remote URL examples::

    pip @ https://github.com/pypa/pip/archive/1.3.1.zip#sha1=da9234ee9982d4bbb3c72346a6de940a148ea686
    pip @ git+https://github.com/pypa/pip.git@7921be1537eac1e97bc40179a57f0349c2aee67d
    pip @ git+https://github.com/pypa/pip.git@1.3.1#7921be1537eac1e97bc40179a57f0349c2aee67d


File URLs
---------

File URLs take the form of ``file://<host>/<path>``. If the ``<host>`` is
omitted it is assumed to be ``localhost`` and even if the ``<host>`` is omitted
the third slash MUST still exist. The ``<path>`` defines what the file path on
the filesystem that is to be accessed.

On the various \*nix operating systems the only allowed values for ``<host>``
is for it to be omitted, ``localhost``, or another FQDN that the current
machine believes matches its own host. In other words, on \*nix the ``file://``
scheme can only be used to access paths on the local machine.

On Windows the file format should include the drive letter if applicable as
part of the ``<path>`` (e.g. ``file:///c:/path/to/a/file``). Unlike \*nix on
Windows the ``<host>`` parameter may be used to specify a file residing on a
network share. In other words, in order to translate ``\\machine\volume\file``
to a ``file://`` url, it would end up as ``file://machine/volume/file``. For
more information on ``file://`` URLs on Windows see
`MSDN <https://web.archive.org/web/20130321051043/http://blogs.msdn.com/b/ie/archive/2006/12/06/file-uris-in-windows.aspx>`_.



Summary of differences from pkg_resources.parse_version
=======================================================

* Note: this comparison is to ``pkg_resources.parse_version`` as it existed at
  the time :pep:`440` was written. After the PEP was accepted, setuptools 6.0 and
  later versions adopted the behaviour described here.

* Local versions sort differently, this specification requires that they sort as greater
  than the same version without a local version, whereas
  ``pkg_resources.parse_version`` considers it a pre-release marker.

* This specification purposely restricts the syntax which constitutes a valid version
  while ``pkg_resources.parse_version`` attempts to provide some meaning from
  *any* arbitrary string.

* ``pkg_resources.parse_version`` allows arbitrarily deeply nested version
  signifiers like ``1.0.dev1.post1.dev5``. This specification however allows only a
  single use of each type and they must exist in a certain order.



.. _version-specifiers-regex:

Appendix: Parsing version strings with regular expressions
==========================================================

As noted earlier in the :ref:`public-version-identifiers` section,
published version identifiers SHOULD use the canonical format. This
section provides regular expressions that can be used to test whether a
version is already in that form, and if it's not, extract the various
components for subsequent normalization.

To test whether a version identifier is in the canonical format, you can use
the following function:

.. code-block:: python

    import re
    def is_canonical(version):
        return re.match(r'^([1-9][0-9]*!)?(0|[1-9][0-9]*)(\.(0|[1-9][0-9]*))*((a|b|rc)(0|[1-9][0-9]*))?(\.post(0|[1-9][0-9]*))?(\.dev(0|[1-9][0-9]*))?$', version) is not None

To extract the components of a version identifier, use the following regular
expression (as defined by the `packaging <https://github.com/pypa/packaging>`_
project):

.. code-block:: python

    VERSION_PATTERN = r"""
        v?
        (?:
            (?:(?P<epoch>[0-9]+)!)?                           # epoch
            (?P<release>[0-9]+(?:\.[0-9]+)*)                  # release segment
            (?P<pre>                                          # pre-release
                [-_\.]?
                (?P<pre_l>(a|b|c|rc|alpha|beta|pre|preview))
                [-_\.]?
                (?P<pre_n>[0-9]+)?
            )?
            (?P<post>                                         # post release
                (?:-(?P<post_n1>[0-9]+))
                |
                (?:
                    [-_\.]?
                    (?P<post_l>post|rev|r)
                    [-_\.]?
                    (?P<post_n2>[0-9]+)?
                )
            )?
            (?P<dev>                                          # dev release
                [-_\.]?
                (?P<dev_l>dev)
                [-_\.]?
                (?P<dev_n>[0-9]+)?
            )?
        )
        (?:\+(?P<local>[a-z0-9]+(?:[-_\.][a-z0-9]+)*))?       # local version
    """

    _regex = re.compile(
        r"^\s*" + VERSION_PATTERN + r"\s*$",
        re.VERBOSE | re.IGNORECASE,
    )



History
=======

- August 2014: This specification was approved through :pep:`440`.
- May 2025: Clarify that development releases are a form of pre-release when
  they are handled.
