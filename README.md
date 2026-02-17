# ERC-8004 Best Practices Skill

A [Claude Code](https://claude.ai/claude-code) skill for building ERC-8004 compliant agents. Covers the official [EIP-8004 Trustless Agents](https://eips.ethereum.org/EIPS/eip-8004) specification and the [8004scan community best practices](https://best-practices.8004scan.io/) for offchain data formats.

## Install

**Claude Code CLI:**
```bash
claude /install-skill https://github.com/0xSardius/erc-8004-best-practices-skill
```

**npx:**
```bash
npx @anthropic-ai/claude-code /install-skill https://github.com/0xSardius/erc-8004-best-practices-skill
```

## What's Covered

### Core Specification
- **Identity Registry** — ERC-721 NFT-based agent registration, `AgentURI` metadata, onchain metadata key-value pairs, `setAgentWallet` with EIP-712/ERC-1271 signatures
- **Reputation Registry** — Feedback with `value`/`valueDecimals` (v2.0), revocation, response appending, `getSummary` aggregation
- **Validation Registry** — Validation requests/responses, zkML (Groth16/PLONK/STARK), TEE attestation, crypto-economic proofs

### Agent Metadata (AgentURI)
- Complete JSON schema with all field definitions
- 8 service types: MCP, A2A, OASF, agentWallet, ENS, DID, web, email
- URI format recommendations ranked by immutability (Data URI > IPFS/Arweave > HTTPS)
- Multi-chain deployment workflow
- Bidirectional verification pattern
- `endpoints` → `services` field migration

### Feedback System (v2.0)
- `value`/`valueDecimals` fixed-point arithmetic (like ERC-20 tokens)
- Standard patterns: percentages, binary flags, financial metrics, latency, ratings
- Domain-specific patterns for AI/LLM, DeFi/Trading, and DePIN agents
- Payment proofs (x402), attachments, protocol-specific fields (A2A, MCP, OASF)

### Implementation
- 5-level metadata validation system (Syntax → Schema → Endpoints → Semantic → Status)
- Complete error codes reference (EA, WA, IA, EF, WF series)
- Agent metadata parser architecture with IPFS gateway fallback
- Data URI compression extension (zstd, gzip, brotli, lz4) with gas benchmarks
- Caching strategies for Data URI, IPFS, and HTTP
- Migration guides: pre-v1 → v1.x → v2.0

### Conventions
- CAIP-2 / CAIP-10 address formats with chain ID reference table
- ISO 8601 UTC timestamps
- URI scheme requirements (https, ipfs, ar, data)
- Keccak-256 content hashing
- Unknown field handling and custom extension patterns

## When It Triggers

The skill activates when your prompt mentions: `ERC-8004`, `8004`, `agent registry`, `agent metadata`, `AgentURI`, `trustless agents`, `agent reputation`, `agent feedback`, `agent validation`, `8004scan`, or any task involving onchain agent identity, reputation, or validation registries.

## Sources

- [EIP-8004 Official Specification](https://eips.ethereum.org/EIPS/eip-8004)
- [8004scan Best Practices](https://best-practices.8004scan.io/)
- [8004scan GitHub](https://github.com/alt-research/8004scan-best-practices)

## License

MIT
