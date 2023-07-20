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

Sample Use Case - BCH cross-chain swaps

# SECTION II: PROTOCOL SPECIFICATION
## Protocol Overview
The protocol defines 3 basic types of attestation transactions, which are stored within OP_RETURN outputs. 
1) Register Attestation Schema (optional)
2) Publish Signed Attestation
3) Revoke Signed Attestation

Registering an attestation schema helps coordinate setting up the schema data for future attestations. If a private attestation is preferred (i.e. interaction between 2 parties), the signed attestation can be directly published with OP_RETURN without referencing a registered schema.

## Schema Fields
Schemas adopt the CashScript ABI for acceptable types. Visit [CashScript.org](https://cashscript.org/docs/language/types) for more details.
<li><b>bool</b> - true or false</li>
<li><b>int</b> - 64 bit-sized signed int</li>
<li><b>string</b> - UTF8 encoded bytes</li>
<li><b>bytes</b> - byte sequence, optionally bounded by byte length</li>
<li><b>addr</b> - address, can be size bytes20 or bytes32 (for P2SH32)</li>
<br />
Additionally, Transaction ID is assigned a special bytes32 value
<li><b>txid</b> - transaction id in size bytes32</li>

## Transaction Details
<table>
  <tr>
    <td><b>Action</b></td>
    <td><b>Prefix</b></td>
    <td><b>Values</b></td>
  </tr>
  <tr>
    <td>Register Attestation Schema</td>
    <td>0x6dff</td>
    <td>&lt;Attestation Struct&gt; (1-217 bytes)</td>
  </tr>
  <tr>
    <td>Publish Signed Attestation</td>
    <td>0x6dfe</td>
    <td>&lt;Signed Unregistered Attestation&gt; (1-217 bytes) <br /><b>OR</b><br /> &lt;Registered Schema TxID &gt; (32 bytes)<br /> &lt;Signed Registered Attestation&gt; (1-184 bytes)</td>
  </tr>
  <tr>
    <td>Revoke Signed Attestation</td>
    <td>0x6dfd</td>
    <td>&lt;Signed Attestation TxID&gt; (32 bytes) <br /> &lt;Revoke Signed Message&gt; (1-184 bytes)</td>
  </tr>
</table>

Signed Attestation on the OP_RETURN is used as part of the CashScript constructor parameter. Subsequently, OP_CHECKDATDASIG is used to validate the signed message against the keypairs involved in the CashScript contract transaction.

## Indexing Registered Attestation Schemas
While unregistered attestations can be created privately among participants, registered schemas have the added benefit of coordinating public participants. Schemas, signed attestations and revocation can be indexed by the specifed action prefix following OP_RETURN. Users can choose their preferred indexing method when quering attestation data.

### Sample CashScript code
```
pragma cashscript ^0.8.0;

contract Attestation(datasig signedAttestation) {
  function spend(pubkey alicePk, sig aliceSig, bytes attestation) {
    require(checkDataSig(signedAttestation, attestation, ownerPk));
    require(checkSig(aliceSig, alicePk));
  }
}
```

**Note:** CashScript contracts will generate the same script hash (either P2SH or P2SH32) with the same constructor parameter(s). This allows for coordination among multi-party attestations

## Sample BitDB query for declared attestation schemas
```
{
  "v": 3,
  "q": {
    "find": { "out.b0": { "op": 106 }, "out.h1": "6dff" },
    "project": { "out.$": 1 }
  },
  "r": {
    "f": "[ .[] | {msg: .out[0].s2} ]"
  }
}
```
Visit [Fountainhead.cash docs](https://docs.fountainhead.cash/docs/bitdb) for more information.

# Appendix
### Example Attestation - Schema and Signage for BCH-ETH Swaps
Register Attestation Schema
```
{
  bchSnd: addr,
  bchGet: addr,
  bchAmt: int,
  bchEsc: addr,
  bchExp: int,
  ethSnd: addr,
  ethGet: addr,
  ethAmt: int,
  ethEsc: addr,
  ethExp: int,
}
```
Both parties in swap sign the attestation with parameters to the schema filled

Alice's Signed Attestation: LXH3hvPIRMBSVXKOmIwx2860KbL74UmYj/QumkxR+LUwHSFZ+b0ZLdLUpgMhGiomf6CR4bhYTDYV4zNlwWtNCiqfI=

Bob's Signed Attestation: LXIBiCWt62mI7KeC981vcvgg1x79XsSWhytNqKR9Y9AVzYQUn7L08zKlA+FKt+cJQvgh3VKFR9doZr9N/3G8ylX0Q=

Create OP_RETURN transaction with signed attestations

https://chipnet.imaginary.cash/tx/015e27d2f86e3811bbc5a78a6e389792f4556e5526a82ee21a2a1327ec020d92

```
pragma cashscript ^0.8.0;

contract SwapLock(
    bytes20 senderPkh,
    bytes20 recipientPkh,
    int timeout,
    datasig senderSignedAttestation,
    datasig recipientSignedAttestation
) {
    // Require recipient's signature to match
    function exchange(
        bytes senderAttestation,
        bytes recipeintAttestation,
        pubkey recipientPubKey,
        sig recipientSig,
        pubkey senderPubKey,
        sig senderSig
    ) {
        require(checkDataSig(senderSignedAttestation, senderAttestation, senderPubKey));
        require(checkDataSig(recipientSignedAttestation, recipeintAttestation, recipientPubKey));
        require(hash160(recipientPubKey) == recipientPkh);
        require(hash160(senderPubKey) == senderPkh);
        require(checkSig(recipientSig, recipientPubKey));
        require(checkSig(senderSig, senderPubKey));
    }

    // Require timeout time to be reached and sender's signature to match
    function withdraw(
        bytes revokeAttestation,
        pubkey senderPubKey,
        sig senderSig
    ) {
        require(tx.time >= timeout);
        require(hash160(senderPubKey) == senderPkh);
        require(checkSig(senderSig, senderPubKey));
        require(checkDataSig(userPWSig, password, ownerPk));
    }
}
```
