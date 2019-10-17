---
title: Protected Headers for Cryptographic E-mail
docname: draft-autocrypt-protected-headers-00
date: 2019-10-14
category: info

ipr: trust200902
area: int
workgroup: openpgp
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: B. R. Einarsson
    name: Bjarni Runar Einarsson
    org: Mailpile ehf
    street: Baronsstigur
    country: Iceland
    email: bre@mailpile.is
 -
    ins: D. K. Gillmor
    name: Daniel Kahn Gillmor
    org: American Civil Liberties Union
    street: 125 Broad St.
    city: New York, NY
    code: 10004
    country: USA
    abbrev: ACLU
    email: dkg@fifthhorseman.net
informative:
normative:
 RFC2119:
 RFC2822:
 RFC3156:
 RFC5751:
 RFC8174:
 RFC8551:
 I-D.draft-luck-lamps-pep-header-protection-03:
--- abstract

This document describes a common strategy to extend the cryptographic protections provided by PGP/MIME etc. to also protect message headers. In particular, how to encrypt the Subject line.

--- middle

Introduction
============

The standards for cryptographic e-mail, PGP/MIME and S/MIME ({{RFC3156}} and {{RFC8551}}), can provide both integrity and confidentiality to the body of a MIME e-mail message. However, PGP/MIME alone provides no protection to message headers and the mechanism specified for header protection within S/MIME has not seen widespread adoption.

This document defines a scheme, "Protected Headers for Cryptographic E-mail", which has been adopted by multiple existing e-mail clients in order to extend the cryptographic protections provided by PGP/MIME to also protect the {{RFC2822}} message headers.

In particular, we describe how to encrypt the Subject line and how to preserve backwards compatibility so an encrypted subject remains available to recipients using software that does not implement support for the Protected Headers scheme.

We also discuss some of the compatibility constraints and usability concerns which motivated the design of the scheme, as well as limitations.

The authors believe the technique is broadly applicable would also apply to other MIME-compatible cryptographic e-mail systems, including S/MIME.  Furthermore, this technique has already proven itself as a useful building block for other improvements to cryptographic e-mail, such as the Autocrypt Level 1.1 "Gossip" mechanism.


Requirements Language
---------------------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.


Terminology
-----------

For the purposes of this document, we define the following concepts:

   * *MUA* is short for Mail User Agent; an e-mail client.
   * *Protection* of message data refers to cryptographic encryption and/or signatures, providing authenticity, confidentiality or both.
   * *Cryptographic Envelope* is all MIME structure directly dictated by the cryptographic e-mail system in use.
   * *Cryptographic Payload* is all message data protected by the Cryptographic Envelope.
   * *Original Headers* are the RFC2822 message headers as known to the sending MUA at the time of message composition.
   * *Protected Headers* are any headers protected by the scheme described in this document.
   * *Exposed Headers* are any headers outside the Cryptographic Payload (protected or not).
   * *Obscured Headers* are any headers which have been modified or removed from the set of Exposed Headers.
   * *Legacy Display Part* is a MIME construct which guarantees visibility of data from the Original Headers which may have been removed or obscured from the Unprotected Headers.

The Cryptographic Envelope fully encloses the Cryptographic Payload, whether the message is signed or encrypted. The Original Headers, aside from Content-Type headers which directly pertain to the cryptographic structure, are considered to be outside of both.


Message Composition
===================

The Protected Headers scheme is summarized as follows:

   1. All message headers known to the MUA at composition time MUST be copied verbatim into the first MIME header of the Cryptographic Payload.
   2. When encrypting, Exposed Headers MAY be *obscured* by a transformation (including deletion).
   3. When encrypting, if the MUA has obscured any user-facing header data, it SHOULD add a Legacy Display Part to the Cryptographic Payload which duplicates this information.

*Note:* The above is not a description of a sequential algorithm.

Details:

   * Encryption SHOULD protect the Subject line
   * When encrypting, the Subject line should be obscured (replaced) by the string "..."
   * Step 3. may require adding a `multipart/mixed` MIME wrapper to the Cryptographic Payload, in turn influencing where to inject the headers from step 1.

See below for a more detailed discussion.


Header Copying
--------------

All headers known to the MUA at composition time MUST be copied verbatim into the header of the target MIME part.

The target MIME part shall always be the first MIME part within the Cryptographic Payload. See Examples below for an illustration.

The reason all headers must be copied, is that otherwise it becomes impossible for implementations to reliably detect tampering with the Exposed Headers, which would greatly reduces the strength of the scheme.

Encrypted Subject
-----------------

When a message is encrypted, the Subject should be obscured by replacing the Exposed Subject with three periods: ...

This value (...) was chosen because it is believed to be language agnostic and avoids communicating any potentially misleading information to the recipient (see Common Pitfalls below for a more detailed discussion).

Obscured Headers
----------------

Due to compatibility and usability concerns, a Mail User Agent SHOULD NOT obscure any of: `From`, `To`, `Cc`, `Message-ID`, `References`, `Reply-To`, (FIXME: MORE?), unless the user has indicated they have security constraints which justify the potential downsides (see Common Pitfalls below for a more detailed discussion).

Aside from that limitation, this specification does not at this time define or limit the methods a MUA may use to convert Exposed Headers into Obscured Headers.

Legacy Display
--------------


Message Interpretation
======================

(Brief discussion about potential strategies to reverse the process above.)


Examples
========

(Use diagrams from Juga to explain the envelope/payload concept)

(Add sample messages using this syntax, where E or S tell us which bits are part of the Cryptographic Payload.)

    . From: Alice Lovelace <alice@example.org>
    . To: Bob Babbage <bob@example.org>
    . Subject: ...
    . Content-Type: multipart/encrypted; protocol=...; boundary=1234
    .
    . --1234
    . Content-Type: application/pgp-encrypted
    .
    . Version: 1
    .
    . --1234
    . Content-Type: application/octet-stream; name="encrypted.asc"
    .
    E Content-Type: multipart/mixed; boundary=ABCD
    E (copied headers)
    E
    E --ABCD
    E (Legacy display part)
    E
    E --ABCD
    E (original message)
    E --ABCD--
    . --1234--



Common Pitfalls and Guidelines
==============================

Misunderstood Obscured Subjects
-------------------------------

(describe why Encrypted Message is a dangerous subject line)


Reply/Forward Losing Subjects
-----------------------------

(describe Re: ...)


Usability Impact of Reduced Metadata
-------------------------------------

(describe the problems ProtonMail/TutaNota have, discuss potential solutions)


Usability Impact of Obscured Message-ID
---------------------------------------

(describe why we don't recommend this just yet)


Usability Impact of Obscured From/To/Cc
---------------------------------------

(describe why we don't recommend this just yet)


IANA Considerations
===================

FIXME: register flag for legacy-display part


Document Considerations
=======================

\[ RFC Editor: please remove this section before publication ]

This document is currently edited as markdown.  Minor editorial changes can be suggested via merge requests at https://github.com/autocrypt/protected-headers or by e-mail to the authors.  Please direct all significant commentary to the public IETF LAMPS mailing list: spasm@ietf.org

Document History
----------------

Acknowledgements
================

Used to be Memory Hole, Autocrypt working group, OpenPGP e-mail summit attendees. Others?