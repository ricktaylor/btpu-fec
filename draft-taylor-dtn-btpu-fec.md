---

title: "Forward Error Correction for the Bundle Transfer Protocol"
abbrev: "BTPU-FEC"
category: std

docname: draft-taylor-dtn-btpu-fec-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Delay/Disruption Tolerant Networking"

venue:
  group: "Delay/Disruption Tolerant Networking"
  type: ""
  mail: "dtn@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dtn/"
  github: "ricktaylor/btpu-fec"
  latest: "https://ricktaylor.github.io/btpu-fec/draft-taylor-dtn-btpu-fec.html"

keyword:

- DTN
- BPv7
- Bundle Protocol
- BTPU
- FEC

author:

- fullname: Rick Taylor
  organization: Aalyria Technologies
  email: rtaylor@aalyria.com

normative:

informative:

--- abstract

TODO Abstract

--- middle

# Introduction

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Forward Error Correction {#fec}

To reduce the overhead associated with repeating Messages, the protocol supports the use of forward error correction, by emitting [Transfer Repair Messages](#transfer-repair-message) containing information that can be used to recover lost Messages.

Every Transfer Message includes an unsigned 8-bit FEC Instance ID field.  The value of this field indicates the FEC scheme and parameters to be used to recover missing parts of the Bundle, and the mapping from FEC Instance ID to the associated scheme and parameters MUST be pre-agreed at both sender and receiver using an out of band mechanism.  FEC Instance ID zero (0) indicates that no FEC mechanism is in use for the Transfer.

The FEC Instance ID MUST be the same for all Messages concerned with an individual Transfer, i.e. the FEC scheme in use MUST NOT change mid-transfer.  If a receiver detects a change in FEC Instance ID during a Transfer, it MUST consider the Transfer cancelled, and ignore all future Messages associated with the Transfer.

### Mapping to RFC 6363

For all non-zero values of FEC Instance ID, the protocol follows the framework defined in {{!RFC6363}}, mapping the concepts defined in {{Section 2 of RFC6363}} to this protocol in the following manner:

Application Data Unit (ADU):
: In this protocol a Bundle is the ADU.

ADU Flow:
: This protocol considers all communication between a sender and receiver as a single ADU Flow, as the link-layer protocol is expected to be point-to-point in nature.

Application Protocol:
: This protocol.

Content Delivery Protocol (CDP):
: This protocol is considered a CDP.

FEC Framework Configuration Information:
: This protocol requires FEC framework configuration information to be delivered via some out-of-band mechanism or a-priori configuration.

FEC Repair Packet:
: The [Transfer Repair Message](#transfer-repair-message) acts as the corresponding FEC Repair Packet.

FEC Source Packet:
: The Transfer Segment Message and Transfer End Message act as the corresponding FEC Source Packet.

Repair Flow:
: The series of Transfer Repair Messages act as the Repair Flow.

Repair FEC Payload ID:
: See [Repair FEC Payload ID](#fec-repair).

Source Flow:
: The series of Transfer Segment and Transfer End Messages act as the Source Flow.

Source FEC Payload ID:
: The combination of FEC Instance ID, Transfer Number, and Segment Sequence Number provide suitable Source FEC Payload ID.

Transport Protocol:
: The underlying link-layer protocol.

Other definitions in RFC 6363 remain as defined in that document.

### FEC Instance ID

TODO - Stuff!

### Repair FEC Payload ID {#fec-repair}

The combination of FEC Instance ID and Transfer Number provide a part of the Repair FEC Payload ID, but additional information is expected to be required in Transfer Repair Messages, depending on the FEC scheme in use.

## Transfer Repair Message

The Transfer Repair Message is used to carry repair information when Forward Error Correction is in use, see [](#fec).  This message corresponds to the "Repair Packet" in {{Section 5.1 of RFC6363}}.

A Transfer Repair Message has a type of TBD. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                               | FEC Inst. ID  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                ... Repair FEC Payload ID ...                  :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                 ... Segment Repair Data ...                   :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the in-progress Transfer that this Segment is part of, encoded as a 24-bit unsigned integer in network byte order.

FEC Instance ID:
: The identifier of the FEC scheme and parameters in use for this Transfer, see [Forward Error Correction](#fec).

Repair FEC Payload ID:
: See [Repair FEC Payload ID](#fec-repair).

Segment Repair Data:
: The octets of the repair data for a Segment of the Transfer, with the length calculated as the Message content length excluding the 12 octets of the Transfer Number, FEC Instance, Segment Sequence Number, and Segment Data Length.

Transfer Repair Messages SHOULD NOT have zero octets of Segment Repair Data, i.e. the total length of the Message SHOULD be greater than 16 octets.  Such Messages only add control-plane overhead and SHOULD NOT be used as an alternative form of padding.

Transfer Repair Messages With FEC Instance ID zero (0) are not valid.  They SHOULD NOT be emitted by a sender and MUST be ignored by a receiver.

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

TODO acknowledge.
