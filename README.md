<!-- cargo-sync-readme start -->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE-MIT)
[![Apache License 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](./LICENSE-APACHE)
[![docs.rs](https://docs.rs/x509-parser/badge.svg)](https://docs.rs/x509-parser)
[![crates.io](https://img.shields.io/crates/v/x509-parser.svg)](https://crates.io/crates/x509-parser)
[![Download numbers](https://img.shields.io/crates/d/x509-parser.svg)](https://crates.io/crates/x509-parser)
[![Github CI](https://github.com/rusticata/x509-parser/workflows/Continuous%20integration/badge.svg)](https://github.com/rusticata/x509-parser/actions)
[![Minimum rustc version](https://img.shields.io/badge/rustc-1.44.0+-lightgray.svg)](#rust-version-requirements)

# X.509 Parser

A X.509 v3 ([RFC5280]) parser, implemented with the [nom](https://github.com/Geal/nom)
parser combinator framework.

It is written in pure Rust, fast, and makes extensive use of zero-copy. A lot of care is taken
to ensure security and safety of this crate, including design (recursion limit, defensive
programming), tests, and fuzzing. It also aims to be panic-free.

The code is available on [Github](https://github.com/rusticata/x509-parser)
and is part of the [Rusticata](https://github.com/rusticata) project.

Certificates are usually encoded in two main formats: PEM (usually the most common format) or
DER.  A PEM-encoded certificate is a container, storing a DER object. See the
[`pem`](https://docs.rs/x509-parser/latest/x509_parser/pem/index.html) module for more documentation.

To decode a DER-encoded certificate, the main parsing method is
[`parse_x509_certificate`](https://docs.rs/x509-parser/latest/x509_parser/fn.parse_x509_certificate.html), which builds a
[`X509Certificate`](https://docs.rs/x509-parser/latest/x509_parser/x509/struct.X509Certificate.html) object.

The returned objects for parsers follow the definitions of the RFC. This means that accessing
fields is done by accessing struct members recursively. Some helper functions are provided, for
example [X509Certificate::issuer()](https://docs.rs/x509-parser/latest/x509_parser/x509/struct.X509Certificate.html#method.issuer) returns the
same as accessing `<object>.tbs_certificate.issuer`.

For PEM-encoded certificates, use the [`pem`](https://docs.rs/x509-parser/latest/x509_parser/pem/index.html) module.

# Examples

Parsing a certificate in DER format:

```rust
use x509_parser::prelude::*;

static IGCA_DER: &[u8] = include_bytes!("../assets/IGC_A.der");

let res = parse_x509_certificate(IGCA_DER);
match res {
    Ok((rem, cert)) => {
        assert!(rem.is_empty());
        //
        assert_eq!(cert.tbs_certificate.version, X509Version::V3);
    },
    _ => panic!("x509 parsing failed: {:?}", res),
}
```

To parse a CRL and print information about revoked certificates:

```rust
#
#
let res = parse_x509_crl(DER);
match res {
    Ok((_rem, crl)) => {
        for revoked in crl.iter_revoked_certificates() {
            println!("Revoked certificate serial: {}", revoked.raw_serial_as_string());
            println!("  Reason: {}", revoked.reason_code().unwrap_or_default().1);
        }
    },
    _ => panic!("CRL parsing failed: {:?}", res),
}
```

See also `examples/print-cert.rs`.

# Features

- The `verify` feature adds support for (cryptographic) signature verification, based on `ring`.
  It adds the
  [X509Certificate::verify_signature()](https://docs.rs/x509-parser/latest/x509_parser/x509/struct.X509Certificate.html#method.verify_signature)
  to `X509Certificate`.

```rust
/// Cryptographic signature verification: returns true if certificate was signed by issuer
#[cfg(feature = "verify")]
pub fn check_signature(cert: &X509Certificate<'_>, issuer: &X509Certificate<'_>) -> bool {
    let issuer_public_key = &issuer.tbs_certificate.subject_pki;
    cert
        .verify_signature(Some(issuer_public_key))
        .is_ok()
}
```

## Rust version requirements

The 5.0 series of `der-parser` requires **Rustc version 1.44 or greater**, based on nom 6
dependencies.

[RFC5280]: https://tools.ietf.org/html/rfc5280
<!-- cargo-sync-readme end -->

## Changes

See [CHANGELOG.md](CHANGELOG.md)

# License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
