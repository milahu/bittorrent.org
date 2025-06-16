:BEP: XXX
:Title: Magnet-URI Webseeding
:Version: $Revision$
:Last-Modified: $Date$
:Author:  T X
:Status:  Draft
:Type:    Standards Track
:Content-Type: text/x-rst
:Created: 02-Mar-2012

--- Work in Progress - Work in Progress - Work in Progress ---

This is an attempt to formalize the "ws" parameter for one
thing. For another, it extents the current, common usage
with the possibility to fetch metadata via a webseed and
therefore adds additional interpretations of the "ws",
"as" and "xs" parameters.

The only "documentation" so far seems to be:
`[1] <https://trac.transmissionbt.com/ticket/2631#comment:2>`__,
`[2] <http://forum.bittorrent.org/viewtopic.php?pid=641#p641>`__
and source code.

Feel free to contribute or declare everything as non-sense
on the
`Discussion <https://wiki.theory.org/Talk_BitTorrent_Magnet-URI_Webseeding>`__
page :).

Abstract
========

This BEP adds support for alternative sources as specified
in BEP19 for both data and metadata in the case of magnet
URIs as specified in BEP9. It updates BEP9 with the magnet
link parameters "as", "xs", "ws" and "cas" and their
interpretation in the presence of an "xt=urn:btih:"
parameter.

Introduction
============

A magnet URI as specified in
<ref>\ `[3] <http://magnet-uri.sourceforge.net/magnet-draft-overview.txt>`__\ </ref>
and BEP9 only provides the ability to retrieve the "info"
section of a torrent file via the "xt=urn:btih:"
parameter, as well as a <!-- display name ("dn") and --> tracker url
("tr") parameter. However, the metadata extension
"url-list" (BEP19) is not part of the "info" section.
Therefore, to be able to use BEP19 in the context of magnet
links the Bittorrent protocol and its interpretation of
magnet URIs needs to be updated to allow fetching data from
webseeds again.

Webseeds are especially useful if the availability of
Bittorrent seeds is limited but alternative sources exist.
However in the case of magnet URIs if not only no Bittorrent
seed but also no Bittorrent peer in general exists, BEP19 is
not applicable although the data itself is available. A
Bittorrent client is not able to verify the retrieved data
due to the "info" part not being available. Therefore this
BEP further updates BEP9 and BEP19 to allow fetching
metadata from webseeds.

Furthermore the purpose of this BEP is to increase the
usability of magnet URIs: It allows the provider of magnet
URIs to embedd a .torrent file web source within a magnet
URI which allows the provider to only have to present a
single link to any users in all circumstances (even if a
client does not support BEP9 or if BEP9 is expected to be
slow due to e.g. a low amount of peers). This removes the
possible confusion two provided links, i.e. a magnet link
and a classic .torrent file link, may cause for a new
BitTorrent user.

The "cas" parameter makes it possible to merge "ws" and "xs"
parameters, to produce shorter magnet URIs.

<!--
TODO explain parameter names
- ws = webseed
- xs = exact source
- as = acceptable source
- cas = content-addressed storage https://en.wikipedia.org/wiki/Content-addressable_storage
-->

Requirements
============

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL",
"SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as
described in BCP 14, RFC 2119 [KEYWORDS].

<!--
TODO reorder the parameter chapters?
first metadata, then payload data.
first "ws", then others.
-->

Retrieving payload data - "ws" parameter
----------------------------------------

As long as no metadata is available, no further steps for
retrieving payload data MUST be taken. Otherwise if the
"ws" parameter is available then it SHOULD be interpreted
and used in the following way:

If the client supports BEP19 then it SHOULD append the "ws"
parameter as a "url-list" key in the main area of the
metadata and perform BEP19 accordingly.

Retrieving metadata - "xs" parameter
------------------------------------

If the "xs" parameter is present then a client SHOULD try
to retrieve a .torrent file via the URL provided by the
"xs" parameter.

If a client does not understand the protocol in this URL
then it MUST silently ignore this specific "xs" parameter
with this URL.

If the retrieved file is not a valid .torrent file then the
.torrent file MUST be silently discarded.

If the Bittorrent Info Hash provided by the magnet URI
("xt=urn:btih:") does not match the "info" section of the
retrieved .torrent file then the .torrent file MUST be
silently discarded.

If a client is capable of BEP9 then it SHOULD use both
mechanisms in parallel.

Retrieving metadata - "as" parameter
------------------------------------

If no metadata could be retrieved with the mechanism
specified in BEP9 or with any provided "xs" parameter and
if an "as" parameter is present then a client SHOULD try to
retrieve a .torrent file via the URL provided by the "as"
parameter.

If a client does not understand the protocol in this URL
then it MUST silently ignore this specific "as" parameter
with this URL.

If the retrieved file is not a valid .torrent file then the
.torrent file MUST be silently discarded.

If the Bittorrent Info Hash provided by the magnet URI
("xt=urn:btih:") does not match the "info" section of the
retrieved .torrent file then the .torrent file MUST be
silently discarded.

Retrieving metadata - "ws" parameter
------------------------------------

The "ws" parameter SHOULD additionally be used in the
following way:

The URL provided by a "ws" parameter, excluding a possible,
following "/" character, MUST be appended by the string
".torrent" and the result SHOULD be interpreted as an "as"
parameter appended to the magnet URI.

Retrieving metadata and payload data - "cas" parameter
------------------------------------------------------

The "cas" parameter SHOULD additionally be used in the
following way:

The URL provided by a "cas" parameter MUST be appended by

1. A "/" character if the "cas" URL does not end with "/"
2. The Bittorrent Info Hash type:
   "btih" or "btmh" in lowercase letters ("[a-z]")
3. A "/" character
4. The Bittorrent Info Hash:
   40 or 64 characters in lowercase letters ("[0-9a-f]")

The result SHOULD be interpreted as an "ws"
parameter appended to the magnet URI.

When a magnet URI has multiple hash types,
then the client SHOULD use all resulting URLs.

<!-- TODO should the client prefer "btmh" over "btih"? -->

Client Implementation Notes
===========================

A client MUST NOT interpret an "as" parameter as an "xs"
parameter.

A client SHOULD NOT interpret an "xs" parameter as an "as"
parameter.

A client MUST NOT assume the applicability of this BEP for
any "xt" parameter other than "xt=urn:btih:". The "ws",
"as" and "xs" parameters might have different meanings for
other URIs provided by an "xt" parameter.

A client SHOULD NOT discard any "ws" parameter if one or
more "url-list" keys are available.

A client SHOULD NOT discard any "url-list" key if one or
more "ws" parameters are available.

Merge Additional Webseed URLs
-----------------------------

When the user adds a magnet link to a client
and when the client has already loaded this torrent
then the client should merge additional webseed URLs
just like it would merge additional tracker URLs.

Magnet URI Provider Notes
=========================

The provider of a magnet link MAY add an "xs" parameter to
decrease the latency for retrieving metadata if BEP9 or the
"as"/"ws" parameters are expected to have a negative
impact on usability (e.g. if there are only a few or no
Bittorrent peers).

The provider of a magnet link SHOULD NOT add an "xs"
parameter if the source is expected to have a "high" latency
or might not have sufficient bandwidth.

Considerations
==============

The "xt=urn:btih:" parameter
----------------------------

A common source of confusion so far was whether a Bittorrent
Info Hash is a URN refering to BitTorrent metadata or
whether it could be interpreted as a URN for the actual
payload data, too. Obviously the former is true, a sha1 hash
as used for the BitTorrent Info Hash is compliant with
RFC1737. The latter is more difficult to see. However it
might violate one particular requirement of RFC1737:

::

         Simple comparison: A comparison algorithm for URNs is simple,
         local, and deterministic. That is, there is a single algorithm for
         comparing two URNs that does not require contacting any external
         server, is well specified and simple.

If two torrents are refering to the same payload data it is
usually not possible to detect their equality without
contacting external network ressources.

RFC1737 does allow the usage of different comparison
algorithms for different authorities though. The question
however is what an authority is in the BitTorrent scenario.
Two possible interpretations exist:

Either a single "macrocosmic" authority, that is the public
specification of a Bittorrent Info hash and the laws of
maths inherent to a cryptograhic hash as being the
authority.

Or uncounted "microcosmic" authorities, that is every
BitTorrent Info Hash being an authority in itself, allowing
only the comparison of two identical Bittorrent "info"
sections.

The former case violates the "Simple comparison" requirement
described above. The latter does not seem to violate any
RFCs but it seems "uncommon" to consider a complete
<NID>:<NSS> pair (RFC2141) as an authority. In practice this
would make the "Simple comparison" requirement basically
superfluous and might therefore violate the intention behind
this requirement.

Therefore this BEP considers a magnet link with an
"xt=urn:btih:" refering to the BitTorrent Info Hash only
and not to the BitTorrent payload data. Even if the
"xt=urn:btih:" parameter were supposed to fullfil the
requirement described above in combination with other yet to
be specified magnet URI parameters in the future.

Therefore future BEPs MUST NOT change the interpretation of
the "ws", "as" or "xs" parameter if a "xt=urn:btih:"
parameter is present to avoid compatibility issues. A future
BEP MAY carefully add additional steps as long as
compatibility is ensured. A future BEP MAY change the
interpretation of the "ws", "as" or "xs" parameter if an
"xt=urn:btih:" parameter is absent.

This BEP SHOULD be declared deprecated if the
"xt=urn:btih:" became deprecated.

The "ws" (as well as "xt=urn:btih:" or "xt=urn:ed2k:"
or "xt=urn:kzhash:" ...) might not be in compliance with
the magnet URI rational, in that they are not protocol
agnostic, they are Bittorrent specific - they are "protocol
centric", not "data centric". Which would make it difficult
to ensure the universal applicability of a magnet URI (i.e.
a "data centric" approach would allow an application to use
any protocol it supports to fetch the according data). They
are not and might therefore never be "officially" supported
by the magnet URI draft. However they are easy to implement
in existing BitTorrent applications at the moment and no
format, algorithm or protocol supporting the translation of
a universal URN to a BitTorrent Info Hash exists as of
writing.

Security Considerations
-----------------------

magnet URIs have no inherent mechanism to ensure its
integrity, authenticity or confidentiality. It is therefore
RECOMMENDED to use a channel which fullfils the security
requirements of the provider and recipient of a magnet URI.

A user MAY add unauthenticated, additional "ws", "as" and
"xs" parameters as the BitTorrent Info Hash of the magnet
links still ensures the integrity and validity of data
received from untrusted sources. However a BitTorrent Info
Hash is not able to ensure confidentiality of the
communication with webseeds, this is highly dependant on the
protocol within these three parameters. If confidentiality
is an issue then the user SHOULD take additional steps on
other layers and a user might want to consider contacting
the operator of a webseed to discuss security concerns.

References
==========

-  http://magnet-uri.sourceforge.net/magnet-draft-overview.txt
-  http://bittorrent.org/beps/bep_0009.html
-  http://bittorrent.org/beps/bep_0017.html
-  http://bittorrent.org/beps/bep_0019.html
-  `https://tools.ietf.org/rfc/rfc1737.txt <http://www.rfc-editor.org/rfc/rfc1737.txt>`__
-  `https://tools.ietf.org/rfc/rfc2141.txt <http://www.rfc-editor.org/rfc/rfc2141.txt>`__

Copyright
=========

This document has been placed in the public domain.
