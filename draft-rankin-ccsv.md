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
    field-char = %x00-1D /
                 %x20-7F /
                 UTF8-2 /
                 UTF8-3 / UTF8-4

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

    field1&lt;US&gt;field2
    lt;USgt;field2
    field1lt;USgt;
    lt;USgt;

where ``lt;USgt;`` represents U+001F.

# Encoding Considerations {#encoding}

CCSV files MUST be encoded using UTF-8 {{!RFC3629}}.

The UTF-8 encoding applies to the entire CCSV file, including the header and all field contents.

A CCSV file MUST NOT begin with a UTF-8 byte order mark (BOM, U+FEFF).

UTF-8 permits CCSV to represent Unicode text while retaining the single-octet representation
of the ASCII control characters used as delimiters.

CR and LF are data characters in CCSV and are not record delimiters.
Applications transporting CCSV content MUST preserve CR and LF occurring within fields as data.

When CCSV is transported using a protocol or representation that
imposes additional requirements on textual content, those
requirements apply to the transport representation. Such processing
MUST NOT alter the CCSV data represented by the payload.

# Internationalization Considerations {#i18n}

CCSV uses UTF-8 {{!RFC3629}} and therefore supports
Unicode text in field and header values.

CCSV does not define a language, locale, collation, numeric format,
date format, or other interpretation of field contents. Applications
processing CCSV data are responsible for applying appropriate
locale-specific or application-specific interpretation to field
values.

Unicode strings are intended to be compared as identifiers
or used in security-sensitive operations may require Unicode
normalization or other application-specific processing. CCSV does
not prescribe a normalization form.


# Security Considerations {#security}

CCSV is a passive data format and does not itself define executable
content. However, CCSV data is commonly imported into applications
such as spreadsheets, databases, statistical software, and scripts.
Those applications may assign additional semantics to particular
field values.

In particular, applications that import CCSV data into spreadsheet
software or other systems capable of evaluating expressions
MUST NOT assume that field contents are trustworthy. An application
may interpret a field beginning with a particular
character as a formula, expression, command, or other executable
content. Such applications SHOULD treat CCSV field
contents as untrusted input and apply appropriate validation or
escaping before passing the data to an interpreter.

Implementations parsing CCSV data SHOULD also
protect against resource-exhaustion attacks, including excessively
large fields, records, or files.

CCSV provides no confidentiality, integrity, authentication, or
access-control mechanisms. Applications requiring those properties
MUST provide them through mechanisms outside the CCSV
format.

Implementations MUST reject malformed UTF-8 rather
than silently interpreting invalid UTF-8 sequences as valid CCSV
data.

# Interoperability Considerations {#interop}

Interoperability depends on sonsistent implementation of the CCSV
formatting and encoding requirements

In particular, implementations MUST agree on all of the following:

- UTF-8 encoding;
- absence of a leading UTF-8 BOM;
- US (U+001F) as the field separator;
- RS (U+001E) as the record separator;
- equal field counts in the header and every record;
- representation of empty fields;
- prohibition of US and RS within field contents; and
- treatment of the CR and LF as field data rather than record separators.

Implementations MUST NOT treat printable delimiter characters as
having special CCSV semantics. CCSV does not require quoting or escaping of printable
characters merely because they have special meaning in
other delimited-value formats.

Implementations that consume CCSV SHOULD validate
the file structure of the file before processing its contents.


# IANA Considerations {#iana}

IANA is requested to register the following media type in the
Standards Tree:

Type name
: text

Subtype name
: ccsv

Required parameters
: charset. The charset parameter MUST have the value "UTF-8".

Optional parameters
: None.

Encoding considerations
: 8bit. CCSV content is encoded using UTF-8 as specified in {{encoding}}.
  CR and LF occuring within fields are data characters and are not CCSV
  record separators.

Security considerations
: See {{security}}.

Interoperability considerations
: See {{interop}}.

Published specification
: This document.

Applications that use this media type
:  Databases, spreadsheets, statistical programs, data
  conversion utilities, and other applications that process
  rectangular tabular data.

Fragment identifier considerations
:  CCSV does not define a fragment identifier syntax.

Additional information
:

  Deprecated alias names for this type
  : None.

  Magic number(s)
  : None.

  File name extension(s)
  : .ccsv

  Macintosh file type code(s)
  : None.

Person & email address to contact for further information
: Mike rankin <mrankin@oldgrognard.pub>

Intended usage
: COMMON

Restrictions on usage
: None.

Author
: Mike Rankin <mrankin@oldgrognard.pub>

Change controller
: IETF

# Examples {#examples}

In the examples below &lt;US&gt; represents U+001F and
&lt;RS&gt; represents U+001E. The notation is used only to
make the control characters visible and not part of an
actual CCSV file

## Basic Example {#basic-example}

The following CCSV data contains three columns and two records:

```
Name&lt;US&gt;Age&lt;US&gt;City&lt;RS&gt;
Alice<US>42<US>Boston&lt;RS&gt;
Bob<US>37<US>Chicago
```

## Empty Fields {empty-fields}

The following record contains an empty second field:

```
Alice<US><US>Boston
```

The consecutive US characters represent an empty field.

The following record begins with an empty field.

```
&lt;US&gt;42&lt;US&gt;Boston
```

## Optional Final Record Separator

Both of the following representations contain the same three
fields and ar valid CCSV representations of a final record:

```
Alice&lt;US&gt;42&lt;US&gt;Boston
```

and:

```
Alice&lt;US&gt;42&lt;US&gt;Boston&lt;RS&rs;
```

A producer SHOULD generally omit the final RS
unless its processing environment benefits from a terminating
record separator.

--- back

# Acknowledgments
{:numbered="false"}

The author thanks those individuals who provided feedbanc and
technical discissions concerning the CCSV format and its proposed
media type
