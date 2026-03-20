```text
BIP: ?
  Layer: Applications
  Title: Blind Relay: Stateless Encrypted WebSocket Coordination for PSBTs
  Author: Sean Carlin <seancarlin90@googlemail.com>
  Comments-URI: https://github.com/scarlin90/bip-stateless-psbt-coordination/issues
  Status: Draft
  Type: Standards Track
  Created: 2026-03-20
  License: CC0-1.0
  License-Code: BSD-3-Clause
  Requires: 174
  ```

## Abstract

This document proposes a standard for the trustless coordination of multi-signature Partially Signed Bitcoin Transactions (PSBTs) utilizing an ephemeral, stateless End-to-End Encrypted (E2EE) WebSocket relay. 

Currently, coordinating N-of-M Bitcoin transactions suffers from a Coordination Vulnerability. Users must either pass PSBTs manually out-of-band or rely on stateful, centralized servers that inherently leak metadata, signer identities, and quorum relationships. 

To resolve this, this standard introduces a Blind Relay architecture. By utilizing AES-GCM-256 encryption with decryption keys stored exclusively in client-side URL fragments or passed out of band, the protocol ensures that sensitive transaction data is never exposed to the server. The relay facilitates real-time, synchronous updates via WebSockets but maintains a strict zero-disk-state and zero-knowledge stance. It routes encrypted payloads strictly in RAM for a max 24 hours or until the signing event has completed. Upon destruction, it retains no transaction content or persistent metadata. This provides the real-time user experience of a centralized coordinator while preserving the security and privacy guarantees of an air-gapped workflow.

## Motivation

Bitcoin multi-signature (N-of-M) coordination currently suffers from a Coordination Gap. While the security of signing devices has evolved rapidly, the process of passing Partially Signed Bitcoin Transactions (PSBTs) between signers remains a critical friction point for all users.

Existing solutions force users into a binary choice, both of which are inadequate:
1. Manual, Out-of-Band Transfers (USB drives, emails, secure messaging): Preserves privacy but introduces heavy operational friction, latency, and human error, making cross-device participation difficult.
2. Stateful Coordination Servers: Centralized databases provide a seamless User Experience (UX) but act as privacy honeypots. They log sensitive metadata, IP addresses, quorum relationships, and often store PSBTs on disk. Furthermore, some platforms hold extended public keys (xpubs) to enforce vendor lock-in, or drift toward quasi-custodial models that inherently compromise the trustless security guarantees of the network.

The Bitcoin community has previously recognized the need for standardized, encrypted coordination. In 2023, BIP 77 (Async Payjoin) successfully introduced a directory-based standard for 2-party transactions, utilizing out-of-band URI sharing to establish encrypted routing. However, BIP 77 relies on an asynchronous store and forward architecture. The directory server must write encrypted payloads to disk until the receiver comes online. While sufficient for 1-to-1 Payjoin, storing multi-party payloads on disk introduces Directory Denial of Service (DoS) vectors and latency for iterative, real-time N-of-M signing rounds.

To bridge this gap, this protocol provides a Digital Airgap. By utilizing full-duplex WebSockets to stream PSBTs instantly between peers, it provides the real-time convenience of a centralized server without many of the associated privacy leaks.

The motivation for this protocol is to establish a standard that:
* Eliminates Metadata Persistence: Utilizing stateless workers ensures no transaction data or quorum associations are ever written to disk.
* Enforces Zero-Knowledge: The server cannot decrypt the data it relays. Decryption keys are shared strictly out-of-band or via URL fragments
* Streamlines UX: Provides a web-standard URI for instant, cross-device participation, allowing hardware wallets and software clients to coordinate seamlessly.

## Rationale

### Why WebSockets over HTTP Polling?
While BIP 77 utilizes HTTP polling for its asynchronous directory, polling introduces significant overhead and latency. In a complex N-of-M multi-signature ceremony where participants may be actively filtering dozens of inputs/outputs or passing a PSBT through 5-7 signing rounds , sub-second latency is required and ensures the coordination is efficient for all users. Full-duplex WebSockets allow the relay to push payloads instantly to all connected clients without the overhead of continuous HTTP GET requests.

### Why a Central Relay instead of WebRTC (P2P)?
An alternative design considered was utilizing WebRTC for true peer-to-peer (P2P) coordination. However, WebRTC inherently exposes the IP addresses of the participants to one another to establish the connection. A Blind Relay acts as a protective proxy, masking the peers' IP addresses from each other while remaining ignorant of the payload.

### Why URL Fragments for Key Distribution?
The protocol relies on URL fragments (#) for distributing the AES-GCM-256 decryption key. By internet standard (RFC 3986), URL fragments are evaluated strictly client-side by the browser or application and are not transmitted to the server during an HTTP or WebSocket handshake. This provides a mathematically guaranteed, frictionless mechanism to distribute the room location and the decryption key in a single string, without ever exposing the key to the relay for convenience but additionally the room and key can be distributed separately. 

### Why Ephemeral RAM State over Disk Storage?
Unlike the persistent store-and-forward model of BIP 77, which writes encrypted payloads to a database, this protocol utilizes an ephemeral, RAM-only architecture. Participants are not strictly required to be online simultaneously. The relay holds the encrypted PSBT state in memory for a maximum of 24 hours. If a signer disconnects, they can seamlessly rejoin the room using the shared URI to establish a new WebSocket session and retrieve the current payload. Storing encrypted PSBTs on disk inherently introduces metadata storage risks. By enforcing a strict RAM-only policy where rooms self-destruct after 24 hours, upon explicit completion, or when the user closes the room, the relay removes many of the legal and security liabilities of persisting encrypted financial data.

### Addressing Client Environment Concerns
A common objection to web-based PSBT coordination is the risk of malicious browser extensions or compromised web environments. It is important to note that this BIP standardizes the transport protocol, not the client environment. While reference implementations may use Progressive Web Apps (PWAs), this WebSocket standard is designed to be natively integrated directly into desktop hardware wallet companions or mobile applications, entirely bypassing the browser environment if desired for maximum opsec.

## Copyright

This document is dual licensed as BSD 3-clause, and Creative Commons CC0 1.0 Universal.

## Backwards Compatibility

This standard does not modify the underlying PSBT format BIP 174 / BIP 370. It strictly standardizes the transport and coordination layer, ensuring full compatibility with existing signing hardware and wallet architectures.

## Reference Implementation

A fully functional, production-ready reference implementation of this stateless WebSocket standard is currently maintained under AGPLv3.
* **Client Interface:** https://signingroom.io
* **Source Code:** https://github.com/scarlin90/signingroom
