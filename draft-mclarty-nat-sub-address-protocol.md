---
title: NAT Sub Address Protocol
abbrev: NATSAP
category: info

docname: draft-mclarty-nat-sub-address-protocol-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Network Address Translators"
keyword:
 - NATSAP
 - DSAAP
 - NATSAP Sub-Address
 - CG NAT
venue:
  group: "Network Address Translators"
  type: ""
  mail: "nat@ietf.org"
  arch: "ftp://ftp.ietf.org/ietf-mail-archive/nat/"
  github: "Daniel-McLarty/NAT-Sub-Address-Protocol"
  latest: "https://Daniel-McLarty.github.io/NAT-Sub-Address-Protocol/draft-mclarty-nat-sub-address-protocol.html"

author:
 -
    fullname: "Daniel McLarty"
    organization: Independent
    email: "daniel@mclarty.tech"

normative:

informative:


--- abstract

This document defines the NAT Sub-Address Protocol (NATSAP), a Layer 5 encapsulation protocol designed to facilitate seamless bidirectional communication with devices behind Carrier-Grade NAT (CG NAT). NATSAP introduces dynamic sub-addresses assigned by the NAT router, which external clients can use alongside the public IP to route traffic back to internal devices without requiring traditional port forwarding. This document also defines the Dynamic Sub-Address Assignemnt Protocol (DSAAP), to facilitate the acquiring of a NATSAP Sub-Address.

The protocol offers backward compatibility with existing IPv4 infrastructure, efficient DNS-based service discovery, and simple, stateless mapping. By encapsulating application-layer traffic, NATSAP enables direct communication with devices behind NATs using a standardized socket notation and DNS records.


--- middle

# Introduction

The proliferation of Carrier-Grade NAT (CG NAT) in IPv4 networks has made it increasingly difficult for devices behind NATs to host services. Traditional NAT traversal techniques, such as port forwarding, STUN, TURN, and UPnP, are cumbersome, inconsistent, and difficult to automate.

NATSAP addresses this issue by introducing:

* Dynamic sub-addresses, automatically assigned by the NAT router using DSAAP.
* Encapsulation at Layer 5, allowing transparent traversal of NAT devices.
* DNS integration for seamless service discovery.
* Backward compatibility with existing network infrastructure.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

NATSAP::
    NAT Sub-Address Protocol
DSAAP::
    Dynamic Sub-Address Assignment Protocol
CG NAT::
    Carrier-Grade Network Address Translation
Sub-Addres::
    A unique 32-bit identifier assigned by the NAT router to an internal device.
NATSAP Table::
    A mapping table in the NAT router that associates sub-addresses with internal IPs.

# Protocol Overview
## Protocol Flow
1. Client Initialization (DSAAP)
    * When a device connects to the CG NAT network, it sends a DSAAP request to the gateway router.
    * The NAT router responds with a DSAAP reply, assigning a 32-bit sub-address to the client.
    * The client stores the sub-address and uses it for external communication.

2. Service Advertising (DNS)
    * The client (or DDNS service) updates the NATSAP TXT record in DNS with its current sub-address.
    * Example DNS record:
    ```
    A: example.com → 203.0.113.5    
    TXT: _natsap.example.com → "example.com, ABCD-1234"
    ```

3. Third-Party Client Connection
    * The external client resolves the public IP via the A record.
    * It looks up the _natsap TXT record for the sub-address.
    * It forms the NATSAP socket: ` natsap://203.0.113.5[ABCD-1234]:443 `

4. NATSAP Encapsulation
    * The external client encapsulates its application-layer traffic inside a NATSAP packet.
    * The CG NAT router receives the packet, extracts the sub-address, and performs a table lookup to route the traffic to the appropriate internal device.
    * The router de-encapsulates the traffic and forwards it to the internal client.
    * On the return path, the router re-encapsulates the response and sends it back to the external client.

# NATSAP Header Format
## NATSAP Header Structure
| Field Name                                                         |
|:------------------------------------------------------------------:|
| Version (8 bits)                                                   |
| Flags (8 bits)                                                     |
| Sub-Address (32 bits)                                              |
| Encapsulated Destination Port (16 bits)                            |
| Encapsulated Data Length in bytes (32 bits)                        |
| Encapsulated Application-Layer Traffic                             |

## Field Descriptions
* Version: NATSAP protocol version (e.g., 0x01).
* Flags: Reserved for future extensions.
* Sub-Address: The 32-bit sub-address assigned by the NAT router.
* Encapsulated Destination Port: The original application port.
* Encapsulated Data Length: Length of the encapsulated payload.
* Encapsulated Data: The original application-layer traffic.

# Security Considerations

1. Sub-Address Privacy:
    * Sub-addresses are public, similar to ports.
    * Applications should use existing encryption protocols (e.g., TLS) for security.
2. Rate Limiting:
    * The CG NAT router should rate-limit DSAAP requests to prevent abuse.
3. Expiration and Reuse:
    * Sub-addresses should have a lease time to prevent stale mappings.
    * Routers should implement keep-alive mechanisms to verify active clients.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
