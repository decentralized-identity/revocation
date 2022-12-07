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

This specification describes a privacy-preserving revocation method for Verifiable Credentials. A _Non-Revocation Token_ that contains a cryptographic accumulator signed by the issuer is used by the holder to prove a Verifiable Credential hasn't been revoked for a particular timestamp.

## Introduction

A non-revocation token contains a cryptographic accumulator signed by the credential issuer, following the signing and proving processes laid out in [zk-SAM](https://hackmd.io/vTyqrJc9QoKgThqQpVtP3g?view). This allows for the holder to prove non-revocation of the credential for a particular timestamp, without revealing any extra correlating information. Like current BBS+ signature implementations the pairing-friendly BLS12-381 elliptic curve is used.

This approach to proving the non-revocation of a verifiable credential has characteristics that meet some practical requirements for a number of important use cases. See the [Requirements and Characteristics](#requirements-and-characteristics) section later in this document.

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

- Practical with large credential registries, where large is 1M+ per revocation registry and issued credentials in the many million (e.g. "Jurisdictional Scale").
  - Assumes that the verifier will receive an Id for the revocation registry in which the Holder/Prover's credential resides.
  - Assumes that a revocation registry size of 1M is sufficient to "hide in the crowd"
- No unique identifier for the holder/prover or the credential is given to the verifier that can be used for correlation.
- Moderate time and resources required by a mobile holder/prover, including for both gathering the data for the construction of the proof and creating the proof.
  - Both parts of the proof construction process must be considered. It is not helpful to be able to construct a proof in nanoseconds if the holder/prover must execute many remote API calls to collect the data to construct the proof. For example, where the holder has to collect deltas going back to the last time they constructed a proof.
  - The registry as received by the holder/prover cannot be too large -- e.g. cannot be multi-megabytes in size.
  - A holder/prover should be able to make a copy of the revocation registries for some credentials for offline use without filling up storage.
  - Likewise, the holder/prover cannot hold/cache megabytes of data per credential just to support fast proof construction.
- Fast verification of the proof, with at most one or two real-time call(s) to an external service to get any data needed to verify the proof of non-revocation.
  - The verifier can do more caching than the holder/prover, but should do so based on the type of credential, not the specific proof being processed.
- A reasonable load on the issuer to construct and publish updates to the registry.
  - "Reasonable" in this case is much higher resource usage than for the mobile holder/prover or verifier based on the assumption that an issuer providing credentials to millions of holders is running the issuer service on enterprise grade servers.
    - For example, up to several seconds of processing to produce a revocation registry is not out of the question, as this should be needed only on a periodic basis -- e.g. at most several times a day in the most extreme cases, more often daily or less.
  - Nice to have is if the artifact is a file that can be put on a file server, or perhaps a combination of a ledger entry and a file that can be shared anywhere (CDN, etc.).
    - A goal is to NOT have a "call home" place for getting the file, so the Issuer knows when the holder/prover is using their credential. This is harder to do if the revocation registry is not just a plain file server, but rather a service that calculates a revocation registry response to requests from holder/provers.

## Open Issues

- [Can revocation proofs (signature proof of knowledge + proof of non-membership) be supported?](https://github.com/decentralized-identity/revocation/issues/15)
- [What should the recommended block size(s) be – need to pick a few standard sizes for libraries to support, for 10K, 100K, 1M+ credentials. Will also affect the hardware demands for provers (smaller block sizes may be chosen to better support IoT devices)](https://github.com/decentralized-identity/revocation/issues/17)
- [Add support for an accepted timestamp interval signed by the issuer? ie. an expiry time](https://github.com/decentralized-identity/revocation/issues/18)
- [Support a standard range proof on the timestamp value in lieu of revealing the exact timestamp](https://github.com/decentralized-identity/revocation/issues/21)
- [Define some potential authenticated timestamp publication methods (ie. blockchain-based, signature-based)](https://github.com/decentralized-identity/revocation/issues/19)
- [Update to match latest BBS+ spec](https://github.com/decentralized-identity/revocation/issues/16)
