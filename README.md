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
  <td><b>v<sub>out</sub></b></td>
  <td><b>ScriptPubKey ("Address")</b></td>
  <td><b>BCH<br/>amount</b></td>
  <td><b>Implied token amount<br/>(base units)</b></td>
</tr>
  <tr>
    <td>0</td>
   <td>
   OP_RETURN<br/>
   &lt;Declare Attestation&gt; (0x6dff)<br/>
   &lt;transaction_type: 'GENESIS'&gt; (7 bytes, ascii)<br/>
   &lt;token_ticker&gt; (0 to ∞ bytes, suggested utf-8)<br/>
   &lt;token_name&gt; (0 to ∞ bytes, suggested utf-8)<br/>
   &lt;token_document_url&gt; (0 to ∞ bytes, suggested ascii)<br/>
   &lt;token_document_hash&gt; (0 bytes or 32 bytes)<br/>
   &lt;decimals&gt; (1 byte in range 0x00-0x09)<br/>
   &lt;mint_baton_vout&gt; (0 bytes, or 1 byte in range 0x02-0xff)<br/>
   &lt;initial_token_mint_quantity&gt; (8 byte integer)
   </td>
    <td>any</td>
    <td>0</td>
  </tr>
</table>
