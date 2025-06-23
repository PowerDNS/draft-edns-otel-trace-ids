---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Communicating OpenTelemetry Trace IDs in EDNS"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-edns-otel-trace-ids-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
area: ops
workgroup: dnsop
keyword:
 - EDNS
 - OpenTelemetry
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

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

```
                   +0 (MSB)                            +1 (LSB)
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   0: |                       OPTION-CODE (TBD1)                      |
      +---+---+---+---+---+---+---+---|---+---+---+---+---+---+---+---+
   2: |                       OPTION-LENGTH (16 or 24)                |
      +---+---+---+---+---+---+---+---|---+---+---+---+---+---+---+---+
   4: |                           TRACE ID                            /
      |                                                               /
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
  20: |                       SPAN ID (optional)                      /
      |                                                               /
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
```

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
