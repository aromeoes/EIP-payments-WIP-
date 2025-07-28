---
eip: <to be assigned>
title: Multi-Chain Payment Request Standard
description: A standard for payment requests supporting multiple tokens across multiple blockchains with signature verification
author: @aromeoes
discussions-to: <URL>
status: Draft
type: Standards Track
category: ERC
created: 2025-07-28
---

## Abstract

This EIP defines a standard for payment requests that extends beyond single-chain single-token transactions. It enables merchants and applications to communicate payment requirements that can be fulfilled using multiple tokens across multiple blockchains, includes expiration mechanisms, and implements signature-based security for request verification.

## Motivation

Current payment request standards like EIP-681 have limitations:

1. **Single-chain focus**: Cannot express acceptance of the same asset across multiple chains
2. **No expiration mechanism**: Payment requests remain valid indefinitely
3. **Security concerns**: No cryptographic proof of the payment request origin
4. **Limited flexibility**: Cannot communicate acceptance of multiple tokens

As the cross-chain ecosystem grows, users hold assets across various networks. This standard enables seamless payment experiences where users can pay with their preferred token on their preferred chain, while merchants can cryptographically sign requests for verification.

## Specification

### URI Format

Payment requests use the following URI scheme:

```
payment:?data=<base64-encoded-payload>
```

### Payload Structure

The payload is a JSON object that MUST be Base64-encoded:

```json
{
  "options": [
    {
      "chainId": 1,
      "token": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      "amount": "1000000",
      "expiration": 19500000
    },
    {
      "chainId": 137,
      "token": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
      "amount": "1000000",
      "expiration": 55000000
    }
  ],
  "metadata": {},
  "signature": "0x..."
}
```

### Field Definitions

#### `options` (REQUIRED)
An array of payment options. Each option MUST contain:

- `chainId` (number, REQUIRED): The EIP-155 chain ID
- `token` (string, OPTIONAL): The token contract address. If omitted, indicates the native token of the specified chain
- `amount` (string, REQUIRED): The amount in the smallest unit of the token (e.g., wei for ETH, smallest unit for ERC-20)
- `expiration` (number, REQUIRED): The block number on the specified chain after which this option is no longer valid

#### `metadata` (OPTIONAL)
An object containing arbitrary key-value pairs for additional information. The structure and interpretation of metadata is left to implementers.

#### `signature` (REQUIRED)
An EIP-712 signature of the payment request data, signed by the payment requestor.

### EIP-712 Signature Specification (work in progress)
TODO: research signature format to increase security and enable reputation systems to be built on top
The signature MUST follow EIP-712 standard with the following structure:

### Implementation Example

A merchant accepting 1 USDC on either Ethereum or Polygon:

```javascript
const paymentRequest = {
  options: [
    {
      chainId: 1,
      token: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      amount: "1000000",
      expiration: 19500000
    },
    {
      chainId: 137,
      token: "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
      amount: "1000000",
      expiration: 55000000
    }
  ],
  metadata: {
    orderId: "12345",
    description: "Payment for order #12345"
  },
  signature: "0x..."
};

const encoded = btoa(JSON.stringify(paymentRequest));
const uri = `payment:?data=${encoded}`;
```

## Rationale

### Multi-chain multi-token Support
By allowing multiple tokens and multiple chains, wallets can simplify users payment experience by showing any of the available options to pay with where balance is .

### Per-chain Expiration
Block numbers are used for expiration as they provide more precise control than timestamps. Since block times vary by chain, each option has its own expiration.

### EIP-712 Signatures
EIP-712 provides a standard way to sign structured data that can be verified by wallets and shown to users in a human-readable format.

### Metadata Flexibility
Leaving metadata unstructured allows for evolution and implementation-specific features without requiring standard updates.

## Security Considerations

### Signature Verification
Wallets SHOULD implement signature verification to ensure payment requests originate from claimed sources. This MAY include:
- ENS resolution to verify domain ownership
- DNS TXT record verification
- Maintaining lists of known/verified requestors

### Replay Protection
The combination of amount, expiration, and metadata should make each payment request unique. Implementers SHOULD track fulfilled payments to prevent double-spending.

### Address Uniqueness
Requestors MUST make sure that each payment request has a unique address.

## Future Improvements

1. **Compression**: Future versions may specify compression schemes to reduce URI length
2. **Advanced routing**: Support for payment routing through specific contracts or protocols
3. **Partial payments**: Allow requests that can be fulfilled partially
4. **Recurring payments**: Extension for subscription-based payment requests

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
