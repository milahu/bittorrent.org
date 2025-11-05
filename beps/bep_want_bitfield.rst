BEP XX: Want Bitfields of Leech-Only Clients
============================================

| **Author:** Milan Hauth <milahu@gmail.com>
| **Status:** Draft
| **Type:** Standards Track
| **Created:** 2025-11-05

Abstract
--------

| This BEP introduces an optional message type, the **Want Bitfield**,
  allowing leech-only peers (clients without completed pieces) to
  advertise which pieces they intend to download.
| By exposing this "interest map", seeders and partial-seed peers can
  anticipate which pieces will soon be requested and optimize upload
  scheduling or cache retention.

Motivation
----------

| In the base BitTorrent protocol (BEP 3), peers reveal
  which pieces they *have* but never which they *want*.
| A leecher with no pieces always sends an empty or all-zero
  ``bitfield``, leaving seeders unable to predict future requests.

| For efficient seeding and disk management-particularly in
  bandwidth-limited or ephemeral storage environments-seeders may wish
  to pre-position or retain pieces likely to be requested soon.
| A **Want Bitfield** gives them this predictive capability.

Existing workarounds and their limitations
------------------------------------------

Because the protocol lacks a way to learn what a leecher wants, some
operators use imperfect "guessing" strategies:

1. | **Pretend-All Strategy**
   | A partial seeder can *lie* by advertising a full ``bitfield``
     (claiming to have every piece).
   | When the leecher begins requesting specific pieces, the seeder
     learns exactly which ones are desired.
   | However, this deception violates protocol expectations: if the
     seeder delays actual delivery while producing or fetching the
     requested data, the leecher perceives high latency and may choke or
     ban the connection.

2. | **Pretend-More Strategy**
   | A seeder may instead advertise possession of *more pieces than it
     truly has*.
   | When the leecher requests any of these extra pieces, the seeder
     dynamically generates or retrieves them and fulfills the request;
     unrequested pieces are skipped.
   | This incremental "brute-force" discovery helps infer demand but
     still relies on misreporting.
   | Excessive latency or unfulfilled requests again risk reputation
     penalties or outright disconnection by honest peers.

| Both approaches depend on **lying about piece availability**, which
  undermines trust and interoperability.
| A standardized *Want Bitfield* lets leechers communicate interest
  directly, removing the need for deceptive signaling and the risk of
  punishment.

Rationale
---------

Providing an explicit *want* map benefits:

- **Predictive availability:** Seeders can cache or hold pieces before
  requests arrive.
- **Bandwidth efficiency:** Expected pieces can be prefetched or read
  into memory.
- **Network coordination:** Private swarms or CDN-style deployments can
  plan distribution based on collective demand.
- **Protocol integrity:** Eliminates the incentive for misreporting
  "have" data.

Specification
-------------

Negotiation
~~~~~~~~~~~

Implemented via the **Extension Protocol** (BEP 10).

During the extended handshake a peer advertises:

.. code:: json

   {
     "m": { "want_bitfield": <extension_id> }
   }

If both peers support it, they may exchange ``want_bitfield`` messages
after the handshake.

Message Format
~~~~~~~~~~~~~~

============== ================== =========================
Field          Type               Description
============== ================== =========================
``len``        4 bytes int        Message length
``ext_msg_id`` 1 byte             Extension message ID
``bitfield``   variable bit array Bits set = pieces desired
============== ================== =========================

| Length equals the torrent’s piece count, rounded to full bytes.
| Peers MAY send updates as priorities change.
| The message is advisory and does not affect standard ``interested`` or
  ``request`` flow.

Backward Compatibility
~~~~~~~~~~~~~~~~~~~~~~

Non-supporting peers ignore the new message per BEP 10 rules; core
behavior remains unchanged.

Security / Privacy
~~~~~~~~~~~~~~~~~~

| Advertising wants can reveal user interest patterns,
| but this interest pattern is also revealed through piece requests.
| Clients SHOULD provide an option to disable or randomize this feature.
| Seeders MUST treat want data as informational, not for access control
  or tit-for-tat enforcement.

Copyright
---------

This document is in the public domain.
