# DRAFT
# Bitcoin-Cash-Attestation-Protocol

## Authors

## Acknowledgements

# SECTION I: BACKGROUND
## Introduction
An attestation plays a crucial role in establishing trust and credibility by confirming the authenticity, accuracy, and reliability of statements, documents, or events. Through the process of attestation, evidence is gathered and verified to support the validity of claims, often in the form of written statements or certifications. Attestations on the blockchain leverage the transparency, immutability, and decentralized nature of the technology to verify the authenticity of data, establish trust in digital identities, and validate external data sources. Below is a proposed method of attestation on Bitcoin Cash that leverages and extends out existing protocols and infrastructures, making it relatively easy to adopt and maintain.

## Summary
This Bitcoin Cash Attestation Protocol records the metadata onto OP_RETURN transaction outputs. The attestation format will follow the action prefix format of Memo.Cash, with the first prefix starting at hex value 0x6dff and counting down for subsequent action prefices. Indexing for attestations can focus on individual attestations or provide full attestation history depending on use case.

## Example Applications
1) Supply chain providence - attestations on the blockchain can be used to verify the origin, authenticity, and quality of products at each stage of the supply chain
2) Health care record - patients can have their medical records attested by healthcare providers, which can include diagnostic reports, lab results, and treatment history
3) Auditing report - audit attestations can be verified by regulators, external auditors, or other authorized parties, ensuring the accuracy and integrity of financial data

# SECTION II: PROTOCOL SPECIFICATION
## Protocol Overview
The protocol defines 3 basic types of attestation transactions, which are stored within OP_RETURN outputs. 
1) Declare Attestation Schema
2) Create & Sign Attestation
3) Revoke Signed Attestation

## Transaction Details
<b>Declare Attestation Schema<b>

<table>
  <tr>
    <td><b>Action</b></td>
    <td><b>Prefix</b></td>
    <td><b>Values</b></td>
  </tr>
  <tr>
    <td>Declare Attestation Schema</td>
    <td>0x6dff</td>
    <td>&lt;Attestation Struct&gt; (1-217 bytes)</td>
  </tr>
  <tr>
    <td>Create & Sign Attestation</td>
    <td>0x6dfe</td>
    <td>&lt;Signed Message&gt; (1-217 bytes)</td>
  </tr>
  <tr>
    <td>Revoke Signed Attestation</td>
    <td>0x6dfd</td>
    <td>&lt;Revoke Signed Message&gt; (1-217 bytes)</td>
  </tr>
</table>

### Example Attestation - Schema and Signage for BCH-ETH Swaps
Declare Attestation Schema
```
{
  bchSender: address,
  bchReceiver: address,
  bchAmount: int,
  bchEscrow: P2SH32,
  bchExpiry: int,
  ethSender: address,
  ethReceiver: address,
  ethAmount: int,
  ethEscrow: address,
  ethExpiry: int,
}
```
Both parties in swap sign the attestation with parameters to the schema filled

Alice's Signed Attestation: LXH3hvPIRMBSVXKOmIwx2860KbL74UmYj/QumkxR+LUwHSFZ+b0ZLdLUpgMhGiomf6CR4bhYTDYV4zNlwWtNCiqfI=

Bob's Signed Attestation: LXIBiCWt62mI7KeC981vcvgg1x79XsSWhytNqKR9Y9AVzYQUn7L08zKlA+FKt+cJQvgh3VKFR9doZr9N/3G8ylX0Q=

Create OP_RETURN transaction with signed attestations

https://chipnet.imaginary.cash/tx/015e27d2f86e3811bbc5a78a6e389792f4556e5526a82ee21a2a1327ec020d92
