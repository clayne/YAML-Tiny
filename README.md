# NAME

YAML::Tiny - Read/Write YAML files with as little code as possible

# PREAMBLE

The YAML specification is huge. Really, __really__ huge. It contains all the
functionality of XML, except with flexibility and choice, which makes it
easier to read, but with a formal specification that is more complex than
XML.

The original pure-Perl implementation [YAML](http://search.cpan.org/perldoc?YAML) costs just over 4 megabytes
of memory to load. Just like with Windows `.ini` files (3 meg to load) and
CSS (3.5 meg to load) the situation is just asking for a __YAML::Tiny__
module, an incomplete but correct and usable subset of the functionality,
in as little code as possible.

Like the other `::Tiny` modules, YAML::Tiny has no non-core dependencies,
does not require a compiler to install, is back-compatible to Perl 5.004,
and can be inlined into other modules if needed.

In exchange for this adding this extreme flexibility, it provides support
for only a limited subset of YAML. But the subset supported contains most
of the features for the more common uses of YAML.

# SYNOPSIS

    #############################################
    # In your file

    ---
    rootproperty: blah
    section:
      one: two
      three: four
      Foo: Bar
      empty: ~





    #############################################
    # In your program

    use YAML::Tiny;

    # Create a YAML file
    my $yaml = YAML::Tiny->new;

    # Open the config
    $yaml = YAML::Tiny->read( 'file.yml' );

    # Reading properties
    my $root = $yaml->[0]->{rootproperty};
    my $one  = $yaml->[0]->{section}->{one};
    my $Foo  = $yaml->[0]->{section}->{Foo};

    # Changing data
    $yaml->[0]->{newsection} = { this => 'that' }; # Add a section
    $yaml->[0]->{section}->{Foo} = 'Not Bar!';     # Change a value
    delete $yaml->[0]->{section};                  # Delete a value

    # Add an entire document
    $yaml->[1] = [ 'foo', 'bar', 'baz' ];

    # Save the file
    $yaml->write( 'file.conf' );

# DESCRIPTION

__YAML::Tiny__ is a perl class for reading and writing YAML-style files,
written with as little code as possible, reducing load time and memory
overhead.

Most of the time it is accepted that Perl applications use a lot
of memory and modules. The __::Tiny__ family of modules is specifically
intended to provide an ultralight and zero-dependency alternative to
many more-thorough standard modules.

This module is primarily for reading human-written files (like simple
config files) and generating very simple human-readable files. Note that
I said __human-readable__ and not __geek-readable__. The sort of files that
your average manager or secretary should be able to look at and make
sense of.

[YAML::Tiny](http://search.cpan.org/perldoc?YAML::Tiny) does not generate comments, it won't necessarily preserve the
order of your hashes, and it will normalise if reading in and writing out
again.

It only supports a very basic subset of the full YAML specification.

Usage is targeted at files like Perl's META.yml, for which a small and
easily-embeddable module is extremely attractive.

Features will only be added if they are human readable, and can be written
in a few lines of code. Please don't be offended if your request is
refused. Someone has to draw the line, and for YAML::Tiny that someone
is me.

If you need something with more power move up to [YAML](http://search.cpan.org/perldoc?YAML) (4 megabytes of
memory overhead) or [YAML::Syck](http://search.cpan.org/perldoc?YAML::Syck) (275k, but requires `libsyck` and a C
compiler).

To restate, [YAML::Tiny](http://search.cpan.org/perldoc?YAML::Tiny) does __not__ preserve your comments, whitespace,
or the order of your YAML data. But it should round-trip from Perl
structure to file and back again just fine.

# YAML TINY SPECIFICATION

This section of the documentation provides a specification for "YAML Tiny",
a subset of the YAML specification.

It is based on and described comparatively to the YAML 1.1 Working Draft
2004-12-28 specification, located at [http://yaml.org/spec/current.html](http://yaml.org/spec/current.html).

Terminology and chapter numbers are based on that specification.

## 1\. Introduction and Goals

The purpose of the YAML Tiny specification is to describe a useful subset
of the YAML specification that can be used for typical document-oriented
use cases such as configuration files and simple data structure dumps.

Many specification elements that add flexibility or extensibility are
intentionally removed, as is support for complex data structures, class
and object-orientation.

In general, the YAML Tiny language targets only those data structures
available in JSON, with the additional limitation that only simple keys
are supported.

As a result, all possible YAML Tiny documents should be able to be
transformed into an equivalent JSON document, although the reverse is
not necessarily true (but will be true in simple cases).

As a result of these simplifications the YAML Tiny specification should
be implementable in a (relatively) small amount of code in any language
that supports Perl Compatible Regular Expressions (PCRE).

## 2\. Introduction

YAML Tiny supports three data structures. These are scalars (in a variety
of forms), block-form sequences and block-form mappings. Flow-style
sequences and mappings are not supported, with some minor exceptions
detailed later.

The use of three dashes "---" to indicate the start of a new document is
supported, and multiple documents per file/stream is allowed.

Both line and inline comments are supported.

Scalars are supported via the plain style, single quote and double quote,
as well as literal-style and folded-style multi-line scalars.

The use of explicit tags is not supported.

The use of "null" type scalars is supported via the ~ character.

The use of "bool" type scalars is not supported.

However, serializer implementations should take care to explicitly escape
strings that match a "bool" keyword in the following set to prevent other
implementations that do support "bool" accidentally reading a string as a
boolean

    y|Y|yes|Yes|YES|n|N|no|No|NO
    |true|True|TRUE|false|False|FALSE
    |on|On|ON|off|Off|OFF

The use of anchors and aliases is not supported.

The use of directives is supported only for the %YAML directive.

## 3\. Processing YAML Tiny Information

__Processes__

The YAML specification dictates three-phase serialization and three-phase
deserialization.

The YAML Tiny specification does not mandate any particular methodology
or mechanism for parsing.

Any compliant parser is only required to parse a single document at a
time. The ability to support streaming documents is optional and most
likely non-typical.

Because anchors and aliases are not supported, the resulting representation
graph is thus directed but (unlike the main YAML specification) __acyclic__.

Circular references/pointers are not possible, and any YAML Tiny serializer
detecting a circular reference should error with an appropriate message.

__Presentation Stream__

YAML Tiny is notionally unicode, but support for unicode is required if the
underlying language or system being used to implement a parser does not
support Unicode. If unicode is encountered in this case an error should be
returned.

__Loading Failure Points__

YAML Tiny parsers and emitters are not expected to recover from adapt to
errors. The specific error modality of any implementation is not dictated
(return codes, exceptions, etc) but is expected to be consistent.

## 4\. Syntax

__Character Set__

YAML Tiny streams are implemented primarily using the ASCII character set,
although the use of Unicode inside strings is allowed if support by the
implementation.

Specific YAML Tiny encoded document types aiming for maximum compatibility
should restrict themselves to ASCII.

The escaping and unescaping of the 8-bit YAML escapes is required.

The escaping and unescaping of 16-bit and 32-bit YAML escapes is not
required.

__Indicator Characters__

Support for the "~" null/undefined indicator is required.

Implementations may represent this as appropriate for the underlying
language.

Support for the "-" block sequence indicator is required.

Support for the "?" mapping key indicator is __not__ required.

Support for the ":" mapping value indicator is required.

Support for the "," flow collection indicator is __not__ required.

Support for the "\[" flow sequence indicator is __not__ required, with
one exception (detailed below).

Support for the "\]" flow sequence indicator is __not__ required, with
one exception (detailed below).

Support for the "{" flow mapping indicator is __not__ required, with
one exception (detailed below).

Support for the "}" flow mapping indicator is __not__ required, with
one exception (detailed below).

Support for the "\#" comment indicator is required.

Support for the "&" anchor indicator is __not__ required.

Support for the "\*" alias indicator is __not__ required.

Support for the "!" tag indicator is __not__ required.

Support for the "|" literal block indicator is required.

Support for the ">" folded block indicator is required.

Support for the "'" single quote indicator is required.

Support for the """ double quote indicator is required.

Support for the "%" directive indicator is required, but only
for the special case of a %YAML version directive before the
"---" document header, or on the same line as the document header.

For example:

    %YAML 1.1
    ---
    - A sequence with a single element

Special Exception:

To provide the ability to support empty sequences
and mappings, support for the constructs \[\] (empty sequence) and {}
(empty mapping) are required.

For example,

    %YAML 1.1
    # A document consisting of only an empty mapping
    --- {}
    # A document consisting of only an empty sequence
    --- []
    # A document consisting of an empty mapping within a sequence
    - foo
    - {}
    - bar

__Syntax Primitives__

Other than the empty sequence and mapping cases described above, YAML Tiny
supports only the indentation-based block-style group of contexts.

All five scalar contexts are supported.

Indentation spaces work as per the YAML specification in all cases.

Comments work as per the YAML specification in all simple cases.
Support for indented multi-line comments is __not__ required.

Separation spaces work as per the YAML specification in all cases.

__YAML Tiny Character Stream__

The only directive supported by the YAML Tiny specification is the
%YAML language/version identifier. Although detected, this directive
will have no control over the parsing itself.

The parser must recognise both the YAML 1.0 and YAML 1.1+ formatting
of this directive (as well as the commented form, although no explicit
code should be needed to deal with this case, being a comment anyway)

That is, all of the following should be supported.

    --- #YAML:1.0
    - foo

    %YAML:1.0
    ---
    - foo

    % YAML 1.1
    ---
    - foo

Support for the %TAG directive is __not__ required.

Support for additional directives is __not__ required.

Support for the document boundary marker "---" is required.

Support for the document boundary market "..." is __not__ required.

If necessary, a document boundary should simply by indicated with a
"---" marker, with not preceding "..." marker.

Support for empty streams (containing no documents) is required.

Support for implicit document starts is required.

That is, the following must be equivalent.

    # Full form
    %YAML 1.1
    ---
    foo: bar

    # Implicit form
    foo: bar

__Nodes__

Support for nodes optional anchor and tag properties are __not__ required.

Support for node anchors is __not__ required.

Support for node tags is __not__ required.

Support for alias nodes is __not__ required.

Support for flow nodes is __not__ required.

Support for block nodes is required.

__Scalar Styles__

Support for all five scalar styles are required as per the YAML
specification, although support for quoted scalars spanning more
than one line is __not__ required.

Support for the chomping indicators on multi-line scalar styles
is required.

__Collection Styles__

Support for block-style sequences is required.

Support for flow-style sequences is __not__ required.

Support for block-style mappings is required.

Support for flow-style mappings is __not__ required.

Both sequences and mappings should be able to be arbitrarily
nested.

Support for plain-style mapping keys is required.

Support for quoted keys in mappings is __not__ required.

Support for "?"-indicated explicit keys is __not__ required.

Here endeth the specification.

## Additional Perl-Specific Notes

For some Perl applications, it's important to know if you really have a
number and not a string.

That is, in some contexts is important that 3 the number is distinctive
from "3" the string.

Because even Perl itself is not trivially able to understand the difference
(certainly without XS-based modules) Perl implementations of the YAML Tiny
specification are not required to retain the distinctiveness of 3 vs "3".

# METHODS

## new

The constructor `new` creates and returns an empty `YAML::Tiny` object.

## read $filename

The `read` constructor reads a YAML file from a file name,
and returns a new `YAML::Tiny` object containing the parsed content.

Returns the object on success, or `undef` on error.

When `read` fails, `YAML::Tiny` sets an error message internally
you can recover via `YAML::Tiny->errstr`. Although in __some__
cases a failed `read` will also set the operating system error
variable `$!`, not all errors do and you should not rely on using
the `$!` variable.

## read\_string $string;

The `read_string` constructor reads YAML data from a string,
and returns a new `YAML::Tiny` object containing the parsed content.

Returns the object on success, or `undef` on error.

## write $filename

The `write` method generates the file content for the properties, and
writes it to disk to the filename specified.

Returns true on success or `undef` on error.

## write\_string

Generates the file content for the object and returns it as a string.

## errstr

When an error occurs, you can retrieve the error message either from the
`$YAML::Tiny::errstr` variable, or using the `errstr()` method.

# FUNCTIONS

YAML::Tiny implements a number of functions to add compatibility with
the [YAML](http://search.cpan.org/perldoc?YAML) API. These should be a drop-in replacement, except that
YAML::Tiny will __not__ export functions by default, and so you will need
to explicitly import the functions.

## Dump

    my $string = Dump(list-of-Perl-data-structures);

Turn Perl data into YAML. This function works very much like
Data::Dumper::Dumper().

It takes a list of Perl data structures and dumps them into a serialized
form.

It returns a string containing the YAML stream.

The structures can be references or plain scalars.

## Load

    my @documents = Load(string-containing-a-YAML-stream);

Turn YAML into Perl data. This is the opposite of Dump.

Just like [Storable](http://search.cpan.org/perldoc?Storable)'s thaw() function or the eval() function in relation
to [Data::Dumper](http://search.cpan.org/perldoc?Data::Dumper).

It parses a string containing a valid YAML stream into a list of Perl data
structures.

## freeze() and thaw()

Aliases to Dump() and Load() for [Storable](http://search.cpan.org/perldoc?Storable) fans. This will also allow
YAML::Tiny to be plugged directly into modules like POE.pm, that use the
freeze/thaw API for internal serialization.

## DumpFile(filepath, list)

Writes the YAML stream to a file instead of just returning a string.

## LoadFile(filepath)

Reads the YAML stream from a file instead of a string.

# SUPPORT

Bugs should be reported via the CPAN bug tracker at

[http://rt.cpan.org/NoAuth/ReportBug.html?Queue=YAML-Tiny](http://rt.cpan.org/NoAuth/ReportBug.html?Queue=YAML-Tiny)

For other issues, or commercial enhancement or support, please contact
<a href="http://ali.as/">Adam Kennedy</a> directly.

# AUTHOR

Adam Kennedy <adamk@cpan.org>

# SEE ALSO

- [YAML](http://search.cpan.org/perldoc?YAML)
- [YAML::Syck](http://search.cpan.org/perldoc?YAML::Syck)
- [Config::Tiny](http://search.cpan.org/perldoc?Config::Tiny)
- [CSS::Tiny](http://search.cpan.org/perldoc?CSS::Tiny)
- [http://use.perl.org/use.perl.org/\_Alias/journal/29427.html](http://use.perl.org/use.perl.org/\_Alias/journal/29427.html)
- [http://ali.as/](http://ali.as/)

# COPYRIGHT

Copyright 2006 - 2012 Adam Kennedy.

This program is free software; you can redistribute
it and/or modify it under the same terms as Perl itself.

The full text of the license can be found in the
LICENSE file included with this module.
