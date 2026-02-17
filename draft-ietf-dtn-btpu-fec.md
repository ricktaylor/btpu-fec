---

title: Forward Error Correction for the Bundle Transfer Protocol
abbrev: BTPU-FEC
category: std

docname: draft-ietf-dtn-btpu-fec-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: INT
workgroup: Delay/Disruption Tolerant Networking

venue:
  group: Delay/Disruption Tolerant Networking
  type: Working Group
  mail: dtn@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/dtn/
  github: ricktaylor/btpu-fec
  latest: https://ricktaylor.github.io/btpu-fec/draft-ietf-dtn-btpu-fec.html

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
    target: https://datatracker.ietf.org/doc/draft-ietf-dtn-btpu/
    title: Bundle Transfer Protocol - Unidirectional
    date: 2025-02

informative:

--- abstract

This document defines an optional extension to the Bundle Transfer Protocol - Unidirectional, as described in {{BTPU}}, to enable forward error correction (FEC) coding to be applied selectively to the transfer of individual bundles on a case by case basis.

The definition and use of FEC follows the FECFRAME framework defined in {{!RFC6363}}, and this document introduces new Message types to BTPU in order to carry the FEC information as defined in the framework.

--- middle

# Introduction

There are a number of use-cases of the Bundle Transfer Protocol - Unidirectional {{BTPU}}, where the use of transfer segment repetition as a mechanism to protect against the loss of frames can be considered sub-optimal.  This document describes an alternate mechanism based on forward error correction (FEC) coding, that requires increased computational complexity but fewer transmitted bits.

Rather than defining novel formats and registries for the variety of standardized FEC mechanisms, this document reuses the primitives and best practices of the FECFRAME framework, defined in {{RFC6363}}.

Just as in core BTPU, a Bundle is split into a series of octet sequences that are emitted into Messages by the sender to be transported to receivers by the underlying link-layer protocol; but when FEC is desired, the Bundle is divided into FECFRAME Application Data Units (ADUs), and the mechanisms defined in the FECFRAME framework are used to produce Repair Symbols that are placed into new Messages, rather than just sub-slices of the original Bundle.  The new Messages are used to distinguish FEC Source and Repair data from core BTPU Segments.

Although the content and processing of the new Messages differs from existing BTPU Messages, the rules around the emission and replication of the Messages are identical to the rules applicable to the core BTPU Segment Messages, and they follow the common BTPU Message format, allowing implementations that do not support this extension to efficiently detect and ignore the new Messages.

## Applicability

Note that when FEC is available at the link-layer it is generally more effective than applying it at the Transfer layer, and ought to be used when it is available.  This extension is designed to provide FEC capabilities when the underlying link-layer protocol does not have native support for FEC, or when per-Transfer FEC is desired by a deployment.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This document uses FEC terminology from {{RFC6363}}.  In particular, the term "Application Data Unit" (ADU) refers to the unit of source data as defined in {{Section 2 of RFC6363}}, not the Application Data Unit defined in {{?RFC9171}}.  In the context of this document, an ADU is a chunk of a Bundle that is provided to the FEC scheme for encoding.

# Protocol Overview

Rather than updating the Segment Messages defined in BTPU, this extension introduces two new pairs of Messages to carry the source and repair symbols of a BTPU Bundle protected with FEC.  The use of new types allows a deployment to select the use of FEC as appropriate on a per-transfer basis, perhaps associated with some upper layer concept of reliability for a particular transfer, or change in transmission environment.

In the language of {{Section 2 of RFC6363}}, the FEC Source Messages act as FEC Source Packets, and the FEC Repair Messages as FEC Repair Packets.  Within the context of a particular Transfer, the sequence of FEC Source Messages is considered the Source Flow, and the sequence of FEC Repair Messages the Repair Flow.

The source and repair Messages are grouped into two pairs:

Pre-agreed FEC:
: The [Pre-agreed FEC Source](#pre-agreed-source) and [Pre-agreed FEC Repair](#pre-agreed-repair) Messages provide a wire-efficient format, for use when the FEC Framework Configuration Information {{Section 5.5 of RFC6363}} has been pre-agreed via some a-priori configuration or out-of-band mechanism.  Each Message carries an 8-bit FEC Instance ID that references pre-configured FEC scheme information.

Explicit FEC:
: The [Explicit FEC Source](#explicit-source) and [Explicit FEC Repair](#explicit-repair) Messages include the FEC-Scheme-Specific Information (FSSI) in the Message content, allowing for the ad-hoc use of different FEC schemes and configuration, given underlying implementation support.  Each Message carries an 8-bit FEC Encoding ID and the FSSI elements required by the FEC scheme.

The following table summarizes the differences between the two approaches:

| Aspect | Pre-agreed FEC | Explicit FEC |
|:-------|:---------------|:-------------|
| Configuration | Out-of-band or a-priori | Self-describing |
| FEC Identifier | FEC Instance ID (8-bit) | FEC Encoding ID (8-bit) |
| Per-Message Overhead | Lower (no FSSI) | Higher (includes FSSI) |
{: #tab-fec-comparison align="left" title="Comparison of Pre-agreed and Explicit FEC"}

Irrespective of whether Pre-agreed or Explicit FEC is in use for a Transfer, the FEC Framework Configuration Information MUST NOT change mid-transfer.  If a receiver detects a change in FEC Framework Configuration Information during a Transfer, it MUST consider any incomplete Transfer affected by the change as cancelled, as defined in Section 5.4 of {{BTPU}}.

## Pre-agreed FEC Instance ID  {#instance-id}

When pre-agreed FEC is desired, a lookup table MUST be configured at the sender and all receivers that maps a unique identifier, the FEC Instance ID, to a particular FEC scheme and corresponding FSSI, such that each [Pre-agreed FEC Source](#pre-agreed-source) and [Pre-agreed FEC Repair](#pre-agreed-repair) Message can refer to the FEC mechanism in use by referencing the FEC Instance ID, rather than including all the FEC configuration information in each Message.

The FEC Instance ID is an unsigned integer in the range 0..255 inclusive, and is carried in the respective FEC Messages encoded in the FEC Instance ID field.  Just like the FEC scheme and configuration, the FEC Instance ID MUST be the same for all Messages concerned with an individual Transfer.  If a receiver detects a change in FEC Instance ID during a Transfer, it MUST consider the Transfer cancelled, as defined in Section 5.4 of {{BTPU}}.

Configuration of the mapping of FEC Instance ID to FEC scheme information MUST be performed out-of-band, or via an a-priori configuration mechanism.

## FEC Transfer Operation

FEC Messages share the same Transfer Number space as the core BTPU Transfer Messages, and the Transfer Window algorithm defined in {{BTPU}} applies to FEC Transfers.  However, a sender MUST NOT mix FEC Messages and core BTPU Transfer Messages (Transfer Segment or Transfer End) within the same Transfer.  If a receiver detects such mixing, it MUST consider the Transfer cancelled, as defined in Section 5.4 of {{BTPU}}.  The Transfer Cancel Message, as defined in {{BTPU}}, MAY be used to cancel an FEC Transfer.

Unlike core BTPU, FEC Transfers do not use an explicit Transfer End Message to signal completion.  Instead, the FEC scheme determines when sufficient ADUs and Repair Symbols have been received to reconstruct the original bundle.  The Transfer Window algorithm provides the receiver with an upper bound on how long to wait for FEC Messages associated with a given Transfer before considering it complete or failed.  The Bundle Length Hint, if present, can be used to verify that the reconstructed bundle has the expected size.

# Message Definitions

All new Messages introduced in this document follow the common message format as defined in Section 4 of {{BTPU}}, and Hint Items MAY be included in these Messages.  The Bundle Length Hint, as defined in {{BTPU}}, MAY be included in FEC Source and FEC Repair Messages to signal the total length of the bundle being transferred.

This specification deviates from the recommendation in {{Section 5.3 of RFC6363}} by placing the Explicit Source FEC Payload ID before the Source Data, as BTPU has no capability analogous to common header compression, as found in Robust Header Compression (ROHC) {{?RFC3095}}, and therefore to maintain consistency with other BTPU messages, the metadata precedes the data.

Unlike the Transfer Segment and Transfer End Messages defined in {{BTPU}}, FEC Messages do not include a Segment Index field.  The FEC Payload IDs, as defined by the FEC scheme in use, serve the equivalent role of identifying and ordering source blocks and repair symbols for reassembly.

## Pre-agreed FEC Source Message {#pre-agreed-source}

The Pre-agreed FEC Source Message is used to encapsulate an Application Data Unit (ADU), as defined in {{Section 2 of RFC6363}}, of a Bundle Transfer that uses FEC with a pre-agreed configuration.  Multiple ADUs together form a Source Block for FEC encoding.

A Pre-agreed FEC Source Message has a type of TBD1. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Inst. ID  |   Explicit Source FEC Payload ID ...          :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                       ... Source Data ...                     :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this ADU is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Instance ID:
: The [FEC Instance ID](#instance-id) of the pre-agreed FEC scheme and configuration in use for the Transfer, encoded as an 8-bit unsigned integer in network byte order.

Explicit Source FEC Payload ID:
: The Explicit Source FEC Payload ID, with the format defined by the FEC scheme.  It is RECOMMENDED that FEC schemes support the Generic Explicit Source FEC Payload ID format defined in {{Section 5.3.1 of RFC6363}}.

Source Data:
: The octets of the ADU, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Instance ID, and Explicit Source FEC Payload ID.

## Explicit FEC Source Message {#explicit-source}

The Explicit FEC Source Message is used to encapsulate an Application Data Unit (ADU), as defined in {{Section 2 of RFC6363}}, of a Bundle Transfer that uses FEC with an explicit FEC scheme and configuration.  Multiple ADUs together form a Source Block for FEC encoding.

An Explicit FEC Source Message has a type of TBD2. The Message Content field is formatted as follows:

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Transfer Number                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | FEC Enc. ID   | FEC-Scheme-Specific Information elements ...  :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Explicit Source FEC Payload ID ...                            :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                       ... Source Data ...                     :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Transfer Number:
: The numeric identifier of the Transfer that this ADU is part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Encoding ID:
: A FEC Encoding ID, as defined in {{Section 5.6 of RFC6363}}, encoded as an 8-bit unsigned integer in network byte order.

FEC-Scheme-Specific Information elements:
: Zero or more FEC-Scheme-Specific Information elements as defined in {{Section 5.5 of RFC6363}}.  The binary encoding format and length of these elements is defined by the FEC scheme identified by the FEC Encoding ID.

Explicit Source FEC Payload ID:
: The Explicit Source FEC Payload ID, with the format defined by the FEC scheme.  It is RECOMMENDED that FEC schemes support the Generic Explicit Source FEC Payload ID format defined in {{Section 5.3.1 of RFC6363}}.

Source Data:
: The octets of the ADU, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Encoding ID, FEC-Scheme-Specific Information elements, and Explicit Source FEC Payload ID.

## Pre-agreed FEC Repair Message {#pre-agreed-repair}

The Pre-agreed FEC Repair Message is used to encapsulate the Repair Symbols ({{Section 2 of RFC6363}}) of a Bundle Transfer that uses FEC with a pre-agreed configuration.

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
: The numeric identifier of the Transfer that these repair symbols are part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Instance ID:
: The [FEC Instance ID](#instance-id) of the pre-agreed FEC scheme and configuration in use for the Transfer, encoded as an 8-bit unsigned integer in network byte order.

Repair FEC Payload ID:
: The Repair FEC Payload ID as defined in {{Section 5.4 of RFC6363}}, with the format specified by the FEC scheme.

Repair Symbol Data:
: The octets of the repair symbols, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Instance ID, and Repair FEC Payload ID.

## Explicit FEC Repair Message {#explicit-repair}

The Explicit FEC Repair Message is used to encapsulate the Repair Symbols ({{Section 2 of RFC6363}}) of a Bundle Transfer that uses FEC with an explicit FEC scheme and configuration.

An Explicit FEC Repair Message has a type of TBD4. The Message Content field is formatted as follows:

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
: The numeric identifier of the Transfer that these repair symbols are part of, encoded as a 32-bit unsigned integer in network byte order.

FEC Encoding ID:
: A FEC Encoding ID, as defined in {{Section 5.6 of RFC6363}}, encoded as an 8-bit unsigned integer in network byte order.

FEC-Scheme-Specific Information elements:
: Zero or more FEC-Scheme-Specific Information elements as defined in {{Section 5.5 of RFC6363}}.  The binary encoding format and length of these elements is defined by the FEC scheme identified by the FEC Encoding ID.

Repair FEC Payload ID:
: The Repair FEC Payload ID as defined in {{Section 5.4 of RFC6363}}, with the format specified by the FEC scheme.

Repair Symbol Data:
: The octets of the repair symbols, with the length calculated as the Message content length excluding the length of the Transfer Number, FEC Encoding ID, FEC-Scheme-Specific Information elements, and Repair FEC Payload ID.

# Security Considerations

The new Messages and mechanisms in this document do not add additional security considerations, nor impact the existing security considerations outlined in {{BTPU}} and {{RFC6363}}.

FEC mechanisms do not provide authentication or integrity protection.  Malicious or corrupted FEC Messages could cause a receiver to reconstruct an incorrect bundle.  If a receiver detects an error during FEC decoding, it SHOULD cancel the Transfer as defined in Section 5.4 of {{BTPU}}.  Additionally, deployments SHOULD use upper-layer integrity mechanisms, such as BPSec {{!RFC9172}}, to detect corruption in reconstructed bundles.  When upper-layer integrity verification fails, implementations SHOULD discard the reconstructed bundle as per the upper-layer's security policy.

# IANA Considerations

IANA is requested to assign new values from the "BTPU Message Types" registry for the new Message types defined in this document:

| Value | Message Type |
|:------|:-------------|
| TBD1  | Pre-agreed FEC Source Message |
| TBD2  | Explicit FEC Source Message |
| TBD3  | Pre-agreed FEC Repair Message |
| TBD4  | Explicit FEC Repair Message |
{: #tab-iana-message-types align="left" title="New BTPU Message Types"}

--- back

# Acknowledgments

The author is indebted to the authors of the FECFRAME framework, and hopes that its successful application in areas outside RTP validates all the obvious hard work that went into making RFC6363 generic and reusable.
