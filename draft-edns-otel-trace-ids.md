---
title: "Communicating OpenTelemetry Trace IDs in EDNS"
abbrev: "EDNS OTTRACEIDS"
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

normative:

informative:

...

--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Wire Format

~~~ ascii-art
                   +0 (MSB)                            +1 (LSB)
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   0: |                       OPTION-CODE (TBD1)                      |
      +---+---+---+---+---+---+---+---|---+---+---+---+---+---+---+---+
   2: |                       OPTION-LENGTH (16 or 24)                |
      +---+---+---+---+---+---+---+---|---+---+---+---+---+---+---+---+
   4: |     VERSION (1 octet)         |
      +---+---+---+---+---+---+---+---|---+---+---+---+---+---+---+---+
   5: |                           TRACE ID                            /
      |                                                               /
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
  21: |                       SPAN ID (optional)                      /
      |                                                               /
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
~~~

Version is 0 for this specification.
The Trace ID is 16 octets long and is mandatory (or not? if empty, receiver selects one, so that empty is just a "please trace" signal. Although then the sender could also just generate a trace ID).
The optional Span ID is 8 octets long.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
