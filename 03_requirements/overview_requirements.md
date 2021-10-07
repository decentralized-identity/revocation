# Requirements

The following are some requirements of an effective revocation mechanism:

- Practical with large credential registries, where large is 1M+ per revocation registry and issued credentials in the many million (e.g. "Jurisdictional Scale").
  - Assumes that the verifier will receive an Id for the revocation registry in which the Holder/Prover's credential resides.
  - Assumes that a revocation registry size of 1M is sufficient to "hide in the crowd"
- No unique identifier for the holder/prover or the credential is given to the verifier that can be used for correlation.
- Moderate time and resources required by a mobile holder/prover, including for both gathering the data for the construction of the proof and creating the proof.
  - Both parts of the proof construction process must be considered. It is not helpful to be able to construct a proof in nanoseconds if the holder/prover must execute many remote API calls to collect the data to construct the proof.  For example, where the holder has to collect deltas going back to the last time they constructed a proof.
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