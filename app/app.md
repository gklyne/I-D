---
title: The app: URI scheme
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
    organization: Apache Software Foundation
    email: stain@apache.org
 
normative:
  RFC2119:
  RFC4086:
  RFC4648:

  # UUID
  RFC4122
  # URI
  RFC3986:
  # .well-known
  RFC5785:
  # URI design
  RFC7320:
  # URI scheme requests
  BCP35:
  # file URI scheme
  RFC8089:
  # IRI
  RFC3987:

informative:
  # bagit
  draft-kunze-bagit-14:  
  w3c-app-url:
    target: https://www.w3.org/TR/2015/NOTE-app-uri-20150723/
    title: The app: URL Scheme (W3C Working Group Note)
    date: 2015-07-23
    author: 
      name: Marcos Caceres



--- abstract

This draft proposes the `app:` URI scheme, which can be used 
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

--- middle

Background        {#background}
==========

Applications that are accessing resources bundled inside a 
file archive (e.g. zip or tar.gz) can struggle to consume
hypermedia content types that use relative
URI references {{RFC3986}}, as it is difficult to 
determine the base URI in a consistent fashion. 

Frequently the archive must be unpacked locally to 
synthesize base URIs like `file:///tmp/a1b27ae03865/`
to represent the root of the archive. Such URIs are fluctual,
might not be globally unique, and could be vulnerable to
attacks such as "climbing out" of the root directory.

Mobile applications that are distributed as a package 
may bundle resources such as stylesheets with 
relative URI references to bundled images and fonts. 

An archive containing multiple HTML or 
Linked Data resources, such as in a 
BagIt archive {{draft-kunze-bagit-14}}, may use
relative URIs to cross-reference constituent files.

Consistent consumption of an archive should be possible 
no matter from which location it was retrieved,
or on which device it is inspected.

Consumptions of archives might be performed
in memory or through a common framework, abstracting
away any local file location. 

Therefore the `file:` URI scheme {{RFC8089}} 
can be ill-suited for purposes of consuming or
referencing resources within an archive, where a
location-independent URI scheme is more flexible, 
secure and globally unique.


Scheme syntax    {#syntax}
============= 

The `app` URI scheme follows the {{RFC3986}} syntax for hierarchical URIs:

    appURI    =  "app://" app-authority path-absolute [ "?" query ] [ "#" fragment ]

The `app-authority` component provides a unique identifier for the opened archive. 
See subsection [Authority](#authority) for details.

The `path-absolute` component provides the absolute path of a resource
(e.g. a file or directory) within the archive. See subsection [Path](#path)
for details.

The meaning of the `query` component is undefined by this Internet-Draft. 
Implementations SHOULD NOT generate a query component for app URIs, 
but MAY facilitate `query` for application or content-type specific behaviour.

The fragment MAY be used by implementations according to {{RFC3986}}
and the implied media type {{RFC2046}} of the path. 
This Internet-Draft does not specify how to determine the media type.


Authority  {#authority}
---------

The purpose of the authority in an app URI is to provide a unique base URI for  
resources within a particular archive. The authority is NOT intended to 
be resolvable without former knowledge of the archive.


The authority of an app URI MUST be valid according to this production:

    app-authority    = UUID | alg-val | authority

The `UUID` production SHOULD match its definition in {{RFC4122}}.

The `alg-val` production SHOULD match its definition in {{RFC6920}}.

The `authority` MUST match its definition in {{RFC3986}}.

The choice of authority depends on the purpose of the app URI within the implementation.

1. For security/sandboxing when interpreting resources in 
  an archive, the authority SHOULD be a 
  {{RFC4122}} UUID v4 created with a suitable random number generator.
  This ensures with high probablity that 
  the app base URI is globally unique. An application MAY choose to 
  reuse a previously assigned UUID that is associated with the archive.
2. For referencing resources in an archive accessed at a particular URL, the
  authority SHOULD be generated as a name-based UUID v5 {{RFC4122}} 
  based on the SHA1 concatination of the URL namespace 
  `6ba7b811-9dad-11d1-80b4-00c04fd430c8` (as UUID bytes) and the 
  ASCII bytes of the URL. It is NOT RECOMMENDED to use this approach 
  with a file URI {{RFC8089}} without a fully qualified `host` name.
3. For purposes of referencing resources in an archive as a 
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

Paths SHOULD be expressed using `/` as the directory separator, 
similar to the production `local-path` in the file URI scheme
{{RFC8089}}.


Scheme semantics    {#semantics}
================

app URIs identify the individual resources within an
archive, e.g. directories and files.




Resolution   {#resolution}
----------

This Internet-Draft do not specify the protocol to 
resolve resources according to the app URI scheme. 

It is envisioned that an implementation will 
have mounted or opened an archive in 
advance, and assigned it an appropriate authority according
to section [Authority](#authority). Such an implementation
can then resolve app URIs programmatically, e.g. by using
in-memory access or mapping paths to the extracted archive on 
the local file system.

Resolution of an app URI within an implementation SHOULD:

1. Fail with the equivalent of "Not Found" if the authority is unknown.
2. Fail with the equivalent of "Gone" if the authority is known, but the content of the archive is no longer available.
3. Fail with the equivalent of "Not Found" if the path does not map to a file or directory within the archive
4. Return the corresponding (potentially uncompressed) bytestream if the path maps to a file within the archive
5. Return an appropriate directory listing if the path maps to a directory within the archive
6. Return an appropriate directory listing of the archive's root directory if the path is /

Not all archive formats have the concept of a directory or a directory listing, in which case
the directory listing should fail with the equivalent of "Not Implemented".

It is not specified in this Internet-Draft how an implementation
can determine the media type of a file within an archive. This may
be expressed in secondary resources (such as a manifest), 
be determined by file extensions or magic bytes.

The media type `text/uri-list` MAY be used to represent 
a directory listing, in which case it SHOULD contain only URIs
that start with the app URI of the directory.


Encoding considerations   {#encoding}
=======================

The production for `UUID` and `alg-val` are restricted to
ASCII and should not require any encoding considerations.

Care should be taken to %-encode the directory and file names 
of `path-absolute` according to {{RFC3986}} (for URIs) or 
{{RFC3987}} (for IRIs).

When used as part an IRI, paths SHOULD be expressed using
international Unicode characters instead of %-encoding as ASCII.

Not all archive media types have an explicit 
character encoding specified for their paths. 
If no such information is available, 
implementations SHOULD assume 
that the path is encoded with UTF-8.

Some archive media types are case-insensitive, in 
which cases it is RECOMMENDED to preserve the casing 
as expressed in the archive.


Interoperability considerations   {#interoperability}
===============================

As multiple authorities are possible (see section [Authority](authority)), 
there could be interoperability challenges when exchanging app URIs
between implementations. For instance:

1. Two implementations describe the same archive 
  (for instance stored in the same local file path), but using 
  different v4 UUIDs. The implementations may 
  need to detect equality of the two UUIDs out of band.
2. Two implementations describe an archive retrieved
  from the same URL, with the same v5 UUIDs, but retrieved
  at different times. The implementations might disagree
  about the content of the archive.
3.  Two implementations describe an archive retrieved
  from the same URL, with the same v5 UUIDs,   
  but retrieved using different content negotiation. 
  The implementations may disagree about path encoding, 
  casing or hierarchy.
4. Two implementations describe the same archive bytes using
  {{RFC6920}} hashing, but they have used
  two different hash algorithms. The implementations may 
  need to negotiate to a common hash algorithm.
5. An implementation describe an archive using
  {{RFC6920}} hashing, but a second
  implementation concurrently modifies the archive's content. 
  The first implementation may need to detect changes to 
  the archive or verify the checksum.


Security Considerations {#security}
=======================





IANA Considerations  {#iana}
===================

This Internet-Draft contains the provisional IANA 
registration of app: URI scheme according to {{BCP35}.

Scheme name: uri

Status: provisional

Applications/protocols that use this protocol:
  Any hypermedia consuming application that can handle archives.


Contact: Stian Soiland-Reyes <stain@apache.org>

Change controller: Stian Soiland-Reyes


--- back


Examples  {#examples}
========

This appendix provides some examples of the STuPiD protocol operation.

~~~~~~~~~~
   Request:

      GET /stupid.php HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:30:37 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 17
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      192.0.2.239:36654
~~~~~~~~~~
{: #figxmpdisco title="Discovering External IP Address and Port"}

~~~~~~~~~~
   Request:

      POST /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive
      Content-Type: application/octet-stream
      Content-Length: 11

      Hello World

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:20:34 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 0
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream
~~~~~~~~~~
{: #figxmpstore title="Storing Data"}

~~~~~~~~~~
   Request:

      GET /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:21:29 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 11
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      Hello World
~~~~~~~~~~
{: #figxmpretr title="Retrieving Data"}


Sample Implementation     {#impl}
=====================

~~~~~~~~~~
<?php
header("Cache-Control: no-cache, must-revalidate");
header("Expires: Sat, 26 Jul 1997 05:00:00 GMT");
header("Content-Type: application/octet-stream");

mysql_connect(localhost, "username", "password");
mysql_select_db("stupid");

$chid = mysql_real_escape_string($_GET["chid"]);

if ($_SERVER["REQUEST_METHOD"] == "GET") {
   if (empty($chid)) {
      echo $_SERVER["REMOTE_ADDR"] . ":" . $_SERVER["REMOTE_PORT"];
   } elseif ($result = mysql_query("SELECT `data` FROM `Data` " .
                         "WHERE `chid` = '$chid'")) {
      if ($row = mysql_fetch_array($result, MYSQL_ASSOC)) {
         echo base64_decode($row["data"]);
      } else {
         header("HTTP/1.0 404 Not Found");
      }
      mysql_free_result($result);
   } else {
      header("HTTP/1.0 404 Not Found");
   }
} elseif ($_SERVER["REQUEST_METHOD"] == "POST") {
   if (empty($chid)) {
      header("HTTP/1.0 404 Not Found");
   } else {
      mysql_query("DELETE FROM `Data` " .
                  "WHERE `timestamp` < DATE_SUB(NOW(), INTERVAL 5 MINUTE)");
      $data = base64_encode(file_get_contents("php://input"));
      if (!mysql_query("INSERT INTO `Data` (`chid`, `data`) " .
                       "VALUES ('$chid', '$data')")) {
         header("HTTP/1.0 403 Bad Request");
      }
   }
} else {
   header("HTTP/1.0 405 Method Not Allowed");
   header("Allow: GET, HEAD, POST");
}
mysql_close();
?>
~~~~~~~~~~
{: #figimpl title="STuPiD Sample Implementation"}


Using XMPP as Out-Of-Band Channel  {#xmpp}
=================================

XMPP {{I-D.ietf-xmpp-3920bis}} is a good choice for
an out-of-band channel.

The notification protocol is closely modeled after XMPP's
In-Band Bytestreams (IBB, see
http://xmpp.org/extensions/xep-0047.html). Just replace the
namespace and insert the STuPiD Retrieval URI instead of the
actual Base64 encoded data, see {{figxmpnots}}.
(Note that the current proposal redundantly sends a sid and a
seq as well as the chid composed of these two; it may be
possible to optimize this, possibly sending the constant prefix
of the URI once at bytestream creation time.)

Notifications MUST be processed in the order they are
received. If an out-of-sequence notification is received for a
particular session (determined by checking the 'seq' attribute),
then this indicates that a notification has been lost. The
recipient MUST NOT process such an out-of-sequence notification,
nor any that follow it within the same session; instead, the
recipient MUST consider the session invalid.  (Adapted from
http://xmpp.org/extensions/xep-0047.html#send)

Of course, other methods can be used for setup and teardown, such as Jingle
(see http://xmpp.org/extensions/xep-0261.html).

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='jn3h8g65'
          to='juliet@capulet.com/balcony'
          type='set'>
        <open xmlns='urn:xmpp:tmp:stupid'
              block-size='65536'
              sid='i781hf64'
              stanza='iq'/>
      </iq>
~~~~~~~~~~
{: #figxmpcri title="Creating a Bytestream: Initiator requests session"}


~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='jn3h8g65'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpcrr title="Creating a Bytestream: Responder accepts session"}



~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='kr91n475'
          to='juliet@capulet.com/balcony'
          type='set'>
        <data xmlns='urn:xmpp:tmp:stupid'
              seq='0'
              sid='i781hf64'
              url='http://example.org/stupid.php?chid=i781hf64-0'/>
      </iq>
~~~~~~~~~~
{: #figxmpnots title="Sending Notifications: Notification in an IQ stanza"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='kr91n475'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpnota title="Sending Notifications: Acknowledging notification using IQ"}

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='us71g45j'
          to='juliet@capulet.com/balcony'
          type='set'>
        <close xmlns='urn:xmpp:tmp:stupid'
               sid='i781hf64'/>
      </iq>
~~~~~~~~~~
{: #figxmpclor title="Closing the Bytestream: Request"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='us71g45j'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpclos title="Closing the Bytestream: Success response"}
