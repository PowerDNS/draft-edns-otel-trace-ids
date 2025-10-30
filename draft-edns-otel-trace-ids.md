---
title: "Communicating Distributed Trace IDs in EDNS"
abbrev: "EDNS TRACEIDS"
category: info

docname: draft-edns-otel-trace-ids-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
# area: ops
# workgroup: dnsop
keyword:
 - EDNS
 - OpenTelemetry
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "PowerDNS/draft-edns-otel-trace-ids"
  latest: "https://PowerDNS.github.io/draft-edns-otel-trace-ids/draft-edns-otel-trace-ids.html"

author:
 -
    fullname: Otto Moerbeek
    organization: PowerDNS.com B.V.
    email: otto.moerbeek@powerdns.com
 -
    fullname: Peter van Dijk
    organization: PowerDNS.com B.V.
    email: peter.van.dijk@powerdns.com
 -
    fullname: Pieter Lexis
    organization: PowerDNS.com B.V.
    email: pieter.lexis@powerdns.com

normative:

informative:
  OT.WEBSITE:
    target: https://opentelemetry.io/
    title: OpenTelemetry Website
  CNCF.WEBSITE:
    target: https://www.cncf.io/
    title: Cloud Native Computing Foundation Website

...

--- abstract

This document defines a new EDNS Option named TRACEID that is used to communicate an identifier for correlating events between DNS systems.

--- middle

# Introduction

In distributed systems or otherwise interacting systems, operators might want to correlate events or know how an incoming request moves through the system.
To achieve this correlation, a tracing identifier is generated on the front end system that receives the initial request and passed to downstream systems.
These downstream systems will generate data related to the request that can be collected and used in system health measurements or trouble shooting.

This document defines a new EDNS{{!RFC6891}} option (TRACEID) to pass tracing identifiers between DNS servers.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

* This document uses DNS Terminology as defined in {{!RFC9499}}.

* Base16 is the representation of arbitrary binary data by an even number of case-insensitive hexadecimal digits ({{!RFC4648, Section 8}}).

# Wire Format

The TRACEID option has the following wire format:

~~~ ascii-art
     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +---------------+---------------+
 0: |        OPTION-CODE (TBD1)     |
    +---------------+---------------+
 2: |         OPTION-LENGTH         |
    +---------------+---------------+
 4: |  VERSION (0)  | TRACE ID TYPE |
    +---------------+---------------+
 6: |         TRACE ID DATA         /
    /                               /
    +---------------+---------------+

~~~

Version (1 octet) has the value 0 for this specification.
Trace ID Type (1 octet) describes how the Trace ID data should be interpreted.
Trace ID Data has a variable length, depending on the Trace ID Type.

# Presentation Format

Even though EDNS options will never appear in DNS zone files, its value could appear in logging or analysis of packet captures.
The presentation format for TRACEID is as follows:

~~~ ascii-art
TRACEID=[mnemonic]:[data]
~~~

Where the `mnemonic` is the mnemonic defined for the ID Type and `data` is the ID Data represented as Base16.
When an unknown Type is encountered, the `mnemonic` MUST be presented as `TYPEN`, where `N` is the decimal representation of the Trace ID Type without leading zeroes.

# Processing of TRACEID

TRACEID SHOULD only be used after mutual agreement between the upstream and downstream server operators.

Performing tracing SHOULD NOT impact DNS query processing.
Hence, nameservers receiving a malformed TRACEID option or a TRACEID option with an unknown or unsupported Type ID SHOULD ignore this option and continue processing the query.
It is RECOMMENDED to inform the operator of the nameserver, for example using logging, about malformed or unknown TRACEID options.

Tracing information is collected outside of the DNS transaction and is independent of the DNS query processing.
The inclusion of a TRACEID option in a query must be seen as a signal from the requestor that tracing should be performed.

This signal can come in two forms; with or without Trace ID Data.
A query with Trace ID data signals "perform tracing and use this Trace ID".
A query without Trace ID data signals "perform tracing and inform me of the Trace ID".

## Requests with Trace ID Data

Queries that contain the TRACEID option with Trace ID Data, should perform data collection as configured by the operator.
As the Trace ID is known, responders MUST NOT include a TRACEID option in responses to queries that contained a TRACEID option.

## Requests without Trace ID Data

Queries that have the TRACEID option without Trace ID Data, should generate a Trace ID and perform data collection as configured by the operator.
The responder SHOULD include a TRACEID option with Trace ID Data in the response.

## Access Control

It is RECOMMENDED to use access control on who can send TRACEID to initiate data collection, e.g. using IP address allow-lists, TSIG{{!RFC8945}}, or other methods.

When a nameserver receives the TRACEID EDNS option from a system that is allowed to initiate tracing, it should perform any operations required to collect tracing information, as configured by the operator.
The nameserver MAY include a TRACEID option in outgoing queries to trigger tracing in downstream servers.

When a nameserver receives the TRACEID EDNS option from a system that is not allowed to initiate tracing, it MUST ignore the option and process the query as if no TRACEID option was present.

# Trace ID Types

This specification defines several values for Trace ID Type.


| Type name     | Mnemonic      | Trace ID Type |
| ------------- | ------------- | ------------- |
| OpenTelemetry | OT            | 0             |
| Private use   | PRIVATENNN (where NNN is the decimal representation of the Type) | 247 - 254     |
| RESERVED      | RESERVED           | 255           |


## OpenTelemetry (0)

OpenTelemetry{{OT.WEBSITE}} is an open standard for telemetry data like metrics, logs and traces.
It is maintained by the Cloud Native Computing Foundation (CNCF){{CNCF.WEBSITE}}.

For OpenTelemetry traces that contain Trace ID Data, the TraceID Data field MUST contain a 16 octet Trace ID and MAY have an 8 octet Span ID following it.
This makes the Trace ID data field either 16 or 24 octets long.

For responses that need a TRACEID option, the TraceID Data field MUST contain a 16 octet Trace ID and MAY have an 8 octet Span ID following it.

## Private use (247 - 254)

These Trace ID Types can be used for private tracing identifiers.

## RESERVED (255)

This Trace ID Type value is reserved for potential future expansion and MUST NOT be used.


# Security Considerations

TODO Security

* ACL
* Mutual agreement

# IANA Considerations

TODO request IANA to create a Trace ID Type registry.

--- back

# Appendix A. Presentation Format Examples
{:numbered="false"}

An OpenTelemetry TraceID of 1234567890ABCDEF1234567890ABCDEF is presented as:

~~~ ascii-art
TRACEID=OT:1234567890ABCDEF1234567890ABCDEF
~~~

A private Type would be represented as:

~~~ ascii-art
TRACEID=PRIVATE250:ABCDEF1234
~~~

An unknown Type would be presented as:

~~~ ascii-art
TRACEID=TYPE19:FE1234
~~~

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

Job Snijders, Wouter de Vries,
