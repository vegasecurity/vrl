---
name: parse
description: Used to create a VRL script that a log entry or file.
---

# VRL Features

The VRL script you are to provide has to be supported by a VRL interpreter compiled using only the following features:

- compiler
- diagnostic
- stdlib
- value

all of the other default features are not included.

# Goal

The output of the VRL script should be JSON, containing a supported timestamp in a top-level field.
If the input entries are valid JSONs with multiple fields, only ensure the timestamp.
In the case where the input is a single field containing all of the data, extract the fields.

In the case where each line is not a valid JSON (new line defined JSON), assume you receive a JSON with a single field, `plain_text`, containing the entries as plain text. in this case, you should create a script that returns a list of valid JSONs, using something like `. = compact(entries)` to create the entries and deleting the initial `plain_text` field.

When extracting a text field into separate fields, try to not split or extract based on hard-coded field names, but rather use generic parsing when possible.
Whenever there is a text blob containing info, try to identify the format of it (JSON, XML, CEF) and normalize it to JSON. Unescape encodings such as unicode.
In the process of extracting a text field to separate fields, attempt to keep the original structure of the data and not completely flatten it.

# Guidelines

Try to make the VRL script as efficient as possible:

- Arguments to functions in VRL are passed on by-value as opposed to by-reference - making it very inefficient.
  - eg. using `push` inside of a `for_each` clones the entire array each time you reference it when calling `push` - `**NEVER** do this
  - use `map_values` or `filter` as alternatives
- When possible, use built-in functions to parse common log formats such as:
  - `parse_syslog`
  - `parse_cef`
  - `parse_glog`
  - `parse_aws_alb_log`
  - `parse_linux_authorization`
  - `parse_csv`
    search the code base for the matching function according to the log format.

# Timestamp Parsing

Each log entry has to have a timestamp in one of the following formats:

- iso8601
- rfc2822
- rfc3339
- unix_timestamp
- strptime (parse dates using the Unix strptime format with some variations)
  - format specifiers: %C, %d, %D, %e, %F, %g, %G, %h, %H, %I, %j, %k, %l, %m, %M, %n, %R, %S, %t, %T, %u, %U, %V, %w, %W, %y, %Y, %%.
  - %f for milliseconds precision support.
  - %z timezone offsets can be specified as (+|-)hhmm or (+|-)hh:mm.
  - The timezone name format specifier (%Z) is not supported currently.

When creating a VRL script, identify the primary timestamp of the log entries and its format.
If the primary timestamp is already in a supported format, specify the name of the timestamp field and the format in your output and do not change or format it in any way.
In the case where the timestamp is not in a supported format, make the script add a field containing the timestamp in a supported format. The new field name should be `timestamp` when such a field doesn't already exist.
make sure to mention the format of the timestamp in the timestamp field. if the timestamp if strptime format, also mention the format string.
The timestamp always has to be at the root level. For example, when the timestamp is `auditLog.timestamp` do `.timestamp = .auditLog.timestamp`
