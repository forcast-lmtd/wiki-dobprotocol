---
icon: square-user
---

# KYC

<figure><img src="../../.gitbook/assets/How does KYC work in Dobprotocol (X) (1).jpg" alt=""><figcaption></figcaption></figure>

KYC (Know Your Customer) is the compliance framework that ensures Dobprotocol operates in line with global regulations and anti–money laundering standards.&#x20;

Instead of building a closed system where every step requires identity checks, Dobprotocol separates discovery from interaction. Anyone can explore the platform, review investment opportunities, and learn how revenue streams are tokenized, all without completing KYC.&#x20;

The process is only required at the moment of direct interaction with Dobprotocol applications (e.g., investing, operating, or withdrawing). We implement **SEP-0012**, a Stellar Ecosystem Proposal that standardizes how KYC data is shared across wallets, anchors, and services.&#x20;

With SEP-0012:

* Customers enter their KYC information once and can reuse it across multiple Stellar-based services.
* Both image and binary data are supported, alongside fields defined in SEP-9.
* Authentication is handled via SEP-10, ensuring secure validation.
* Data is interoperable with other SEPs (6, 24, 31), enabling seamless financial flows.
* Users retain control, with the ability to request complete data erasure at any time.

### KYC + Dobprotocol

By adopting SEP-0012, Dobprotocol ensures that compliance is a standard aligned with the broader Stellar ecosystem. This makes KYC interoperable, reusable, and user-centric, reducing friction while maintaining transparency and trust.

With KYC integrated into the platform’s architecture, Dobprotocol balances regulatory assurance with the openness and inclusivity that define Web3.

### **Learn more**

Read the official SEP-0012 specification on GitHub:\
[SEP-0012: KYC API](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0012.md?utm_source=chatgpt.com)
