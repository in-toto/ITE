# ITE-6: Generalized link format

<table>
<tr><td>ITE<td>6
<tr><td>Title<td>Generalized link format
<tr><td>Sponsor<td><a href="https://github.com/santiagotorres">Santiago Torres-Arias</a>
<tr><td>Status<td>Draft 💬
<tr><td>Type<td>Standards
<tr><td>Created<td>2020-10-30
</table>

## Abstract

This [in-toto enhancement (ITE)](https://github.com/in-toto/ITE) defines a new
schema for in-toto link files, which are now generally called "attestations." An
attestation is authenticated metadata about one or more software artifacts.
Instead of a fixed schema (`materials`, `products`, `command`, etc.), the
attestation format allows users to define their own custom "predicates" with
arbitrary schemas. This makes it easier and more natural to express any type of
metadata about software. Several pre-defined predicate types cover the most
common cases, including
[provenance](https://github.com/in-toto/attestation/blob/main/spec/predicates/provenance.md)
for describing the origins of a software artifact,
[spdx](https://github.com/in-toto/attestation/blob/main/spec/predicates/spdx.md)
for supporting [SPDX](https://spdx.dev), and
[link](https://github.com/in-toto/attestation/blob/main/spec/predicates/link.md)
as a drop-in replacement for the existing link schema.

A future ITE will update the layout format to take advantage of this extra
information.

More broadly, this ITE moves in-toto to follow the [SLSA Attestation Model
](https://github.com/slsa-framework/slsa/blob/main/controls/attestations.md) and
is developed jointly with
[Binary Authorization](https://cloud.google.com/binary-authorization). The goal
is to have an industry standard artifact metadata format that can be consumed by
any system.

## Specification

In-toto will support the new attestation format defined in
<https://github.com/in-toto/attestation>. This defines the schema of the signed
payload, which is `signed` in the current envelope and `payload` in
[ITE-5](../5/README.adoc).

The `_type` field identifies the type of the object, which is unchanged from the
status quo. Old-style links use the value `link`. New-style attestations use the
value `https://in-toto.io/Statement/v0.1`, where `0.1` identifies the major
version of the schema. Implementations can support multiple versions
simultaneously by inspecting the `_type`.

Initially, only
["link"-type](https://github.com/in-toto/attestation/blob/main/spec/predicates/link.md)
predicates will be supported because the layout language refers to
predicate-specific fields, namely `name`, `command`, `products`, and
`materials`. The filename remains the same: `[name].[KEYID-PREFIX].link`.

A future ITE will extend the layout language to support arbitrary
predicates. That ITE may also update the filename suffix since not all
attestations are links.

### Example

The following new-style attestation:

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "subject": [{"name": "out.bin", "digest": {"sha256": "fedc9876..."}}],
  "predicateType": "https://in-toto.io/Link/v0.2",
  "predicate": {
    "name": "build",
    "command": "make",
    "materials": {"in.txt": {"sha256": "abcd1234..."}},
    "byproducts": {},
    "environment": {}
  }
}
```

is equivalent to the following old-style link:

```json
{
  "_type": "link",
  "name": "build",
  "command": "make",
  "materials": {"in.txt": {"sha256": "abcd1234..."}},
  "products": {"out.bin": {"sha256": "fedc9876..."}},
  "byproducts": {},
  "environment": {}
}
```

## Motivation

The two main goals of this ITE are to:

*   Make it easier to author and consume arbitrary software artifact metadata.
    The existing link schema is rigidly focused on running a command that
    converts materials into products. Several types of metadata do not cleanly
    fit this model, such as:

    *   Provenance of a CI/CD workflow, where there is no particular "command"
        to run.
    *   Code review status.
    *   Vulnerability scan result showing known vulnerabilities.
    *   Policy decision allowing artifact to be used in particular environment.

*   Build a larger ecosystem around software artifact metadata that is not
    specific to in-toto (or
    [Binary Authorization](https://cloud.google.com/binary-authorization) or any
    other consumer). By supporting a more generic, extensible attestation
    format, we allow CI/CD pipelines, vulnerability scanners, and other systems
    to generate a single set of attestations that can be consumed by anyone.
    Furthermore, this paves the way for in-toto to support existing formats such
    as SPDX or CycloneDX.

Furthermore, this ITE fits within the
[SLSA Framework](https://github.com/slsa-framework/slsa). The provenance format
in particular is the official SLSA recommendation.

## Reasoning

See <https://github.com/in-toto/attestation#reasoning>. That spec was designed
specifically for this ITE.

## Backwards Compatibility

New-style link-type attestations will be supported as a drop-in replacement for
old-style links.

Old-style links will continue to be supported.

## Security

This does not affect security because the link-type attestation is isomorphic to
the existing link schema and can be translated freely in both directions.

## Testing

## Infrastructure Requirements

## References
