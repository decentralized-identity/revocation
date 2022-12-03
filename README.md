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
