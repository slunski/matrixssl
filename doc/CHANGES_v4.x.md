# MatrixSSL 4.x changelog

## Changes between 4.0.2 and 4.1.0 [April 2019]

- TLS:

    * (RoT Edition only): Added support for Inside Secure VaultIP
      (Root-of-Trust) crypto provider.

    * Improved the separation of private and public TLS header files
      for better private-public separation. The public headers now of
      the form matrixsslApi\*.h, while private headers are of the form
      matrixssllib\_*.h.

    * Added client-side support for X25519 in TLS 1.2.

    * Added client-side support for RSASSA-PSS signatures in TLS 1.2.

    * Added support for RSASSA-PSS key/cert pairs.

    * Fix vulnerabilities reported by Robert Święcki (discovered using
      Hongfuzzer): a server-side heap buffer read overflow when
      parsing maliciously crafted ClientHello extensions and a
      segfault in TLS 1.2 GCM decryption of maliciously crafted
      records with small ciphertext.

    * Added the simpleClient.c and simpleServer.c example
      applications. These are intended as minimalistic examples of how
      to use the top-level TLS API.

    * Fixed bugs in matrixSslSessOptsServerTlsVersionRange and
      matrixSslSessOptsClientTlsVersionRange.

    * Fixed bug that caused non-insitu app data encryption to fail in
      tls13EncodeAppData when using the matrixSslEncodeToOutdata API
      instead of the more standard matrixSslGetWriteBuf +
      matrixSslEncodeWritebuf pattern.

    * Added new minimal example configurations: tls12-minimal,
      tls12-minimal-client-ecc, tls13-minimal,
      tls13-minimal-client-ecc

    * When performing TLS 1.2 renegotiation, re-send the original
      ClientHello cipher list.

    * Added the USE_LENIENT_TLS_RECORD_VERSION_MATCHING compatibility
      option.

## Changes between 4.0.1 and 4.0.2 [February 2019]

This version fixes a critical vulnerability in RSA signature
verification. A maliciously crafted certificate can be used to trigger
a stack buffer overflow, allowing potential remote code execution
attacks. The vulnerability only affects version 4.0.1 and the standard
Matrix Crypto provider. Other providers, such as the FIPS crypto
provider, are not affected by the bug. Thanks to Tavis Ormandy for
reporting this.

## Changes between 4.0.0 and 4.0.1 [November 2018]

This version improves the security of RSA PKCS #1.5 signature
verification and adds better support for run-time security
configuration.

- TLS:

    * Added a run-time security callback feature
      (matrixSslRegisterSecurityCallback). The security callback can
      allow or deny a cryptographic operation based on the operation
      type and the key size. Currently only authentication and key
      exchange operations are supported. The default security callback
      supports pre-defined security profiles
      (matrixSslSetSecurityProfile).

    * Added an example security profile: WPA3 1.0 Enterprise 192-bit
      mode restrictions for EAP-TLS.

    * Added support for the TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
      ciphersuite.

    * Changed the way how protocol version IDs are stored internally
      and rewrote most of the version negotiation code. This is almost
      entirely an internal code refactoring. To the API user, the only
      visible change is that version selection APIs now take in an
      argument of type psProtocolVersion_t instead of int32_t. See the
      API reference guide for details.

    * Refactored ServerKeyExchange signature generation and
      verification code.

- Crypto:

    * Changed from a parsing-based to a comparison-based approach in
      DigestInfo validation when verifying RSA PKCS #1.5
      signatures. There are no known practical attacks against the old
      code, but the comparison-based approach is theoretically more
      sound. Thanks to Sze Yiu Chau from Purdue University for
      pointing this out.

    * (MatrixSSL FIPS Edition only:) Fix DH key exchange when using DH
      parameter files containing optional privateValueLength argument.

    * psX509AuthenticateCert now uses the common psVerifySig API for
      signature verification. Previously, CRLs and certificates used
      different code paths for signature verification.

## Changes between 3.9.5 and 4.0.0 [August 2018]

This version adds support for RFC 8446 (TLS 1.3), new APIs for
configuring session options as well as fixes to security
vulnerabilities.

- TLS:

    * Added support for TLS 1.3 (RFC 8446 version) as well as draft
      versions 23, 24, 26 and 28.
    * Supported TLS 1.3 handshake types:
        - Basic handshake with server authentication
        - Incorrect DHE key share (HelloRetryRequest) handshake
        - PSK handshake
        - Resumed handshake
        - 0RTT data handshake
    * Supported TLS 1.3 ciphersuites:
        - TLS_AES_128_GCM_SHA256
        - TLS_AES_256_GCM_SHA384
        - TLS_CHACHA20_POLY1305_SHA256
    * Supported key exchange modes in TLS 1.3:
        - DHE with the ffdhe2048, ffdhe3072 and ffdhe4096 groups
        - ECDHE with the P-256, P-384, P-521 and X25519 groups
        - PSK with (EC)DHE
        - PSK only
    * Supported signature algorithms in TLS 1.3:
        - ECDSA with P-256, P-384 and P-521
        - Ed25519
        - RSASSA-PSS
        - RSA PKCS #1.5 (certificates only)
    * Supported PKI features in TLS 1.3:
        - X.509 certificates
        - CRLs
        - OCSP stapling
    * Supported TLS 1.3 extensions:
        - supported_versions
        - supported_groups
        - key_share
        - signature_algorithms
        - signature_algorithms_cert
        - server_name
        - certificate_authorities
        - cookie
        - status_request
        - max_fragment_length
    * Support for TLS 1.3 record padding
    * Fixed several client-side crashes and undefined behaviours on
      maliciously crafted server messages. The bugs were found using
      TLS-Attacker. Thanks to Robert Merget from the Ruhr-University
      Bochum for reporting these.
    * Added the matrixSslSessOptsSetServerTlsVersions and
      matrixSslSessOptsSetClientTlsVersions APIs for selecting the
      supported protocol versions at run-time. Please consult the API
      reference for details.
    * Added a couple of TLS 1.3 specific APIs:
         - matrixSslSessOptsSetSigAlgsCert
         - matrixSslSessOptsSetKeyExGroups
         - matrixSslGetEarlyDataStatus
         - matrixSslGetMaxEarlyData
         - matrixSslLoadTls13Psks
         - matrixSslSetTls13BlockPadding
    * Added an API for selecting supported signature algorithms:
      (usable in both TLS 1.3 and TLS 1.2):
         - matrixSslSessOptsSetSigAlgs
    * Added new example configurations. The recommended configuration
      for using TLS 1.3 and below is tls13 (Commercial Edition) or
      nonfips-tls13 (FIPS Edition)
    * Updated and improved the Developer Guide and the MatrixSSL APIs
      reference document.
    * Improved the example client and server programs and fixed bugs.
    * Resend user extensions (e.g. SNI) when responding to HelloRequest
    * sslTest now allows specifying the ciphersuites and protocol
      versions to test via environment variables.
    * Improvements to identity management, including support for
      loading multiple identities (key and cert pairs) during
      initialization and postponed key and cert loading. See the
      MatrixSSL Developer Guide for details.
    * Refactored key loading and protocol version negotiation.
    * Fixed server-side signature algorithm selection when the server
      certificate is signed with a different algorithm (RSA or ECDSA)
      than the public key contain therein.
    * Much improved TLS-level debug prints and logging
      (tlsTrace.c). USE_SSL_HANDSHAKE_MSG_TRACE now consistently
      enables messages such as "parsing/creating handshake message X
      or extension Y". USE_SSL_INFORMATIONAL_TRACE now prints out more
      details on the contents of handshake messages and extensions.
    * Refactored public header files.

- Crypto:

    * NCC Group'ss Keegan Ryan has found a side-channel attack
      affecting multiple cryptographic libraries. The "ROHNP" Key
      Extraction Side Channel (CVE-2018-0495) has been fixed.
    * Added support for Ed25519 signatures in TLS 1.3
    * Added support for ECDHE with X25519 in TLS 1.3
    * Added algorithm-independent signature and verification APIs:
      psSign and psVerify.
    * Source file reorganization. New new naming scheme aims for
      better consistency, clarity and makes it easier to ifdef out
      unneeded features.
    * Added psEccWritePrivKeyMem and psEccWritePrivKeyFile the public
      crypto API

- X.509 and PKCS standards

    * Fixed processing of indefinite expiration date (31.12.9999).
    * Basic Constraints no longer unconditionally added when generating CSR data
    * Session option for requesting subrange of allowed tls versions.
    * Specify certificate validity dates when generating certificate.
    * Support for reading PKCS #12 and CA certificates from memory
      (der encoded).
    * Support for key usage encipher only and decipher only bits
      in generating certificate generation.
    * Option for MD2/MD4/MD5 signatures compatibility on certificates.
    * X.509 certificates allow NIL character at the end of GeneralName field.
      This is for compatibility with various other products.
    * It is now possible to compile X.509 certificate and CSR
      generation code only ECC or RSA support for smaller footprint.
    * Added Ed25519 specific functions such as psEd25519ParsePrivKey,
      psEd25519Sign, etc.

- Other changes

    * Added export.mk, which generates example binary packaging of a
      previously compiled MatrixSSL package and includes two of the
      example applications within the package. This package shows how
      to export MatrixSSL includes and libraries outside the source tree
      keeping configuration with the includes.

- Known issues

    * The TLS 1.3 code has not yet been fully optimized for footprint.
    * If the client sends a TLS 1.3 ClientHello with X25519 as the key
      exchange group, the server downgrades to TLS 1.2 but still
      wishes to use X25519, the handshake will fail, because MatrixSSL
      does not yet support X25519 in TLS 1.2 and below.