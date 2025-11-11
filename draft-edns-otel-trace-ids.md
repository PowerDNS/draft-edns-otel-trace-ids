---
title: "Communicating Distributed Trace IDs in EDNS"
abbrev: "EDNS TRACEPARENT"
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
  W3C.trace-context:
    target: https://www.w3.org/TR/2021/REC-trace-context-1-20211123/
    display: 'W3C Recommendation: Trace Context'

informative:

...

--- abstract

This document defines a new EDNS Option named TRACEPARENT that is used to communicate an identifier for correlating events between DNS systems.

--- middle

# Introduction

In distributed systems or otherwise interacting systems, operators might want to correlate events or know how an incoming request moves through the system.
To achieve this correlation, a tracing identifier is generated on the front end system that receives the initial request and passed to downstream systems.
These downstream systems will generate data related to the request that can be collected and used in system health measurements or trouble shooting.

This document defines a new EDNS{{!RFC6891}} option (TRACEPARENT) to pass tracing identifiers between DNS servers. It follows the W3C recommendation for Trace Context and the traceparent HTTP header, version 00{{!W3C.trace-context}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

* This document uses DNS Terminology as defined in {{!RFC9499}}.

* Base16 is the representation of arbitrary binary data by an even number of case-insensitive hexadecimal digits ({{!RFC4648, Section 8}}).

# Wire Format

The TRACEPARENT option has the following wire format:

~~~ ascii-art
       0                   1
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
      +---------------+---------------+
 0:   |        OPTION-CODE (TBD1)     |
      +---------------+---------------+
 2:   |         OPTION-LENGTH         |
      +---------------+---------------+
 4:   |    VERSION    |    RESERVED   |
      +---------------+---------------+
 6:   |       TRACEPARENT DATA        /
      +---------------+---------------+
~~~

The VERSION field defined in this specification MUST be set to 0.

The TRACEPARENT DATA field for version 0 contains 3 fields: a 16 byte trace-id, an 8 byte parent-id, a 1 byte trace-flags field.

~~~ ascii-art
       0                   1
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
      +---------------+---------------+
 0:   |           TRACE-ID            /
      /                               |
      +---------------+---------------+
 16:  |           PARENT-ID           /
      /                               |
      +---------------+---------------+
 24:  |  TRACEFLAGS   |
      +---------------+
~~~

# Presentation Format

Even though EDNS options will never appear in DNS zone files, its value could appear in logging or analysis of packet captures.
The presentation format for TRACEPARENT follows the traceparent HTTP header from Section 3.2 of {{!W3C.trace-context}}:

~~~ ascii-art
TRACEPARENT=[version]-[trace-id]-[parent-id]-[trace-flags]
~~~

Where each field is represented as Base16 with the hexadecimal characters in lowercase.

# Processing of TRACEPARENT

TRACEPARENT SHOULD only be used after mutual agreement between the upstream and downstream server operators.
A nameserver MAY include a TRACEPARENT option in outgoing queries to trigger tracing in downstream servers.
This model follows the recommendations of Section 4 of {{!W3C.trace-context}}.

Performing tracing SHOULD NOT impact DNS query processing.
Hence, nameservers receiving a malformed TRACEPARENT option SHOULD ignore this option and continue processing the query.
It is RECOMMENDED to inform the operator of the nameserver, for example using logging, about malformed TRACEPARENT options.

Tracing information is collected outside of the DNS transaction and is independent of the DNS query processing.
The inclusion of a TRACEPARENT option in a query must be seen as a signal from the requestor that tracing should be performed.

This signal can come in two forms; with or without Trace ID Data.
A query with Trace ID data signals "perform tracing and use this Trace ID".
A query with the EDNS option without Trace Parent data signals "perform tracing and inform me of the Trace ID".

## Requests with Trace ID Data

Queries that contain the TRACEPARENT option with Trace ID Data, should perform data collection as configured by the operator.
As the Trace Parent is known to the requester and receiver, responders MUST NOT include a TRACEPARENT option in responses to queries that contained a TRACEPARENT option.

## Requests without Trace ID Data

Queries that have the TRACEPARENT option without Trace Parent Data, should generate these values themselves and perform data collection as configured by the operator.
The responder SHOULD include a TRACEPARENT option in the response containing this information.

## Access Control

It is RECOMMENDED to use access control on who can send TRACEPARENT to initiate data collection, e.g. using IP address allow-lists, TSIG{{!RFC8945}}, or other methods.

When a nameserver receives the TRACEPARENT EDNS option from a system that is allowed to initiate tracing, it should perform any operations required to collect tracing information, as configured by the operator.

When a nameserver receives the TRACEPARENT EDNS option from a system that is not allowed to initiate tracing, it MUST ignore the option and process the query as if no TRACEPARENT option was present.

# Security Considerations

TODO Security

* ACL
* Mutual agreement

# IANA Considerations

None.

--- back

# Appendix A. Presentation Format Examples
{:numbered="false"}

An OpenTelemetry Trace ID of 1234567890ABCDEF1234567890ABCDEF, Parent ID of FEDCBA0987654321, and no Trace Flags is presented as:

~~~ ascii-art
TRACEPARENT=00-1234567890abcdef1234567890abcdef-fedcba0987654321-00
~~~

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

Job Snijders, Wouter de Vries,
