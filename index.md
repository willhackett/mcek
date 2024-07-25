# Internet Engineering Task Force (IETF) Draft Specification

### Title: Secure/Multipurpose Internet Mail Extensions (S/MIME) Extension for Mail Client Encryption Keys (MCEK)

#### Abstract

This document defines an extension to S/MIME for the lookup and management of encryption keys using the `keys._mcek` subdomain of the mail provider or listed MX records. It details the protocols and procedures for the synchronization, lookup, and usage of encryption keys and certificates for email communication. The extension aims to enhance security by automating key management and ensuring verified domain usage.

#### Status of This Memo

This document is an Internet-Draft and is subject to change. It expires six months after publication.

#### Table of Contents

1. Introduction
2. Terminology
3. Overview
4. Lookup and Storage Mechanism
5. Key Management
6. Key Expiry and Revocation
7. Certificate Lookup and Usage
8. Signing Authority Requirements
9. Client Behavior
10. Security Considerations
11. IANA Considerations
12. References

### 1. Introduction

S/MIME provides a mechanism for sending and receiving secure email messages. This extension specifies the use of a standardized host (`keys._mcek` subdomain) for the lookup and management of mail client encryption keys (MCEK). This extension ensures that encryption keys are securely stored, synchronized, and verified by a trusted authority.

### 2. Terminology

- **MCEK**: Mail Client Encryption Keys.
- **MCEK Host**: The DNS host used for key lookup (`keys._mcek` subdomain of the mail domain or listed MX records).
- **S/MIME**: Secure/Multipurpose Internet Mail Extensions.
- **PKI**: Public Key Infrastructure.
- **CA**: Certificate Authority.
- **Base64**: A method of encoding binary data as ASCII text.

### 3. Overview

The extension specifies a method for email clients to automatically lookup, store, and manage encryption keys using a designated DNS host. The keys are encrypted and synchronized by the server, with provisions for key expiry and retrieval of older keys for decryption purposes. Public keys are looked up using a base64 encoding of the recipient's username.

### 4. Lookup and Storage Mechanism

- **DNS Host**: The keys are stored and looked up at `keys._mcek` subdomain of the mail provider's domain or on one of the listed MX records.
- **Alternative Lookup Hosts**: In addition to the primary lookup host, clients may use the listed MX servers under the `/.well-known/mcek` path for key retrieval.
- **Public Key Lookup**: Public keys are available for lookup using a base64 encoding of the recipient's username as an HTTP GET parameter.
  - Example: For `user@example.com`, the HTTP GET request would be `GET https://keys._mcek.example.com?username=dXNlckBleGFtcGxlLmNvbQ==`.

### 5. Key Management

- **Key Storage**: Encrypted keys are stored on the MCEK server and synchronized across mail clients.
- **Private Key Synchronization**: While optional, private key synchronization is encouraged. Technologies like Apple iCloud Keychain or similar should be used to facilitate key rotation and historic decryption keys.
- **Key Encryption**: Keys must be encrypted using strong encryption algorithms (e.g., AES-256) before storage.
- **Synchronization**: Keys are automatically synchronized across all devices the user uses to access their email.

### 6. Key Expiry and Revocation

- **Auto-expiry**: Keys auto-expire after a specified period (e.g., 1 year), but older keys remain available for decrypting received mail.
- **Revocation**: Users can manually revoke keys if they suspect compromise.

### 7. Certificate Lookup and Usage

- **Certificate Authority**: S/MIME public keys must be signed by a Certificate Authority (CA) that has verified the domain.
- **Failed Lookups**: If a public key lookup fails, the server must return a S/MIME certificate for signing messages, even if it is a placeholder.
- **Certificate Retrieval**: Clients retrieve certificates at the time an email address is entered to determine key application.

### 8. Signing Authority Requirements

- **Verification**: Authorities must verify domain ownership before signing S/MIME certificates.
- **Availability**: Signing services must be made available for free to ensure widespread adoption and usage.

### 9. Client Behavior

#### Acquiring a S/MIME Certificate for Recipients

1. **Email Entry**: When an email address is typed into the email client, the client performs a lookup to `keys._mcek.domainName`.
2. **Base64 Encoding**: The client encodes the recipient's username in Base64 and appends it as a query parameter to the URL.
3. **DNS Query**: The client performs a DNS query to the constructed host.
4. **HTTP GET Request**: The client sends an HTTP GET request to the URL.
   - Example: `GET https://keys._mcek.example.com?username=dXNlckBleGFtcGxlLmNvbQ==`.
5. **Certificate Retrieval**: If a public key is found, it is retrieved and used for S/MIME encryption.
6. **Fallback**: If no key is found, the client should retrieve a placeholder S/MIME certificate to ensure messages can still be signed.

#### Retrieving the User's S/MIME Private Key and Certificate

1. **Initial Setup**: Upon initial setup or key rotation, the user generates a new private key and certificate.
2. **Encryption and Storage**: The private key is encrypted using a strong passphrase and stored on the MCEK server.
3. **Synchronization**: If private key synchronization is enabled, the encrypted private key is synchronized using a service like Apple iCloud Keychain.
4. **HTTP POST Request**: The client sends an HTTP POST request to retrieve the private key.
   - Example: `POST https://keys._mcek.example.com/retrieve`
   - Payload:
     ```json
     {
       "username": "user@example.com",
       "password": "user_password"
     }
     ```
   - Alternatively, client certificate authentication can be used to improve security.
5. **Decryption**: When needed, the client retrieves the encrypted private key, decrypts it locally using the user's passphrase, and uses it for S/MIME operations.

### 10. Security Considerations

- **Key Encryption**: Keys must be stored and transmitted in an encrypted form to prevent unauthorized access.
- **Authority Verification**: CAs must rigorously verify domain ownership to prevent issuance of fraudulent certificates.
- **Auto-expiry**: Regular key expiry and the availability of old keys ensure that keys are rotated frequently without losing access to old emails.
- **Private Key Security**: Private keys should never be transmitted in plain text and should always be protected by a strong passphrase.

### 11. IANA Considerations

This document does not require any new IANA registrations.

### 12. References

- **RFC 5751**: Secure/Multipurpose Internet Mail Extensions (S/MIME) Version 3.2 Message Specification.
- **RFC 5280**: Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile.

---

This draft includes the required details, using the expected language of IETF specifications. Please review this draft and provide feedback or suggest additional adjustments to ensure it fully meets your requirements.
