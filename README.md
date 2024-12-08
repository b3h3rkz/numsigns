# NumSigns: A Universal Protocol for Cryptographic Identity Derived from Phone Verification

**Version:** Draft 1.0

**Date:** December 2024

## Executive Summary

NumSigns is a protocol that transforms verified real phone numbers into globally resolvable, cryptographic digital identities (DIDs). By anchoring trust in the well-established and ubiquitous phone number system, NumSigns enables verifiable credentials (VCs) to attest that a given decentralized identifier was originally linked to a controlled, real-world phone number. This process preserves user privacy (via hashing and credentialization) while providing a universal, extensible foundation for identity verification, secure messaging relays, payment endpoints, and more.

Traditional phone-based identity checks rely on one-time passwords (OTPs) sent via SMS — a fragile, costly, and privacy-invasive process. 

NumSigns leverages the existing global numbering infrastructure as a trust anchor without continuously exposing the underlying real phone number. Once verified, a virtual number and a DID are issued, along with a VC attesting to the verification. Any relying party (e.g., messaging apps, online retailers) can trust these credentials without re-verifying each number.

The NumSigns approach blends the familiar mental model and regulatory stability of phone numbers with the powerful privacy and interoperability of decentralized identity standards (DIDs, VCs). As a result, it opens pathways for more secure user onboarding, reduced spam, cost savings for service providers, and integration with advanced services like message relays, payment addresses, and beyond.

---

## Background

### The Problem 

Identity verification online is riddled with inefficiencies. While phone number-based verification is common, it has significant drawbacks:

- **Lack of Privacy:** Users often share their real numbers, risking spam, phishing, and unwanted contact.  
- **High Costs for Businesses:** Each verification (SMS OTP) costs money and adds friction.  
- **No Unified Standard:** Each service repeats the verification process.  
- **Data Exposure and Security Risks:** Centralized databases holding phone numbers are prime targets for attackers.

### Limitations of Existing Approaches

- **Centralized Email/Phone Verification:** Requires repeated checks, storing personally identifiable information (PII), and trusting each platform’s security posture.  
- **Federated Logins (OAuth/OpenID Connect):** Rely on large identity providers (e.g., Google, Facebook), introducing vendor lock-in, data sharing, and privacy trade-offs.  
- **Self-Sovereign Identity (SSI) and DIDs Alone:** While powerful, these still struggle with trust bootstrapping—how to assure that a DID corresponds to a real person without relying on complex, unfamiliar processes.

### Opportunity

NumSigns merges these worlds by using phone verification (ubiquitous and well-understood) as a one-time trust anchor. After initial verification, all subsequent identity assurances can be made cryptographically and privacy-preservingly through VCs and DIDs.

---

## Core Concepts

### From Real Phone Number to Virtual Number

**Concept**:  
A user provides their real phone number to a Verification Service. The service sends an SMS or voice call OTP to confirm control. 
An existing service that has already verified a user's phone number can use this to issue a NumSigns DID.
Upon success, the service:

1. Issues a **Virtual Number**, formatted as:  
   **`{CountryCode}00{10-digit unique number}`**  
   This format ensures no collisions with real E.164 numbers and provides a globally consistent namespace.

2. Creates a **Decentralized Identifier (DID)** associated with this Virtual Number.

3. Issues a **Verifiable Credential (VC)** stating that this DID is controlled by a user who verified a real phone number.

### Hashing & Privacy

The real phone number is hashed (e.g., via SHA-256) after verification, never stored in plaintext. This ensures that neither the Verification Service nor relying parties can trivially correlate the DID back to the original number, significantly reducing privacy risks.

### DIDs and VCs

- **DIDs**: A standard by W3C allowing entities to have decentralized, cryptographic identifiers independent of centralized authorities.  
- **VCs**: Signed digital credentials that assert claims about a DID, such as “This DID was verified from a real phone number at issuance time.”

By combining these, NumSigns establishes trust in the DID without continuously exposing PII.

---

## System Architecture and Flow

### Actors

- **User**: Owns a real phone number, wishes to obtain a virtual, privacy-preserving digital identity.  
- **Verification Service (Issuer)**: Performs phone verification and issues the DID + VC.  
- **Relying Parties (Verifiers)**: Services like Signal, Amazon, or any online platform that accepts the VC instead of performing their own phone verification.  
- **Relay Services**: Optional endpoints that handle messaging or data routing associated with the virtual number’s DID.

### Data Flow

1. **Verification Step**:  
   User provides real number → Verification Service sends OTP → User proves control → Verification successful.

2. **Issuance Step**:  
   Verification Service creates DID (e.g., `did:web:example.com:user123` or `did:ion:...`) and Virtual Number.  
   Issues a VC asserting successful phone verification.

3. **Usage Step**:  
   When a user presents this Virtual Number and DID to a third party, that party fetches the DID Document, checks the VC’s signature against the issuer’s public keys, and trusts that the virtual number was once tied to a verified phone number.

4. **Service Discovery**:  
   DIDs can list various endpoints (e.g., messaging relays, payment addresses). Third parties can find these endpoints and route messages or funds without knowing the real phone number.

---

## Technical Details

### DID Methods

NumSigns supports multiple DID methods to ensure broad interoperability:

- **did:web**: Simple implementation relying on DNS and HTTPS.  
- **did:ion**: Decentralized, blockchain-anchored approach using Sidetree for strong trust minimization.

This dual support offers a balanced approach—easy adoption via did:web and fully decentralized trust with did:ion.

### VC Schema

A VC for NumSigns might include:  
- Issuer DID (the Verification Service)  
- Subject DID (the user’s DID)  
- Claim: “Subject’s DID was originally verified against a real phone number on {date}.”  
- Non-correlatable hash of the original number, known only to issuer and subject, ensuring no external party can identify the number directly.

### Hashing and Cryptography

- **Hashing Real Numbers**: `H = SHA-256(phone_number)` ensures irreversible transformation.  
- **Key Management**: The user holds a private key controlling the DID. The issuer and verifiers rely on cryptographic signatures for trust.

---

## Ecosystem Integration

### Message Relay

A DID’s service endpoint might specify a relay URL. Instead of Amazon sending an SMS (costly and exposes the real number), it sends a message to the relay endpoint. The relay forwards it to the user securely, possibly with end-to-end encryption (DIDComm), maintaining user privacy and reducing spam.

### Payment Integration

The DID Document can also list a payment endpoint or static Bitcoin address for example. A mobile money account can also be used. Anyone can send funds to the user’s virtual identity without revealing personal details. Over time, more sophisticated integrations (LNURL, BIP47) can ensure privacy and seamless payments. 

### Other Services

Email proxies, VoIP endpoints, federated social IDs—any service that can be represented as a DID endpoint can be integrated. The user controls this identity hub, updating or rotating endpoints as needed.

---

## Security and Trust Model

### Attack Vectors

- **Credential Forgery**: Prevented by cryptographic signatures on VCs and DID Documents.  
- **Sybil Attacks**: Limited by the requirement of initially verifying a real phone number—costly for mass attackers.  
- **Relay Impersonation**: Users and third parties verify the relay endpoint from the DID Document, signed by the user’s keys, ensuring authenticity.

### Revocation and Expiration

VCs can have expiration dates. If a user’s phone number changes or the Verification Service loses trust, credentials can be revoked. DID rotations let users migrate between verification services without losing continuity.

---

# Cryptographic Virtual Number Generation - Avoiding Collisions
Once a user’s real phone number has been verified, we generate a unique 10-digit identifier in a collision-resistant manner. By leveraging cryptographic hashing of a public key derived from a randomly generated key pair, we ensure that the resulting 10-digit number is essentially unique. The probability of collision is extremely low, given a well-chosen cryptographic hash function.

**Key Points**:  
- Each user gets a dedicated key pair (private/public key).  
- Hash the public key to produce a large, unpredictable number.  
- Convert the hash to a decimal and truncate to 10 digits.  
- Combine the user’s country code with "00" and the 10-digit identifier to form the Virtual Mobile Number (VMN).

This ensures that two independently generated virtual numbers are astronomically unlikely to collide.

## Algorithm Steps

1. **Generate a Cryptographic Key Pair**  
   - Use a strong elliptic-curve or RSA key generation algorithm.  
   - For this example, we’ll assume Ed25519 or secp256k1 keys (common in modern cryptographic systems).

2. **Extract the Public Key**  
   - The public key is a binary string (e.g., 32 bytes for Ed25519).

3. **Hash the Public Key**  
   - Use a secure hash function like SHA-256.  
   - `hash = SHA256(public_key_bytes)`

4. **Convert and Truncate**  
   - Use 64 bits (~2^64) of the hash output, which is far larger than 10^10, ensuring a uniform distribution.
   - Apply % 10000000000 (10^10) to get a number in the range `[0, 9,999,999,999]`
   - Zero-pad the number to ensure it’s always 10 digits

5. **Assemble the VMN**  
   - If the user’s country code is `cc` (e.g., `1` for the US),  
   - VMN = `cc + "00" + 10-digit_identifier`

**Example**:  
- Country Code: `1`  
- 10-digit ID: `1234567891`  
- VMN = `1001234567891`

## Collision Resistance

The purpose of the virtual number system is to establish a globally unique, cryptographically controlled identifier that can represent a user or entity in a privacy-preserving manner. While phone numbers are a familiar anchor, they expose personal information and rely on telecom infrastructure. Virtual numbers look like phone numbers but are derived using cryptography, ensuring they remain independent of telecom providers and do not compromise user privacy.

**Key Goals**:  
1. **Uniqueness**: Each virtual number must be unique worldwide, ensuring no two users are assigned the same identifier.  
2. **Cryptographic Assurance**: By tying the creation of the virtual number to a cryptographic process, we prevent tampering and guarantee that the identifier is unpredictable and verifiable.  
3. **Scalability and Decentralization**: Virtual numbers are created algorithmically, without a single authority controlling the namespace. This approach supports unlimited global growth while maintaining trust and security.

**How Virtual Numbers Are Generated**:  
1. **Key Generation**:  
   - Begin by generating a cryptographic key pair using an elliptic curve like Ed25519 or secp256k1.  
   - The public key derived from this pair is a large, random binary string that cannot be guessed or influenced externally.

2. **Hashing the Public Key**:  
   - Apply a secure hash function (e.g., SHA-256) to the public key.  
   - The hash function outputs a fixed-length, uniformly distributed number. Even the smallest input change radically alters the output, ensuring unpredictability and collision resistance.

3. **Deriving a 10-Digit Identifier**:  
   - Convert the hash output to a decimal representation and then extract a 10-digit number from it.  
   - This is typically done using a modulus operation (`hash_decimal % 10^10`) to produce a 10-digit integer in the range 0 to 9,999,999,999.  
   - If necessary, this number is zero-padded to ensure it always has exactly 10 digits.

4. **Constructing the Virtual Number**:  
   - Prepend a recognized country code and "00" to this 10-digit identifier.  
   - The final format looks like:  
     \[
     \text{Virtual Number} = \text{CountryCode} + "00" + \text{10-digit ID}
     \]

   The "00" creates a distinct namespace separate from real phone numbers, ensuring no overlap with existing numbering plans.

**Why 10 Digits?**:  
Selecting a 10-digit identifier provides a vast namespace of 10 billion possible values for each country code. This large space:

- **Ensures Extremely Low Collision Probability**: With 10 billion unique combinations, the chance of two independent keys producing the same identifier is negligible, even at global scale.  
- **Accommodates Global Adoption**: As the system grows, billions of users can be assigned virtual numbers without exhausting the available range.  
- **Future-Proofing the System**: If scalability becomes a concern, a proposal could be to increase the ID length to 11 digits or support multiple ID lengths.

**User Impact**:  
Users will rarely need to manually type or memorize these virtual numbers. Instead, applications and services handle lookups, routing, and verification behind the scenes. The 10-digit length is a technical choice that makes the system robust, scalable, and reliable. From the user’s perspective, the result is a secure and trustworthy identity layer that works seamlessly with familiar phone number patterns.

By generating virtual numbers through cryptographic hashing and selecting a 10-digit length, we ensure that the system can scale globally, maintain extremely low collision rates, and operate independently of traditional telecom constraints—all while preserving user privacy and security.
---
## Generating Virtual Numbers
## JavaScript Code Sample

Below is a Node.js code example using common libraries (`crypto` for hashing). For key generation, we’ll use a simplified example assuming we have a public key available. In a the reference implementation we use `@noble/secp256k1`.

**Dependencies**:  
- Node.js built-in `crypto` module.
- For key generation, this example will mock a random 32-byte public key. In a production environment, generate a real key pair using a proper crypto library.

```js
const crypto = require('crypto');

/**
 * Generate a cryptographically unique 10-digit identifier:
 * 1. Generate a public key (mocked here as random bytes).
 * 2. Hash the public key using SHA-256.
 * 3. Take the first 8 bytes of the hash, interpret as a 64-bit integer.
 * 4. Mod by 10^10 to ensure we have a 10-digit number.
 * 5. Zero-pad to ensure exactly 10 digits.
 */

function generate10DigitID() {
  // Mock public key generation (32 bytes).
  // In production, replace with actual cryptographic key generation.
  const publicKey = crypto.randomBytes(32);

  // Hash the public key using SHA-256
  const hash = crypto.createHash('sha256').update(publicKey).digest();

  // Extract first 8 bytes as a 64-bit unsigned integer
  const firstEightBytes = hash.readBigUInt64BE(0);

  // Mod by 10^10 to get a number in the range [0, 9,999,999,999]
  const idBigInt = firstEightBytes % 10000000000n;

  // Convert to string and zero-pad to 10 digits
  const idStr = idBigInt.toString().padStart(10, '0');
  return idStr;
}

// Example usage:
const countryCode = '1'; // e.g., US
const tenDigitID = generate10DigitID();
const vmn = `${countryCode}00${tenDigitID}`;

console.log('Generated 10-digit ID:', tenDigitID);
console.log('Virtual Mobile Number:', vmn);

```
**Example Output**:  
```
Generated 10-digit ID: 0123456789
Virtual Mobile Number: 1000123456789
```

---

## Integration into the Issuance Flow

- After the Verification Service confirms the phone number, it calls `generate9DigitID()` to produce the unique identifier.  
- Combine it with the user’s country code and “00” prefix to form the VMN.  
- Store the VMN, public key, and DID for future reference.  
- Include the VMN in the user’s final credential package.

---

## Compliance and Regulatory Considerations

**AML/KYC**: The NumSigns protocol is identity infrastructure. It does not inherently violate AML laws. Financial service providers adopting this model must still comply with AML/KYC regulations. They can leverage VCs to streamline identity checks, but compliance duties remain with them.

**Data Protection**: By minimizing the exposure of raw phone numbers, NumSigns aligns well with privacy-focused regulations (e.g., GDPR). It reduces PII leakage risk and gives users more control over their data.

---

## Implementation and Reference Designs

### MVP and Reference Implementation (in the works)

- **Verification Service**: A Node.js server that:
  - Sends an OTP to verify a phone number.  
  - Issues a DID using did:web (hosting a did.json file at a known URL) and optionally did:ion.  
  - Generates and signs a VC.
  
- **DID Resolver**: A simple resolver tool can fetch the DID Document from the appropriate method.

- **Credential Verification**: A CLI or web tool demonstrates how third parties verify the VC and trust the virtual number.
---
## References
- W3C DID Core: [https://www.w3.org/TR/did-core/](https://www.w3.org/TR/did-core/)
- Web5 :[https://developer.tbd.website/docs/web5/]
- W3C Verifiable Credentials: [https://www.w3.org/TR/vc-data-model/](https://www.w3.org/TR/vc-data-model/)  
- Decentralized Identity Foundation: [https://identity.foundation/](https://identity.foundation/)  
- ION DID Method: [https://github.com/decentralized-identity/ion](https://github.com/decentralized-identity/ion)  
- BIP47 Payment Codes: [https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki)
- E164 [https://www.itu.int/ITU-D/treg/Events/Seminars/2010/Ghana10/pdf/Session2.pdf]
---

**This draft whitepaper is a starting point.** We are happy to discuss feedback and ideas that could improve the protocol.
