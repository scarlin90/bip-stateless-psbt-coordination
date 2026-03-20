# BIP Draft: Blind Relay – Stateless Encrypted WebSocket Coordination for PSBTs

This repository hosts the draft Bitcoin Improvement Proposal (BIP) for a **stateless, zero-knowledge WebSocket relay** to enable real-time, end-to-end encrypted coordination of multi-signature Partially Signed Bitcoin Transactions (PSBTs).

The protocol provides the convenience of a centralized coordinator (instant updates, cross-device participation) without privacy leaks: no disk storage, no metadata persistence, no decryption keys on the server.

## Draft Document

The current BIP draft is in [`bip-draft.md`](./bip-draft.md) (Markdown for easy reading/editing).

- When submitting to the official [bitcoin/bips](https://github.com/bitcoin/bips) repo, it will be converted to MediaWiki format as `bip-XXXX.mediawiki`.
- Preamble uses placeholder `BIP: ?` — number assigned by BIP Editors upon merge.

## Reference Implementation

A production reference impl (stateless relay server + client interface) is available here:  
- Live demo: https://signingroom.io  
- Source code: https://github.com/scarlin90/signingroom (AGPLv3)

The protocol itself (as defined in this draft) is licensed permissively (CC0-1.0 / BSD-3-Clause).

## Status

- **Status**: Early Draft  
- **Seeking**: Feedback on protocol design, security model, URI scheme, encryption details, and implementability.  
- Next step: Post to bitcoin-dev mailing list for discussion.

## Contributing

Issues, PRs, and comments welcome—especially on the spec wording, security considerations, or test vectors.

See [Issues](https://github.com/scarlin90/bip-stateless-psbt-coordination/issues) or open a discussion.

## License

- This BIP draft: CC0-1.0 (public domain) + BSD-3-Clause for any code snippets  
- Repository content: Same as above unless noted.

Feedback welcome—let's make multi-sig coordination private and frictionless!
