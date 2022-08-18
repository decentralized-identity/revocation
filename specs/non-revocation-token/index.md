# Non-Revocation Token

**Specification Status:** Draft

**Latest Draft:**
[https://identity.foundation/revocation/non-revocation-token](https://identity.foundation/revocation/non-revocation-token)

Editors:
~ [Andrew Whitehead](https://www.linkedin.com/in/andrew-whitehead-986a2913/)
~ [Brian Richter](https://www.linkedin.com/in/brianrichter3/)

Participate:
~ [GitHub repo](https://github.com/decentralized-identity/revocation)
~ [File a bug](https://github.com/decentralized-identity/revocation/issues)
~ [Commit history](https://github.com/decentralized-identity/revocation/commits/master)

---

## Abstract

A non-revocation token contains a cryptographic accumulator signed by the credential issuer, following the signing and proving processes laid out in [zk-SAM](https://hackmd.io/vTyqrJc9QoKgThqQpVtP3g?view). This allows for the holder to prove non-revocation of the credential for a particular timestamp, without revealing any extra correlating information. Like current BBS+ signature implementations the pairing-friendly BLS12-381 elliptic curve is used.

This approach to proving the non-revocation of a verifiable credential has characteristics that meet some practical requirements for a number of important use cases. See the [Requirements and Characteristics](#Requirements-and-Characteristics) section later in this document.

## Linking Issued Credentials

Each issued credential must contain two additional signed messages in order to enable revocation support, both represented as opaque 32-byte scalar values:

- The registry block identifier, constructed from the unique registry identifier and the credential's assigned block
- The assigned member value (unique per block)

In addition to these two signed messages, the prover will require the following:

- The credential's assigned block index
- The credential's member index within the assigned block
- The URI of the token registry metadata
- The URI of the latest token registry state

The signed credential envelope must reference a verification method which requires validation of the revocation status (example TBD). This is asserted by the prover in deriving a signature proof of knowledge for the issued credential, with known blinding factors for the two revocation messages, and then deriving the related token's non-revocation proof of knowledge using the same blinding factors. Both proofs must be included in the same challenge value calculation.

The verifier is responsible for choosing and enforcing the earliest accepted timestamp (method also TBD).

## Token Registry Metadata

The token registry metadata is likely unchanged for the duration of the registry and may be cached by provers. It is not required by verifiers. It includes the following:

- The public revocation key ($P_{2}z$): 96 bytes
- The block size (ie. 1024): 4 bytes
- The base accumulator value, randomly chosen ($V_0$): 48 bytes
- The registry member values, randomly chosen (32 bytes \* block size)
- The registry member witness values (48 bytes \* block size): calculated as $V_0(n + z)^{-1}$

## Token Registry State

The token registry state is updated each time outstanding revocations are published, or the number of issuable credentials is increased. It is not required by verifiers, although the latest timestamp (or guidelines regarding timestamp freshness) will need to be accessible.

Each registry state contains the following:

- The effective timestamp
- The starting and ending block indices
- A sequence of token data blocks

The number of data blocks will depend on the number of issuable credentials. There are three block types:

- Fully revoked: this block will have no associated accumulator or signature, as there are no provable credentials.
- Fully non-revoked: this block will inherit the base accumulator. A signature is provided.
- Partially revoked: this block will include a bit array indicating the revocation status of each of the member indices, as well as a signature. Members will derive the block's accumulator as well as their corresponding witness value for said accumulator using the witness values contained in the registry metadata.

## Signature Generation

- Set $q$, the registry block identifier as $\mathit{H2I}(\text{registry id} \parallel \text{block index})$
- Take $t$ as the timestamp value of the registry
- Take $V_0$ as the base accumulator from the registry metadata and calculate $V \gets V_0\prod_{n \in \mathcal{R}}(n + z)^{-1}$ as the accumulator for the registry block, where $z$ is the private revocation key and $\mathcal{R}$ is the set of revoked member values
- Take $H_1, H_2$ as the corresponding zk-SAM message generators for a signature with 2 messages
- Calculate $e \gets \mathit{H2I}(V \parallel q \parallel t)$
- Output $A \gets (P_1 + V + H_1{q} + H_2{t})(e + k)^{-1}$ where $k$ is the private issuance key

## Expected Data Sizes

- Registry metadata: (100 + 80 \* block size) bytes; 80Kb for block size 1024
- Registry state \[1024 block size; 1M credentials\]:
  - Every block revoked: <1Kb
  - No members revoked (~32 bytes per block): 30Kb
  - Every block partially revoked (~160 bytes per block): 153Kb
- Serialized non-revocation token (after extraction by the prover): 344 bytes
- Non-revocation token proof of knowledge: 368 bytes + metadata

## Implementation

See [sample implementation in Rust](https://github.com/andrewwhitehead/non-revocation-token) at GitHub.

## Current Timings

Tested on a 2017 Macbook Pro, with block size 1024. Further optimizations are yet to be applied:

- Create registry metadata: **0.25s**
  - performed once by the issuer when establishing a new registry
- Output a registry state for 1000 blocks, or about 1M credentials: **up to 2s**
  - performed by the issuer when publishing a new registry state
  - output a fully non-revoked block: **1.5ms**
  - output a partially revoked block: **2ms**
- Extract a non-revocation token for a partially-revoked block: **up to 0.5s**
  - performed by the prover after fetching a new registry state
  - TODO: explain method for deriving the witness and accumulator values
  - duration is expected to be reduced significantly
- Prepare a non-revocation token proof of knowledge: **5ms**
  - performed by the prover once per verification
- Verify token proof of knowledge: **12ms**
  - performed by the verifier

## Requirements and Characteristics

[To Do SWC]

## Open Questions

- Can revocation proofs (signature proof of knowledge + proof of non-membership) be supported?
- What should the recommended block size(s) be – need to pick a few standard sizes for libraries to support, for 10K, 100K, 1M+ credentials. Will also affect the hardware demands for provers (smaller block sizes may be chosen to better support IoT devices)
- Add support for an accepted timestamp interval signed by the issuer? ie. an expiry time
- Support a standard range proof on the timestamp value in lieu of revealing the exact timestamp
- Define some potential authenticated timestamp publication methods (ie. blockchain-based, signature-based)
