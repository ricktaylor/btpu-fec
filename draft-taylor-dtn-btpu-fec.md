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
area: INT
workgroup: "Delay/Disruption Tolerant Networking"

venue:
  group: "Delay/Disruption Tolerant Networking"
  type: "Working Group"
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
  BTPU:
    target: https://www.ietf.org/archive/id/draft-taylor-dtn-btpu-00.html
    title: Bundle Transfer Protocol - Unidirectional
    date: 2025-02

informative:

--- abstract

This document defines an extension to the Bundle Transfer Protocol - Unidirectional, as described in {{BTPU}}, to enable forward error correction (FEC) coding to be applied selectively to the transfer of individual bundles on a case by case basis.

The definition and use of FEC follows the FECFRAME framework defined in {{!RFC6363}}, and this document introduces new Message types to BTPU in order to carry the FEC information as defined in the framework.

--- middle

# Introduction

There are a number of use-cases of the Bundle Transfer Protocol - Unidirectional {{BTPU}}, where the use of transfer segment repetition as a mechanism to protect against the loss of frames can be considered sub-optimal.  This document describes an alternate mechanism based on forward error correction (FEC) coding, that requires increased computational complexity but fewer transmitted bits.  Rather than defining novel formats and registries for the variety of standardized FEC mechanisms, this document reuses the primitives and best practices of the FECFRAME framework, defined in {{RFC6363}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Mapping to RFC 6363

This document follows the framework defined in {{!RFC6363}}, mapping the concepts defined in {{Section 2 of RFC6363}} to this document in the following manner:

| FECFRAME Definition| BTPU Definition |
|:-------------------|:----------------|
|Application Data Unit (ADU)| Bundle|
|ADU Flow |This document considers all communication between a sender and receivers as a single ADU Flow, as the underlying BTPU link-layer protocol is expected to provide a single logical channel|
|Application Protocol|BTPU|
|FEC Repair Packet|The [Pre-agreed FEC Repair Message](#pre-agreed-repair) and [Explicit FEC Repair Message](#explicit-repair) act as the corresponding FEC Repair Packet|
|FEC Source Packet|The [Pre-agreed FEC Source Message](#pre-agreed-source) and [Explicit FEC Source Message](#explicit-source) act as the corresponding FEC Source Packet|
|Repair Flow| The series of Transfer Repair Messages act as the Repair Flow|
|Source Flow| The series of Transfer Segment and Transfer End Messages act as the Source Flow|
|Transport Protocol| The underlying BTPU link-layer protocol|

### FEC Instance ID

Every Transfer Message includes an unsigned 8-bit FEC Instance ID field.  The value of this field indicates the FEC scheme and parameters to be used to recover missing parts of the Bundle, and the mapping from FEC Instance ID to the associated scheme and parameters MUST be pre-agreed at both sender and receiver using an out of band mechanism.  FEC Instance ID zero (0) indicates that no FEC mechanism is in use for the Transfer.

The FEC Instance ID MUST be the same for all Messages concerned with an individual Transfer, i.e. the FEC scheme in use MUST NOT change mid-transfer.  If a receiver detects a change in FEC Instance ID during a Transfer, it MUST consider the Transfer cancelled, and ignore all future Messages associated with the Transfer.

TODO - Stuff!

# Message Definitions

All new Messages introduced in this document follow the common message format as defined in Section 4 of {{BTPU}}.

This specification deviates from the recommendation in {{Section 5.3 of RFC6363}} by placing the Explicit Source FEC Payload ID before the Source Data, as BTPU has no capability analogous to common header compression, as found in Robust Header Compression (ROHC) {{?RFC3095}}, and hence for consistency with other BTPU messages, the metadata precedes the data.

## Pre-agreed FEC Source Message {#pre-agreed-source}

The Pre-agreed FEC Source Message is used to encapsulate a source block of a Bundle Transfer that uses FEC with a pre-agreed configuration.

A Pre-agreed FEC Source Message has a type of TBD1. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Inst. ID  |   Explicit Source FEC Payload ID ...          :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  ... Source Block Data ...                    :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this source block is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Instance ID:
: The identifier of the pre-agreed FEC scheme and configuration in use for the Transfer, encoded as an 8-bit unsigned integer in network byte order.

Explicit Source FEC Payload ID:
: The Explicit Source FEC Payload ID as defined in {{Section 5.3.1 of RFC6363}}.

Source Block Data:
: The octets of the source block of the Transfer, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Instance ID, and Explicit Source FEC Payload ID.

## Explicit FEC Source Message {#explicit-source}

The Explicit FEC Source Message is used to encapsulate a source block of a Bundle Transfer that uses FEC with an explicit FEC scheme and configuration.

A Explicit FEC Source Message has a type of TBD2. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Enc. ID   | FEC-Scheme-Specific Information elements ...  :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Explicit Source FEC Payload ID ...                            :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  ... Source Block Data ...                    :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this source block is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Encoding ID:
: A FEC Encoding ID, as defined in {{Section 5.6 of RFC6363}}, encoded as an 8-bit unsigned integer in network byte order.

FEC-Scheme-Specific Information elements:
: Zero or more FEC-Scheme-Specific Information elements as defined in {{Section 5.5 of RFC6363}}.

Explicit Source FEC Payload ID:
: The Explicit Source FEC Payload ID as defined in {{Section 5.3.1 of RFC6363}}.

Source Block Data:
: The octets of the source block of the Transfer, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Encoding ID, FEC-Scheme-Specific Information elements, and Explicit Source FEC Payload ID.

## Pre-agreed FEC Repair Message {#pre-agreed-repair}

The Pre-agreed FEC Repair Message is used to encapsulate the repair symbols of a Bundle Transfer that uses FEC with a pre-agreed configuration.

A Pre-agreed FEC Repair Message has a type of TBD3. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Inst. ID  |            Repair FEC Payload ID ...          :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  ... Repair Symbol Data ...                   :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this source block is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Instance ID:
: The identifier of the pre-agreed FEC scheme and configuration in use for the Transfer, encoded as an 8-bit unsigned integer in network byte order.

Repair FEC Payload ID:
: The Repair FEC Payload ID as defined in {{Section 5.4.1 of RFC6363}}.

Repair Symbol Data:
: The octets of the repair symbols of the Transfer, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Instance ID, and Repair FEC Payload ID.

## Explicit FEC Repair Message {#explicit-repair}

The Explicit FEC Repair Message is used to encapsulate the repair symbols of a Bundle Transfer that uses FEC with an explicit FEC scheme and configuration.

A Explicit FEC Repair Message has a type of TBD4. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Enc. ID   | FEC-Scheme-Specific Information elements ...  :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Repair FEC Payload ID ...                                     :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  ... Repair Symbol Data ...                   :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this source block is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Encoding ID:
: A FEC Encoding ID, as defined in {{Section 5.6 of RFC6363}}, encoded as an 8-bit unsigned integer in network byte order.

FEC-Scheme-Specific Information elements:
: Zero or more FEC-Scheme-Specific Information elements as defined in {{Section 5.5 of RFC6363}}.

Repair FEC Payload ID:
: The Repair FEC Payload ID as defined in {{Section 5.4.1 of RFC6363}}.

Repair Symbol Data:
: The octets of the repair symbols of the Transfer, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Instance ID, and Repair FEC Payload ID.

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

TODO acknowledge.
