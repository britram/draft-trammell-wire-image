---
title: The Wire Image of a Network Protocol
abbrev: Wire Image
docname: draft-trammell-wire-image-latest
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [sortrefs, symrefs]

author:
  -
    ins: B. Trammell
    name: Brian Trammell
    org: ETH Zurich
    email: ietf@trammell.ch
    street: Gloriastrasse 35
    city: 8092 Zurich
    country: Switzerland
  -
    ins: M. Kuehlewind
    name: Mirja Kuehlewind
    org: ETH Zurich
    email: ietf@trammell.ch
    street: Gloriastrasse 35
    city: 8092 Zurich
    country: Switzerland

normative:

informative:

--- abstract

This document defines the wire image, an abstraction of the information
available to an on-path non-participant in a networking protocol. This
abstraction is intended to shed light on current discussions within the IETF
on the implications on increased encryption has for network functions that use
the wire image.

--- middle

# Introduction

A protocol specification defines a set of behaviors for each participant in
the protocol: which lower-layer protocols are used for which services, how
messages are formatted and protected, which participant sends which message
when, how each participant should respond to each message, and so on.

Implicit in a protocol specification is the information the protocol radiates
toward nonparticipant observers of the messages sent among participants. Any
information that has a clear definition in the protocol's message format(s),
or is implied by that definition, and is not cryptographically
confidentiality-protected can be unambiguously interpreted by those observers.

This information comprises the protocol's wire image, which we define and
discuss in this document. It is the wire image, not the protocol's
specification, that determines how third parties on the network paths among
protocol participants can interact with that protocol.

Several documents currently under discussion in IETF working groups and the
IETF in general, for example
{{?QUIC-MANAGEABILITY=I-D.ietf-quic-manageability}},
{{?EFFECT-ENCRYPT=I-D.mm-wg-effect-encrypt}}, and
{{?TRANSPORT-ENCRYPT=I-D.fairhurst-tsvwg-transport-encrypt}}, discuss in
part impacts on the third-party use of wire images caused by a migration from
protocols whose wire images are largely not confidentiality protected (e.g.
HTTP over TCP) to protocols whose wire images are confidentiality protected
(e.g. H2 over QUIC).

This document presents the wire image abstraction with the hope that it can
shed some light on these discussions.

# Definition

More formally, the wire image of a protocol consists of:

1. The sequence of messages sent by each participant in the protocol, each
   expressed as a sequence of bits with an associated arbitrary-precision time
   at which it was sent.
2. The vectore of the values of any arbitrary set of functions applied to the
   sequence (1) above.

# Discussion

This definition is so vague as to be difficult to apply to protocol analysis,
but it does illustrate some important properties of the wire image:

- The wire image is not limited to merely "the unencrypted bits in the
  header". In particular, timing, size, and sequence information can be used
  to infer other parameters of the behavior of the protocol, or to fingerprint
  protocols and/or specific implmentations of the protocol.

- The wire image is multidimensional. This implies that the name "image" is
  not merely metaphorical, and that general image recognition techniques can
  be applied to extracting paterns and information from it.

## Obscuring timing and sizing information

Cryptography can protect the confidentiality of a protocol's headers, to the
extent that forwarding devices do not need the confidentiality-protected
information for basic forwarding operations. However, it cannot be applied to
protecting non-header information in the wire image. Of particular interest is
the sequence of packet sizes and the sequence of packet times. These are
characteristic of the operation of the protocol. A sender may use padding to
increase the size of packets, and inject delay into sending in order to
increase delay components, however it does this as the expense of bandwidth
efficiency and latency. This technique is therefore limited to the tolerance
for inefficiency and latency of the application.

## Integrity Protection of the Wire Image

Portions of the wire image of a protocol that are neither
confidentiality-protected nor integrity-protected are writable by devices on a
path. A protocol with a wire image that is largely writable operating over a
path with devices that understand the semantics of the protocol's wire image
can modify it, in order to induce behaviors at the protocol's participants.
This is the case with TCP in the modern Internet.

Adding end-to-end integrity protection to portions of the wire image makes
those portions nonwritable. Nonwritable portions of the wire image may be
observed by on-path devices, but not changed.

Note that a protocol's wire image cannot be made completely nonwritable in a
packet-switched network: the observed delay sequence is modified as packets
move through the network and experience different delays on different links,
packets may always be dropped, as a consequence of the network's operation.
Intermediate systems with knowledge of the protocol semantics in the readable
portion of the wire image can also purposely delay or drop packets in order to
affect the protocol's operation.

## Engineering the Wire Image

We note that understanding the nature of a protocol's wire image allows it to
be engineered. The general principle at work here, observed through experience
with deployability and non-deployability of protocols at the network and
transport layers in the Internet, is that all observable parts of a protocol's
wire image will eventually ossify, and become difficult or impossible to
change in future extensions or revisions of the protocol.

A network function which serves a purpose useful to its deployer will use the
information it needs from the wire image, and will tend to get that
information from the wire image in the simplest way possible. A protocol's
wire image should therefore be explicitly designed to explicitly expose
information to those network functions in an obvious way, and to expose as
little other information as possible.



# Acknowledgments

This work is partially supported by the European Commission under Horizon 2020
grant agreement no. 688421 Measurement and Architecture for a Middleboxed
Internet (MAMI), and by the Swiss State Secretariat for Education, Research, and
Innovation under contract no. 15.0268. This support does not imply endorsement.
