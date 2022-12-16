# Introduction

The revocation work item group is part of the crypto-wg (https://identity.foundation/working-groups/crypto.html).
We will define one or more revocation methods for verifiable credentials.
The base of this working group is the definition of the work item. You can find the description here https://github.com/decentralized-identity/crypto-wg/blob/main/work_items/revocation_methods_for_verifiable_credentials_.md

# Meetings

Regular meetings will be held bi-weekly on Thursday at 1:00PM EST.
Zoom link: https://us02web.zoom.us/j/82260779505?pwd=RTVsNGZGaFZ2cHdCd0hBanNvQnRudz09
Agenda and meeting minutes can be found in the folder /meeting_minutes

# Work item owner

- Brian Richter (brian@aviary.tech)
- Andrew Whitehead (andrew.whitehead@portagecybertech.com)
- Mike Lodder (redmike7@gmail.com)
- Dave Huseby (dave@cryptid.tech)

# Documents

### Specifications

The following revocation methods are currently being explored in the form of spec-up specifications:

- [Non-Revocation Token](https://identity.foundation/revocation/non-revocation-token/)

_If you would like to explore other revocation methods please join one of the working group calls and submit a proposal._

### Spreadsheet

All information can be found in the following spreadsheet:

- https://docs.google.com/spreadsheets/d/1B6Koo8_wUoN4SPDvX7gaKBtkKBr6efOLwKIeVoy4mdI/edit?usp=sharing![image](https://user-images.githubusercontent.com/17032235/141115199-8ba436f5-76c3-4b91-91b4-92923b8b55db.png)

It includes the list of revocation methods, the assessment setup and assessment results (if available)

### Presentation

A presentation of the revocation working group and the schemas can be found in the following presentation (under construction)

- https://docs.google.com/presentation/d/1neH5LxuWlcagAa6jUcrikVOU_fgSFhWAiascqdaRl1w/edit#

# Procedure and Status

We do not start with a specific accumulator method. We will first collect all possible methods, group them, assess them and then select suitable methods.
With this method we want to avoid iterations and ensure that we have assessed all promissing methods.

| Step # | Step                                                                                                                                                                                                                      | Status      | Owner |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ----- |
| 1      | Collect revocation methods                                                                                                                                                                                                | in progress | tbd   |
| 2      | Define assessment setup (#of entries, #of revocations per epoch,...)<br />for performance testing                                                                                                                         | in progress | tbd   |
| 3      | - Define requirements for privacy (correlation (linked, linkable), traceability,<br />leaking of usage data,..)<br />- Define performance requirements (computational effort, storage,<br />data transmission, sizes,...) | in progress | tbd   |
| 4      | Assess the revocation methods and define one or more revocation methods<br />that fits the purpose                                                                                                                        | To-Do       | tbd   |
| 5      | Create the protocol                                                                                                                                                                                                       | To-Do       | tbd   |

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
