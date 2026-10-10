---
title: "Common Format and Media Type for Control-Character-Separated Values (CCSV) Files"
abbrev: "CCSV File Format"
category: info

docname: draft-rankin-ccsv-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: Applications and Real-Time
# workgroup: Independent Submission
keyword:
 - CCSV
 - media type
 - tabular data
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "oldgrognard/ccsv-draft"
  latest: "https://oldgrognard.github.io/ccsv-draft/draft-rankin-ccsv.html"

author:
 -
    fullname: "Mike Rankin"
    email: "mrankin@reactor.zone"

normative:

informative:

...

--- abstract

This document specifies the Control-Character-Separated Values
(CCSV) file format, a UTF-8 encoded tabular data format that uses
ASCII control characters as field and record separators. CCSV is
intended for exchanging structured, rectangular data between
applications such as databases, spreadsheets, statistical software,
and data conversion utilities. This document also specifies the
``text/csv`` media type for CCSV files.


--- middle

# Introduction {#intro}

A CCSV (Control-Character-Separated Values) file is a file format
for moving rectangular data between applications such as
spreadsheets, statistical analysis programs, databases, and
conversion utilities.

CCSV is similar in purpose to Comma-Separated Values (CSV) files
{{!RFC4180}}, Tab-Separated Values (TSV) files, and their
derivatives. CCSV differs primarily in its use of non-printable
ASCII control characters as field and record separators.

Using control characters as separators reduces ambiguity when
field values contain characters commonly used as delimiters in other
formats.  In particular, CCSV does not require quoting or escaping
ordinary printable characters solely because those characters have
special meaning in other delimited-value formats.

A CCSV file consists of a required header followed by zero or more
records. The header and every record contain the same number of
fields. The first row therefore defines the names of the columns,
while subsequent rows contain the corresponding values.

This document defines the CCSV file format and registers the
``text/ccsv`` media type.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

The following terms are used throughout this document:

- **Field:** A single value withing a CCSV record.
- **Record:** A sequence of fields representing one row of data.
- **Header:** The first record in a CCSV file. The fields in the header identify the columns of the file.
- **Unit Separator (US):** The Unicode character U+001F, used to separate fields.
- **Record Separator (RS):** The Unicode character U+001E, used to separate records.

# CCSV Format {#format}

## File Structure {#structure}

A CCSV file MUST begin with a header.

The header and eqch subsequent record MUST contain the ame number of fields

A Unit Separator (US, U+001F) separates adjacent fields within a record.

A Record Separator (RS, U+001E) separates adjacent records, including the header from the first data record.

A CCSV file MUST therefore contain an RS following the header. An RS MAY also follow the final
record. The presence or absence of an RS following the final record does
not change the contents of the file.

The end of the encoded CCSV data constitutes the end of the file.
No additional end-of-file character or sequence is required.

## Formatting Requirements {#requirements}

A conforming CCSV file MUST satisfy all of the following requirements:

1. The file MUST be encoded in UTF-8 {{!RFC3629}}.
2. The file MUST NOT begin with a UTF-8 byte order mark (BOM, U+FEFF).
3. The file MUST begin with a header. The header consists of one or more fields separated by US (U+001F).
4. US (U+001F) MUST occur between adjacent fields in a record.
5. RS (U+001E) MUST occur between adjacent records, including between the header and the first data record.
6. An RS MAY occur immediately after the final record. A producer SHOULD NOT add an RS after the final record unless doing so is useful for the producer or its surrounding processing system.
7. The header and every record MUST contain the same number of fields.
8. Empty fields are represented by consecutive delimiters or by a delimiter adjacent to the beginning or end of a record, as applicable.
9. US (U+001F) and RS (U+001E) MUST NOT occur within the contents of a field.
10. Carriage return (CR, U+000D) and line feed (LF, U+000A) are valid field characters and MUST NOT be interprested as record separators.

Except for US and RS, a field MAY contain any Unicode scalar value representable in UTF-8 including other ASCII control characters.

## Syntax {#syntax}

The following ABNF {{!RFC5234}} describes the CCSV structure
at the UTF-8 octet level. The UTF-8 productions are
taken from {{!RFC3629}}.


    file       = header RS *(record RS) [record]
    header     = field *(US field)
    record     = field *(US field)
    field      = *field-char
    field-char = %x00-1D / %x20-7F / UTF8-2 / UTF8-3 / UTF8-4
    RS         = %x1E ; Record Separator, U+001E
    US         = %x1F ; Unit Separator, U+001F
    UTF8-2     = %xC2-DF UTF8-tail
    UTF8-3     = %xE0 %xA0-BF UTF8-tail /
                 %xE1-EC 2( UTF8-tail ) /
                 %xED 3( UTF8-tail ) /
                 %xEE-EF 2( UTF8-tail )
    UTF8-4     = %xF0 %x90-BF 2( UTF8-tail ) /
                 %xF1-F3 3( UTF8-tail ) /
                 %xF4 %x80-8F 2( UTF8-tail )
    UTF8-tail  = %x80-BF

The UTF-8 grammar above is provided for implementation convenience. The authoritative definition of UTF-8 is {{!RFC3629}}.

A field MAY be empty. Consequently, the following are valid field arrangements when the surrounding
record contains the appropriate number of fields.

    field1<US>field2
    <US>field2
    field1<US>
    <US>

where ``<US>`` represents U+001F.

# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
