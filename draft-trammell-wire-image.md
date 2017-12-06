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
    email: mirja.kuehlewind@tik.ee.ethz.ch
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
protocol participants will interact with that protocol.

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

More formally, the wire image of a protocol consists of the sequence of
messages sent by each participant in the protocol, each expressed as a
sequence of bits with an associated arbitrary-precision time at which it was
sent.

# Discussion

This definition is so vague as to be difficult to apply to protocol analysis,
but it does illustrate some important properties of the wire image.

Key is that the wire image is not limited to merely "the unencrypted bits in the
header". In particular, interpacket timing, packet size, and message sequence
information can be used to infer other parameters of the behavior of the
protocol, or to fingerprint protocols and/or specific implementations of the
protocol; see {{time-and-size}}.

An important implication of this property is that a protocol which uses
confidentiality protection for the headers it needs to operate can be
deliberately designed to have a specified wire image that is separate from
that machinery; see {{engineering}}. Note that this is a capability unique to
encrypted protocols. Parts of a wire image may also be made visible to devices
on path, but immutable through end-to-end integrity protection; see
{{integrity}}.

Note also that the wire image is multidimensional. This implies that the name
"image" is not merely metaphorical, and that general image recognition
techniques may be applicable to extracting paterns and information from it.

## Obscuring timing and sizing information {#time-and-size}

Cryptography can protect the confidentiality of a protocol's headers, to the
extent that forwarding devices do not need the confidentiality-protected
information for basic forwarding operations. However, it cannot be applied to
protecting non-header information in the wire image. Of particular interest is
the sequence of packet sizes and the sequence of packet times. These are
characteristic of the operation of the protocol. A sender may use padding to
increase the size of packets, and inject delay into sending in order to
increase delay components. However it does this as the expense of bandwidth
efficiency and latency. This technique is therefore limited to the tolerance
for inefficiency and latency of the application.

## Integrity Protection of the Wire Image {#integrity}

Portions of the wire image of a protocol that are neither
confidentiality-protected nor integrity-protected are writable by devices on a
path. A protocol with a wire image that is largely writable operating over a
path with devices that understand the semantics of the protocol's wire image
can modify it, in order to induce behaviors at the protocol's participants.
This is the case with TCP in the modern Internet.

Adding end-to-end integrity protection to portions of the wire image makes it
impossible for on-path devices to modify them without detection by the
endpoints, which can then take action in response to those modifications,
making these portions of the wire image effectively immutable. However, they
can still be observed by devices on path.

Note that a protocol's visible wire image cannot be made completely immutable
in a packet-switched network. Interarrival timings, for instance, cannot be
easily protected, as the observable delay sequence is modified as packets move
through the network and experience different delays on different links.
Message sequences are also not practically protectable, as packets may be
dropped or reordered at any point in the network, as a consequence of the
network's operation.

Intermediate systems with knowledge of the protocol semantics in the readable
portion of the wire image can also purposely delay or drop packets in order to
affect the protocol's operation.

## Engineering the Wire Image {#engineering}

We note that understanding the nature of a protocol's wire image allows it to
be engineered. The general principle at work here, observed through experience
with deployability and non-deployability of protocols at the network and
transport layers in the Internet, is that all observable parts of a protocol's
wire image will eventually ossify, and become difficult or impossible to
change in future extensions or revisions of the protocol.

A network function which serves a purpose useful to its deployer will use the
information it needs from the wire image, and will tend to get that
information from the wire image in the simplest way possible. A protocol's
wire image should therefore be designed to explicitly expose information to
those network functions in an obvious way, and to expose as little other
information as possible.

However, even when information is explicitly provided to the network, any
information that is exposed by the wire image, even that informaiton not
intended to be consumed by an observer, must be designed carefully as it might
ossify, making it immutable for future versions of the protocol. For example,
information needed to support decryption by the receiving endpoint
(cryptographic handshakes, sequence numbers, and so on) may be used by the
path for its own purposes.

# Acknowledgments

This work is partially supported by the European Commission under Horizon 2020
grant agreement no. 688421 Measurement and Architecture for a Middleboxed
Internet (MAMI), and by the Swiss State Secretariat for Education, Research, and
Innovation under contract no. 15.0268. This support does not imply endorsement.
