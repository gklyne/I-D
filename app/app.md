---
title: The app URI scheme
abbrev: app
docname: draft-soilandreyes-app-00
date: 2018-01-31
category: info

ipr: trust200902
area: General
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Soiland-Reyes
    name: Stian Soiland-Reyes
    organization: The University of Manchester
    street: Oxford Road
    city: Manchester
    country: United Kingdom
    email: stain@apache.org
 
normative:
  # Media types
  RFC2046:
  # RFC keywords
  RFC2119:
  # UTF-8
  RFC2279:
  # text/uri-list
  RFC2483:
  # URI
  RFC3986:
  # IRI
  RFC3987:
  # Randomness requirements
  RFC4086:
  # UUID
  RFC4122:
  # .well-known
  RFC5785:
  # ni hashes
  RFC6920:
  # URI design
  RFC7320:
  # Registration Procedures for URI Schemes
  RFC7595:
  # file URI scheme
  RFC8089:

informative:
  # base64
  RFC4648:
  # bagit
  I-D.draft-kunze-bagit-14:  
  # w3c app://
  W3C.NOTE-app-uri-20150723:
  # w3c widget://
  W3C.NOTE-widgets-uri-20120313:
  # RO bundle
  ROBundle:
    seriesinfo:
      Zenodo: report
      DOI: 10.5281/zenodo.12586
    title: Research Object Bundle 1.0
    target: https://w3id.org/bundle/
    author:
    - name: Stian Soiland-Reyes
      ins: S. Soiland-Reyes
    - name: Matthew Gamble
      ins: M. Gamble
    - name: Robert Haines
      ins: R. Haines
    date: '2014-11-05'
  CWLViewer:
    seriesinfo:
      Zenodo: Software
      DOI: 10.5281/zenodo.823534
    title: 'Common-Workflow-Language/CWLviewer: CWL Viewer'
    target: https://view.commonwl.org/
    author:
    - name: Mark Robinson
      ins: M. Robinson
    - name: Stian Soiland-Reyes
      ins: S. Soiland-Reyes
    - name: Michael R. Crusoe
      ins: M. Crusoe
    date: '2017-08-24'




--- abstract

This Internet-Draft proposes the `app:` URI scheme, which can be used 
to consume or reference hypermedia resources bundled inside 
a file archive or a mobile application package. 

This URI scheme provides mechanisms to generate a 
unique base URI to represent the root of the archive, 
so that relative URI references in a bundled resource 
can be resolved within the archive without having to 
extract the archive content on the local file system.

An app URI can be used for purposes of isolation
(e.g. when consuming multiple archives), 
security constraints (avoiding "climb out" from the archive),
or for externally identiyfing sub-resources in 
other hypermedia formats.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
{{RFC2119}}.

--- middle

Background        {#background}
==========

Applications that are accessing resources bundled inside a 
file archive (e.g. zip or tar.gz) can struggle to consume
hypermedia content types that use relative
URI references {{RFC3986}}, as it is challenging to 
determine the base URI in a consistent fashion. 

Frequently the archive must be unpacked locally to 
synthesize base URIs like `file:///tmp/a1b27ae03865/`
to represent the root of the archive. Such URIs are fluctual,
might not be globally unique, and could be vulnerable to
attacks such as "climbing out" of the root directory.

Mobile applications that are distributed as a package 
may bundle resources such as stylesheets with 
relative URI references to images and fonts. 

An archive containing multiple HTML or 
Linked Data resources, such as in a 
BagIt archive {{I-D.draft-kunze-bagit-14}}, may be using
relative URIs to cross-reference constituent files.

Consumptions of archives might be performed
in memory or through a common framework, abstracting
away any local file location. 

Consumption of an archive with a consistent base URL 
should be possible no matter from which location it was retrieved,
or on which device it is inspected.

The `file:` URI scheme {{RFC8089}} 
can be ill-suited for purposes such as above, where a
location-independent URI scheme is more flexible, 
secure and globally unique.


Scheme syntax    {#syntax}
============= 

The `app` URI scheme follows the {{RFC3986}} syntax for hierarchical
URIs according to the following production:

    appURI    =  "app://" app-authority path-absolute
                   [ "?" query ] [ "#" fragment ]

The `app-authority` component provides a unique identifier for the opened archive. 
See [Authority](#authority) for details.

The `path-absolute` component provides the absolute path of a resource
(e.g. a file or directory) within the archive. See [Path](#path)
for details.

The semantics of the `query` component is undefined by this Internet-Draft. 
Implementations SHOULD NOT generate a query component for app URIs.

The "fragment" component MAY be used by implementations according
to {{RFC3986}} and the implied media type {{RFC2046}} of the
resource at the path. This Internet-Draft does not specify how
to determine the media type.


Authority  {#authority}
---------

The purpose of the `authority` component in an app URI is 
to build a unique base URI for a particular archive. The 
authority is NOT intended to be resolvable without former
knowledge of the archive.


The authority of an app URI MUST be valid according to
this production:

    app-authority    = UUID | alg-val | authority

The `UUID` production match its definition in {{RFC4122}}, e.g.
`2a47c495-ac70-4ed1-850b-8800a57618cf`

The `alg-val` production match its definition in {{RFC6920}}, e.g.
``sha-256;JCS7yveugE3UaZiHCs1XpRVfSHaewxAKka0o5q2osg8`

The `authority` production match its definition in {{RFC3986}}, e.g. `example.com`. 
As this production necessarily also match the `UUID` and `alg-val`
productions, consumers of app URIs should attempt to match those first.
While {{RFC7320}} section 2.2 says an extension may not 
_"define the structure or the semantics for URI authorities", 
any extensions of this Internet-Draft **are** permitted to do so, 
if using a DNS domain name under their control. 
For instance, a vendor owning `example.com` may specify that 
`{OID}` in `{OID}.oid.example.com` has special semantics.

The choice of authority depends on the purpose of the app URI within the implementation.
Below are some recommendations:

1. For security/sandboxing when independently interpreting resources in 
  an archive, the authority SHOULD be a 
  UUID v4 {{RFC4122}} created with a suitable random number generator {{RFC4086}}.
  This ensures with high probablity that 
  the app base URI is globally unique. An application MAY choose to 
  reuse a previously assigned UUID that is associated with the archive.
2. For referencing resources in an archive accessed at a particular URL, the
  authority SHOULD be generated as a name-based UUID v5 {{RFC4122}}; that is 
  based on the SHA1 concatination of the URL namespace 
  `6ba7b811-9dad-11d1-80b4-00c04fd430c8` (as UUID bytes) and the 
  ASCII bytes of the particular URL. It is NOT RECOMMENDED to use this approach 
  with a file URI {{RFC8089}} without a fully qualified `host` name.
3. For referencing resources in an archive as a 
  particular bytestream, independent of its location, the authority SHOULD be 
  a checksum of the archive bytes. The checksum MUST be expressed 
  according to {{RFC6920}}'s `alg-val` production, and SHOULD use the
  `sha-256` algorithm. It is NOT RECOMMENDED to use truncated hash methods.

The generic `authority` production MAY be used 
for extensions if the above mechanisms are not suitable. 
Care should be taken so that the custom `authority` 
do not match the `UUID` nor `alg-val` productions.  


Path  {#path}
----

The `path-absolute` component MUST match the production in 
{{RFC3986}} and provide the absolute path of a resource
(e.g. a file or directory) within the archive. 

Archive media types vary in constraints and flexibilities 
of how to express paths. Here we assume an archive generally
consists of a single root directory, which can contain 
multiple directories and files at arbitrary nesting levels.

Paths SHOULD be expressed using `/` as the directory separator.
The below productions are from {{RFC3986}}:

    path-absolute = "/" [ segment-nz *( "/" segment ) ]
    segment       = *pchar
    segment-nz    = 1*pchar

In an app URI, each intermediate `segment` (or `segment-nz`) 
represent a directory name, while the last segment represent 
either a directory or file name.

It is RECOMMENDED to include the trailing `/` if it is known 
the path represents a directory.


Scheme semantics    {#semantics}
================

This Internet-Draft does not constrain what particular format
might constitute an _archive_, and neither does it require 
that the archive is retrievable as a single bytestream or file.
Examples of archive media types include 
`application/zip`, `application/vnd.android.package-archive`, 
`application/x-tar`, `application/x-gtar` and 
`application/x-7z-compressed`.

The _path_ of an app URI identify individual resources within a
particular archive, typically a *directory* or *file*.

The app URIs can be used for uniquely identifying 
the resources independent of the location of the archive,
such as within an information system.

Assuming an appropriate resolution mechanism which have
knowledge of the corresponding archive, an app URI
can also be used for resolution.

Resolution   {#resolution}
----------

This Internet-Draft do not specify the protocol to 
resolve resources according to the app URI scheme. 
For instance, one implementation might rewrite app URIs to 
localized `file:///` paths in a temporary directory, while
another implementation might use an embedded HTTP server.


It is envisioned that an implementation will 
have extracted or opened an archive in 
advance, and assigned it an appropriate authority according
to [Authority](#authority). Such an implementation
can then resolve app URIs programmatically, e.g. by using
in-memory access or mapping paths to the extracted archive on 
the local file system.

Resolution of an app URI within an implementation SHOULD:

1. Fail with the equivalent of _Not Found_ if the authority is unknown.
2. Fail with the equivalent of _Gone_ if the authority is known, but the content of the archive is no longer available.
3. Fail with the equivalent of _Not Found_ if the path does not map to a file or directory within the archive.
4. Return the corresponding (potentially uncompressed) bytestream if the path maps to a file within the archive.
5. Return an appropriate directory listing if the path maps to a directory within the archive.
6. Return an appropriate directory listing of the archive's root directory if the path is `/`

Not all archive formats or implementations will have the 
concept of a directory listing, in which case the directory listing 
SHOULD fail with the equivalent of "Not Implemented".

It is not specified in this Internet-Draft how an implementation
can determine the media type of a file within an archive. This may
be expressed in secondary resources (such as a manifest), 
be determined by file extensions or magic bytes.

The media type `text/uri-list` {{RFC2483}} MAY be used to represent 
a directory listing, in which case it SHOULD contain only URIs
that start with the app URI of the directory.

Some archive formats might support resources which are 
neither directories nor regular files (e.g. device files, 
symbolic links). This Internet-Draft does not specify the 
semantics of attempting to resolve such resources.


Resolving from a .well-known endpoint  {#well-known}
-------------------------------------

If the `authority` component of an app URI matches the `alg-val`
production, an application MAY attempt to resolve the authority
from any `.well-known/ni/` endpoint {{RFC5785}} as specified in 
{{RFC6920}} section 4, in order to retrieve the complete
archive. Applications SHOULD verify the checksum of the 
retrieved archive before resolving the individual path.


Encoding considerations   {#encoding}
=======================

The production for `UUID` and `alg-val` are restricted to
ASCII and should not require any encoding considerations.

Care should be taken to %-encode the directory and file segments 
of `path-absolute` according to {{RFC3986}} (for URIs) or 
{{RFC3987}} (for IRIs).

When used as part an IRI, paths SHOULD be expressed using
international Unicode characters instead of %-encoding as ASCII.

Not all archive media types have an explicit 
character encoding specified for their paths. 
If no such information is available, 
implementations SHOULD assume 
that the path is encoded with UTF-8 {{RFC2279}}.

Some archive media types are case-insensitive, in 
which cases it is RECOMMENDED to preserve the casing 
as expressed in the archive.


Interoperability considerations   {#interoperability}
===============================

As multiple authorities are possible ([Authority](#authority)), 
there could be interoperability challenges when exchanging app URIs
between implementations. For instance:

1. Two implementations describe the same archive 
  (e.g. stored in the same local file path), but using 
  different v4 UUIDs. The implementations may 
  need to detect equality of the two UUIDs out of band.
2. Two implementations describe an archive retrieved
  from the same URL, with the same v5 UUIDs, but retrieved
  at different times. The implementations might disagree
  about the content of the archive.
3.  Two implementations describe an archive retrieved
  from the same URL, with the same v5 UUIDs, but retrieved 
  using different content negotiation resulting in different
  archive representations. The implementations may disagree 
  about path encoding, file name casing or hierarchy.
4. Two implementations describe the same archive bytestream 
  using the `alg-val` production, but they have used
  two different hash algorithms. The implementations may
  need to negotiate to a common hash algorithm.
5. An implementation describe an archive using
  the `alg-val` production, but a second
  implementation concurrently modifies the archive's content.
  The first implementation may need to detect changes to 
  the archive or verify the checksum at the end of its operations.
6. Two implementations might have different views of the 
  content of the same archive if the format permits 
  multiple entries with the same path. Care should
  be taken to follow the convention and specification 
  of the particular archive format.


Security Considerations {#security}
=======================

As when handling any content, extra care should be taken when
consuming archives and app URIs from unknown sources.

An archive could contain compressed files that expand to
fill all available disk space.

A maliciously crafted archive could contain paths with characters
(e.g. backspace) which could make an app URI invalid or
misleading if used unescaped.

A maliciously crafted archive could contain paths
(e.g. combined Unicode sequences) that cause the 
app URI to be very long, causing issues in information
systems propagating said URI. 

An archive might contain symbolic links that, if 
extracted to a local file system, might address files 
outside the archive's directory structure.

An maliciously crafted app URI might contain `../` segments, 
which if naively converted to a `file:///` URI might address
files outside the archive's directory structure. 

In particular for IRIs, an archive might contain multiple 
paths with similar-looking characters or with different
Unicode combine sequences, which could be facilitated
to mislead users.


IANA Considerations  {#iana}
===================

This Internet-Draft contains the Provisional IANA 
registration of the app URI scheme according to {{RFC7595}}.

Scheme name: uri

Status: provisional

Applications/protocols that use this protocol:
  Hypermedia-consuming application that handle archives.


Contact: Stian Soiland-Reyes <stain@apache.org>

Change controller: Stian Soiland-Reyes


--- back


Examples  {#examples}
========



History   {#history}
=======

This Internet-Draft proposes the URI scheme `app`, which was originally 
proposed by {{W3C.NOTE-app-uri-20150723}} but never registered with IANA.
This Note was evolved from {{W3C.NOTE-widgets-uri-20120313}} which 
proposed the URI scheme `widget`.

The W3C Notes did not progress further as Recommendation track documents.

While the focus of W3C Notes was to specify how to resolve resources from
within a packaged application, this Internet-Draft generalize 
the URI scheme to support addressing and identifying resources within
any archive, and de-emphasize the resolution mechanism.

For compatibility with existing adaptations of the `app` URI scheme, 
e.g. {{ROBundle}} and {{CWLViewer}}, this Internet-Draft reuse the same
scheme name and remains compatible with the intentions of
{{W3C.NOTE-app-uri-20150723}}.
