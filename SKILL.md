# ERC-8004 Best Practices Skill

Build ERC-8004 compliant agents with correct metadata, feedback, and validation patterns. This skill covers the official EIP-8004 specification (onchain smart contracts) and the community-maintained best practices for offchain data formats.

## Trigger

Use when building, registering, or interacting with ERC-8004 agents. Triggers on: "ERC-8004", "8004", "agent registry", "agent metadata", "AgentURI", "trustless agents", "agent reputation", "agent feedback", "agent validation", "8004scan", or any task involving onchain agent identity, reputation, or validation registries.

---

## ERC-8004 Overview

ERC-8004 ("Trustless Agents") uses blockchains to discover, choose, and interact with agents across organizational boundaries without pre-existing trust. It defines three lightweight registries deployable on L2 or Mainnet:

1. **Identity Registry** - ERC-721 NFT-based agent registration with metadata URIs
2. **Reputation Registry** - Onchain feedback with flexible value/decimals system
3. **Validation Registry** - Proofs and attestations (zkML, TEE, crypto-economic)

### Relationship: Official Spec vs Best Practices

| Aspect | EIP-8004 (Official) | Best Practices (8004scan) |
|--------|---------------------|---------------------------|
| Defines | Smart contract interfaces | Offchain data patterns |
| Scope | Onchain behavior | Offchain metadata schemas |
| Authority | Ethereum EIP process | Community recommendations |
| Adoption | MUST for compliance | SHOULD for interoperability |

Where conflicts exist, EIP-8004 takes precedence.

---

## Conventions

### RFC 2119 Keywords
- **MUST/REQUIRED/SHALL** - Absolute requirements
- **SHOULD/RECOMMENDED** - Valid exceptions possible
- **MAY/OPTIONAL** - Completely optional

### Address Formats

**CAIP-2 (Blockchain ID):** `namespace:chainId`
```
eip155:1          - Ethereum Mainnet
eip155:11155111   - Sepolia Testnet
eip155:8453       - Base Mainnet
eip155:84532      - Base Sepolia
eip155:137        - Polygon PoS
eip155:42161      - Arbitrum One
eip155:10         - Optimism
```

**CAIP-10 (Account Address):** `namespace:chainId:address`
```
eip155:1:0x8004a6090Cd10A7288092483047B097295Fb8847
eip155:84532:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
```

Addresses MUST be checksummed (EIP-55) or lowercase.

### Timestamps
All timestamps MUST use ISO 8601 UTC: `YYYY-MM-DDTHH:mm:ssZ`
```
2025-12-21T14:30:00Z     # valid
2025-12-21T14:30:00.123Z # valid (milliseconds optional)
2025-12-21 14:30:00      # INVALID (missing T)
2025-12-21T14:30:00+08:00 # INVALID (non-UTC)
```

### URI Schemes
| Scheme | Example | Use Case |
|--------|---------|----------|
| `https://` | `https://example.com/file.json` | Production APIs |
| `ipfs://` | `ipfs://bafkreiabc...` | Immutable decentralized storage |
| `ar://` | `ar://abc123...` | Permanent storage (Arweave) |
| `data:` | `data:application/json;base64,ew...` | Inline data embedding |

- REQUIRED: `https://`
- RECOMMENDED: `ipfs://`, `data:`
- OPTIONAL: `ar://`

Reject relative URIs and URIs without schemes.

### Content Hashing
Algorithm: **Keccak-256**. For JSON: serialize with no whitespace, encode UTF-8, compute hash, represent as `0x`-prefixed hex.

### Unknown Field Handling
- MUST NOT reject documents with unknown fields
- SHOULD preserve unknown fields when re-serializing
- Custom extensions use `x-<domain>-<fieldname>` prefix

---

## Identity Registry (Agent Registration)

### Onchain Functions

```solidity
// Register agent
function register(string agentURI, MetadataEntry[] calldata metadata) external returns (uint256 agentId)
function register(string agentURI) external returns (uint256 agentId)
function register() external returns (uint256 agentId)

// Update URI
function setAgentURI(uint256 agentId, string calldata newURI) external

// Onchain metadata
function getMetadata(uint256 agentId, string metadataKey) external view returns (string)
function setMetadata(uint256 agentId, string metadataKey, string metadataValue) external

// Agent wallet (reserved key, requires EIP-712 or ERC-1271 signature)
function setAgentWallet(uint256 agentId, address newWallet, uint256 deadline, bytes calldata signature) external
```

### AgentURI Metadata (Offchain JSON)

Complete registration file structure:

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My Agent",
  "description": "What my agent does",
  "image": "https://example.com/logo.png",
  "services": [
    {
      "name": "web",
      "endpoint": "https://web.agentxyz.com/"
    },
    {
      "name": "A2A",
      "endpoint": "https://agent.example/.well-known/agent-card.json",
      "version": "0.3.0"
    },
    {
      "name": "MCP",
      "endpoint": "https://mcp.agent.eth/",
      "capabilities": [],
      "version": "2025-06-18"
    },
    {
      "name": "OASF",
      "endpoint": "https://github.com/agntcy/oasf/",
      "version": "0.8.0",
      "skills": ["analytical_skills/data_analysis/blockchain_analysis"],
      "domains": ["technology/blockchain"]
    },
    {
      "name": "agentWallet",
      "endpoint": "eip155:1:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7"
    },
    {
      "name": "ENS",
      "endpoint": "myagent.eth",
      "version": "v1"
    },
    {
      "name": "DID",
      "endpoint": "did:ethr:0xfoobar",
      "version": "v1"
    },
    {
      "name": "email",
      "endpoint": "mail@myagent.com"
    }
  ],
  "x402Support": false,
  "active": true,
  "registrations": [
    {
      "agentId": 22,
      "agentRegistry": "eip155:1:0x742..."
    }
  ],
  "supportedTrust": ["reputation", "crypto-economic", "tee-attestation"],
  "updatedAt": 1763112328
}
```

### Field Reference

#### SHOULD Fields (Recommended)

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | MUST be `"https://eips.ethereum.org/EIPS/eip-8004#registration-v1"` |
| `name` | string | Agent identifier, 3-200 chars, avoid generic names like "Agent #123" |
| `description` | string | Capabilities, interaction methods, pricing. 50-500 chars recommended |
| `image` | string | Avatar/logo URI. 512x512px min, 1:1 aspect ratio. PNG/SVG/WebP/JPG |
| `services` | array | Communication endpoints (at least one if agent is interactive) |

#### MAY Fields (Optional)

| Field | Type | Description |
|-------|------|-------------|
| `x402Support` | boolean | Indicates x402 payment protocol support |
| `active` | boolean | Production readiness (default: false) |
| `registrations` | array | Links to onchain NFT identity for bidirectional verification |
| `supportedTrust` | array | Trust models: `"reputation"`, `"crypto-economic"`, `"tee-attestation"`, `"social-graph"` |
| `updatedAt` | number | Unix timestamp for metadata freshness |

### Service Types

#### MCP (Model Context Protocol)
```json
{
  "name": "MCP",
  "endpoint": "https://mcp.agent.example/",
  "version": "2025-06-18",
  "capabilities": [],
  "mcpTools": ["tool1", "tool2"],
  "mcpPrompts": ["prompt1"],
  "mcpResources": ["resource1"]
}
```
- Version format: date-based `YYYY-MM-DD` (e.g., `"2025-06-18"`, `"2025-11-25"`, `"2026-01-29"`)
- `capabilities` is an array (changed from object in v1.1)

#### A2A (Agent-to-Agent Protocol)
```json
{
  "name": "A2A",
  "endpoint": "https://agent.example/.well-known/agent-card.json",
  "version": "0.3.0",
  "a2aSkills": ["analytical_skills/data_analysis"]
}
```
- SHOULD use `/.well-known/agent-card.json` path
- Both `"0.3.0"` and `"0.30"` accepted

#### OASF (Open Agent Skills Framework)
```json
{
  "name": "OASF",
  "endpoint": "https://github.com/agntcy/oasf/",
  "version": "0.8.0",
  "skills": ["analytical_skills/data_analysis/blockchain_analysis"],
  "domains": ["technology/blockchain"]
}
```
- At least one of `skills` or `domains` REQUIRED
- Skills format: `category/subcategory/specific_skill`
- Domains format: `domain_name` (e.g., `finance_and_business/finance`)

#### agentWallet
```json
{
  "name": "agentWallet",
  "endpoint": "eip155:1:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7"
}
```
- CAIP-10 format required
- Multiple wallets on different chains allowed
- Onchain `agentWallet` is authoritative; offchain is discovery hint only

### Field Name Migration: `endpoints` to `services`

As of January 2026, the official EIP-8004 spec renamed `endpoints` to `services`:
- New implementations SHOULD use `services`
- Parsers SHOULD support both field names
- If both exist, `services` takes precedence
- Individual service objects still use `endpoint` (singular) for the URL property

### URI Format Recommendations (by Immutability)

1. **Data URI** - Perfect immutability, high gas cost (70K-340K gas for 3-20KB)
2. **IPFS/Arweave** - Strong immutability, low gas cost (40-70K gas). Best balance for production
3. **HTTP/HTTPS** - No immutability, low gas. Development/testing only

### Bidirectional Verification Pattern

```
NFT (onchain) → agentURI → Metadata (offchain) → registrations → NFT (onchain) ✅
```

The `registrations` array creates a cryptographic bidirectional link between onchain identity and offchain metadata.

### Multi-Chain Deployment

1. **Phase 1**: Register on all chains (without URI) to collect token IDs
2. **Phase 2**: Create unified metadata JSON with all registrations
3. **Phase 3**: Call `setAgentURI()` on each registry pointing to same file
4. Use CAIP-10 format for all chain identifiers

### Endpoint Domain Verification (Optional)

Prove control of HTTPS domains by publishing:
`https://{service-domain}/.well-known/agent-registration.json`
containing a matching `registrations` list.

---

## Reputation Registry (Feedback)

### Onchain Functions

```solidity
// Submit feedback
function giveFeedback(
  uint256 agentId,
  int128 value,
  uint8 valueDecimals,
  string calldata tag1,
  string calldata tag2,
  string calldata endpoint,
  string calldata feedbackURI,
  bytes32 feedbackHash
) external

// Revoke feedback
function revokeFeedback(uint256 agentId, uint64 feedbackIndex) external

// Append response (by agent owner)
function appendResponse(
  uint256 agentId,
  address clientAddress,
  uint64 feedbackIndex,
  string calldata responseURI,
  bytes32 responseHash
) external

// Read functions
function getSummary(uint256 agentId, address[] calldata clientAddresses, string tag1, string tag2)
  external view returns (uint64 count, int128 averageValue, uint8 valueDecimals)

function readFeedback(uint256 agentId, address clientAddress, uint64 feedbackIndex)
  external view returns (int128 value, uint8 valueDecimals, string tag1, string tag2, bool isRevoked)

function readAllFeedback(uint256 agentId, address[] calldata clientAddresses, string tag1, string tag2, bool includeRevoked)
  external view returns (address[] memory, uint64[] memory, int128[] memory, uint8[] memory, string[] memory, string[] memory, bool[] memory)

function getClients(uint256 agentId) external view returns (address[] memory)
function getLastIndex(uint256 agentId, address clientAddress) external view returns (uint64)
```

### Constraints
- AgentId must be validly registered
- ValueDecimals: 0-18
- Submitter cannot be agent owner or approved operator
- `tag1`, `tag2`, `endpoint`, `feedbackURI`, `feedbackHash` are optional
- FeedbackHash only required for non-content-addressed URIs

### Value/ValueDecimals System (v2.0)

The actual rating is: `value / 10^valueDecimals`

| tag1 | Measurement | Example | value | decimals |
|------|-------------|---------|-------|----------|
| `starred` | Quality (0-100) | 87/100 | 87 | 0 |
| `reachable` | Binary | true | 1 | 0 |
| `ownerVerified` | Binary | true | 1 | 0 |
| `uptime` | Percentage | 99.77% | 9977 | 2 |
| `successRate` | Percentage | 89% | 89 | 0 |
| `responseTime` | Milliseconds | 560ms | 560 | 0 |
| `blocktimeFreshness` | Blocks | 4 blocks | 4 | 0 |
| `revenues` | Currency | $560 | 560 | 0 |
| `tradingYield` | Yield | -3.2% | -32 | 1 |

#### Domain-Specific Patterns

**AI/LLM Agents:**
- Tokens per second: tag1=`tps`, value=455, decimals=1 (45.5 tok/s)
- Cost per 1M tokens: tag1=`costPer1M`, value=500000, decimals=6 ($0.50)

**DeFi/Trading:**
- Sharpe ratio: tag1=`sharpeRatio`, value=255, decimals=2 (2.55)
- Max drawdown: tag1=`maxDrawdown`, value=-1540, decimals=2 (-15.40%)

**DePIN:**
- Latitude: tag1=`latitude`, value=40712800, decimals=6 (40.712800)
- Signal strength: tag1=`signalStrength`, value=-75, decimals=0 (-75 dBm)

### Offchain Feedback JSON

```json
{
  "agentRegistry": "eip155:84532:0x8004C269D0A5647E51E121FeB226200ECE932d55",
  "agentId": 22,
  "clientAddress": "eip155:84532:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "createdAt": "2026-01-20T12:00:00Z",
  "value": "4.75",
  "valueDecimals": 2,
  "tag1": "defi",
  "tag2": "analytics",
  "endpoint": "https://api.agent.example.com/v2",
  "reasoning": "Excellent DeFi analytics service.",
  "skill": "analytical_skills/data_analysis/blockchain_analysis",
  "domain": "finance_and_business/finance",
  "context": "Portfolio optimization for DeFi positions",
  "capability": "tools",
  "name": "blockchain_analyzer",
  "proofOfPayment": {
    "fromAddress": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "toAddress": "0x8004C269D0A5647E51E121FeB226200ECE932d55",
    "chainId": "84532",
    "txHash": "0xabc123...",
    "amount": "100000000000000000",
    "currency": "ETH",
    "timestamp": "2026-01-20T11:45:00Z"
  },
  "attachments": [
    {
      "name": "Analysis Report",
      "uri": "ipfs://QmXxx...",
      "mimeType": "application/pdf",
      "size": 250000,
      "description": "Full portfolio analysis with recommendations",
      "uploadedAt": "2026-01-20T11:50:00Z"
    }
  ]
}
```

### Required Offchain Fields
| Field | Type | Format |
|-------|------|--------|
| `agentRegistry` | string | CAIP-10 |
| `agentId` | number | uint128 |
| `clientAddress` | string | CAIP-10 |
| `createdAt` | string | ISO 8601 UTC |
| `value` | string | BigDecimal (int128) |
| `valueDecimals` | number | 0-18 (uint8) |

### Protocol-Specific Fields
- **A2A/OASF**: `skill`, `domain`, `context`, `task`
- **MCP**: `capability` (one of: `prompts`, `resources`, `tools`, `completions`), `name`

### Feedback Best Practices

1. Always provide `reasoning` with human-readable context
2. Include evidence via `attachments` for disputes
3. Use protocol-specific fields for discoverability
4. Include `proofOfPayment` for x402 interactions
5. Use CAIP formats for all addresses
6. Include `endpoint` to track which service was used
7. Use `camelCase` for tag1 values
8. Never include private keys, API keys, PII, or confidential data

### Scoring Guide (5-Star)
- **5**: Outstanding, exceeded expectations
- **4**: Good, met expectations
- **3**: Acceptable, neutral
- **2**: Below average, significant problems
- **1**: Poor, failed to deliver
- **0**: Very poor, non-functional
- **< 0**: Dispute/scam/malicious behavior

---

## Validation Registry

> **WARNING**: The Validation Registry specification is UNSTABLE and PENDING FINALIZATION. Do not use in production.

### Onchain Functions

```solidity
// Request validation
function validationRequest(
  address validatorAddress,
  uint256 agentId,
  string requestURI,
  bytes32 requestHash
) external

// Respond to validation
function validationResponse(
  bytes32 requestHash,
  uint8 response,
  string responseURI,
  bytes32 responseHash,
  string tag
) external

// Read functions
function getValidationStatus(bytes32 requestHash)
  external view returns (address, uint256, uint8, string, uint256)

function getSummary(uint256 agentId, address[] calldata validatorAddresses, string tag)
  external view returns (uint64 count, uint8 averageResponse)

function getAgentValidations(uint256 agentId) external view returns (bytes32[] memory)
function getValidatorRequests(address validatorAddress) external view returns (bytes32[] memory)
```

### Validation Types
- `crypto-economic` - Stake-secured re-execution
- `tee-attestation` - TEE oracle proofs (Intel SGX, AMD SEV, AWS Nitro)
- `zkml-proof` - Zero-knowledge ML verification (Groth16, PLONK, STARK)

### Response Values
- `response`: 0-100 (uint8). Binary: 0=fail, 100=pass. Or spectrum.
- Multiple responses allowed for same `requestHash` (progressive validation)

### Validation Request JSON

```json
{
  "agentRegistry": "eip155:84532:0x8004C269D0A5647E51E121FeB226200ECE932d55",
  "agentId": 56,
  "validatorAddress": "eip155:84532:0x4baf957830423f36978039e32d95ee958aa0560c",
  "createdAt": "2025-12-20T05:00:38Z",
  "validationType": "zkml-proof",
  "reasoning": "Validate agent rebalancing logic",
  "deadline": "2025-12-21T05:00:38Z",
  "zkmlRequest": {
    "circuit": "zyfi-rebalancing-validation",
    "publicInputs": ["4190804445", "1"],
    "verifierAddress": "0x4baf957830423f36978039e32d95ee958aa0560c"
  }
}
```

### Validation Response JSON

```json
{
  "requestHash": "0x5c1a08c66c0489030f05990a83534280ee08da7cd944dc353bf4b661c5c96657",
  "validatorAddress": "eip155:84532:0x4baf957830423f36978039e32d95ee958aa0560c",
  "agentId": 56,
  "response": 100,
  "createdAt": "2025-12-20T06:00:00Z",
  "tag": "zkml-proof",
  "result": "passed",
  "reasoning": "Agent passed all validation tests",
  "confidence": 0.95,
  "zkmlProof": {
    "proof": {
      "pi_a": ["...", "...", "1"],
      "pi_b": [["...", "..."], ["...", "..."], ["1", "0"]],
      "pi_c": ["...", "...", "1"],
      "protocol": "groth16",
      "curve": "bn128"
    },
    "publicSignals": ["4190804445", "1"],
    "verifiedAt": "2025-12-20T05:45:00Z"
  }
}
```

### Validation Best Practices

- Always include clear `result` (`approved`, `rejected`, `conditional`), detailed `reasoning`, and supporting `attachments`
- Match validation type to request
- Use `confidence` score (0.0-1.0)
- Include test environment details for reproducibility
- Wrap custom proof formats (Groth16) in ERC-8004 standard envelope
- Always use CAIP-10 format for addresses

---

## Attachment Pattern

Reusable across feedback and validation records.

```typescript
interface AttachmentFile {
  name: string;        // Required: 1-200 chars
  uri: string;         // Required: valid URI scheme
  mimeType: string;    // Required: IANA MIME type
  size: number;        // Required: bytes, >= 0
  description?: string; // Optional: 0-1000 chars
  uploadedAt?: string;  // Optional: ISO 8601 UTC
}
```

### Best Practices
- Total size < 10 MB per record
- Prefer IPFS/Arweave for permanence
- Never attach private keys, PII, or executables
- Use descriptive filenames
- Test accessibility post-upload

---

## Data URI Compression (Optional Extension)

Reduces gas costs for onchain Data URI storage by 35-67%.

### Format
```
data:application/json;enc=<algorithm>[;level=<level>];base64,<compressedData>
```

### Algorithms
| Algorithm | Ratio | Speed | Recommendation |
|-----------|-------|-------|----------------|
| **zstd** (recommended) | 35-59% | Fast | Best balance |
| **br** (Brotli) | 48-67% | Slow | Highest ratio |
| **gzip** | 35-60% | Fast | Best compatibility |
| **lz4** | 15-45% | Fastest | Speed priority |

### When to Compress
- < 500 bytes: Don't compress
- 500B-2KB: Optional (~2K-6K gas saved)
- 2-5KB: Recommended (~31K-35K gas saved)
- \> 5KB: Strongly recommended (35K+ gas saved)

### Security
- MUST enforce 100KB decompression size limit (zip bomb protection)
- MUST whitelist algorithms (zstd, gzip, br, lz4)
- SHOULD decompress in background workers

### Example (Python, Zstd)
```python
import zstandard as zstd
import base64, json

json_bytes = json.dumps(metadata, separators=(',', ':')).encode('utf-8')
compressed = zstd.ZstdCompressor(level=15).compress(json_bytes)
encoded = base64.b64encode(compressed).decode('ascii')
uri = f"data:application/json;enc=zstd;level=15;base64,{encoded}"
```

---

## Agent Metadata Parsing

### URI Format Detection Order (by frequency)
1. Data URIs (36% of agents) - fastest, no network needed
2. IPFS URIs (51%) - requires gateway fetch
3. HTTP/HTTPS (22%) - requires network fetch
4. Plain JSON fallback - edge case

### IPFS Gateway Fallback Strategy
```
https://ipfs.io/ipfs/{CID}
https://cloudflare-ipfs.com/ipfs/{CID}
https://gateway.pinata.cloud/ipfs/{CID}
```
5-second timeout per gateway. ~95% success rate with fallback.

### 5-Level Validation System
1. **Syntax** - URI format, JSON parsing, base64 decoding
2. **Schema** - Required/recommended fields presence
3. **Endpoints** - Protocol-specific field validation
4. **Semantic** - Trust model values, cross-field consistency, CAIP format
5. **Status** - Production readiness flags

### Common Parser Edge Cases
- **ChaosChain pattern**: `data:application/json;base64,{...}` where base64 claim contains plain JSON. Detect `{` at start, skip decoding.
- **Null agentId**: Expected for first-time deployments (~30% of registrations). Accept as info, not error.
- **Empty services**: Agent unreachable but metadata valid (~15% of agents). Warn but don't fail.
- **`endpoints` vs `services`**: Support both field names, prefer `services`.

### Caching Strategy
- Data URIs: In-memory LRU (immutable, cache indefinitely)
- IPFS: Redis with 1-hour TTL
- HTTP: Redis with 5-minute TTL

---

## Error Codes Reference

### Code Format: `<Severity><Object><Number>`
- Severity: E (Error), W (Warning), I (Info)
- Object: A (Agent), F (Feedback)

### Critical Errors (Parse Fails)

| Code | Description |
|------|-------------|
| EA001 | Empty or invalid URI |
| EA002 | Invalid JSON syntax |
| EA003 | Invalid base64 encoding |
| EA004 | Decompression security error (zip bomb, >100KB) |
| EA005 | Decompression failed (algorithm mismatch) |
| EA006 | Unsupported URI format |
| EA007 | IPFS fetch failed (all gateways exhausted) |
| EA008 | HTTP fetch failed (timeout, 404, SSL) |
| EA009 | Arweave fetch failed |
| EA010 | Non-dict metadata root (must be JSON object) |

### Key Warnings (Parse Succeeds)

| Code | Description |
|------|-------------|
| WA001 | Missing `type` field |
| WA002 | Invalid `type` value |
| WA003 | Missing `name` field |
| WA004 | Missing `description` field |
| WA005 | Invalid image URL format |
| WA013 | Invalid agentRegistry format (should be CAIP-2) |
| WA020 | Typo: `endpoint` (singular) instead of `endpoints`/`services` |
| WA030 | agentWallet not in CAIP-10 format |
| WA031 | Using legacy `endpoints` field (recommend `services`) |
| WA050 | base64 URI contains plain JSON |
| WA070 | Metadata doesn't match onchain agentHash |
| WA080 | Conflict between onchain and offchain metadata |

### Info Messages (Advisory)

| Code | Description |
|------|-------------|
| IA001 | Missing image (displays default placeholder) |
| IA002 | Missing services array (agent unreachable) |
| IA006 | Missing agentId (expected for first-time deployments) |
| IA009 | Unknown trust model (may be future extension) |
| IA020 | MCP endpoint missing version |
| IA024 | A2A endpoint should use `.well-known/agent-card.json` path |
| IA040 | HTTP/HTTPS URI is not content-addressed |

---

## Migration Guides

### v1.x to v2.0 (Feedback)

**Breaking change**: `score` (uint8, 0-100) replaced by `value` (int128) + `valueDecimals` (uint8, 0-18)

Score-to-value conversion (5-star scale):
```javascript
// Simple: keep as percentage
value = score;      // e.g., 95
valueDecimals = 0;

// Convert to 5-star
value = score / 20; // e.g., 4.75
valueDecimals = 0;
```

Update contract calls:
```solidity
// OLD: submitFeedback(agentId, score, tag1, tag2, endpoint, feedbackURI, feedbackHash)
// NEW: submitFeedback(agentId, value, valueDecimals, tag1, tag2, endpoint, feedbackURI, feedbackHash)
```

### pre-v1 to v1.x (Feedback)

Key changes:
1. **feedbackAuth removed** - Anyone can submit feedback directly without pre-authorization
2. **Tags: bytes32 to string** - Use plain strings instead of hex-encoded bytes32
3. **feedbackUri to feedbackURI** - Capitalization fix
4. **endpoint field added** - Optional string for tracking which agent endpoint was used
5. **feedbackIndex emitted in event** - No longer calculated by indexers
6. **Addresses use CAIP format** - `eip155:chainId:0x...` instead of raw `0x...`

---

## Quick Start Examples

### Register an Agent
```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My Agent",
  "description": "What my agent does",
  "image": "https://example.com/logo.png",
  "services": [
    {
      "name": "MCP",
      "endpoint": "https://mcp.myagent.com/",
      "version": "2025-06-18"
    },
    {
      "name": "A2A",
      "endpoint": "https://myagent.com/.well-known/agent-card.json",
      "version": "0.3.0"
    }
  ],
  "registrations": [
    {
      "agentId": null,
      "agentRegistry": "eip155:84532:0x8004C269D0A5647E51E121FeB226200ECE932d55"
    }
  ],
  "active": true,
  "supportedTrust": ["reputation"]
}
```

### Submit Feedback
```json
{
  "agentRegistry": "eip155:84532:0x8004C269D0A5647E51E121FeB226200ECE932d55",
  "agentId": 42,
  "clientAddress": "eip155:84532:0x742d35Cc...",
  "createdAt": "2026-01-20T12:00:00Z",
  "value": "4",
  "valueDecimals": 0,
  "tag1": "quality",
  "reasoning": "Excellent service, fast responses"
}
```

### Request Validation
```json
{
  "agentRegistry": "eip155:84532:0x8004C269D0A5647E51E121FeB226200ECE932d55",
  "agentId": 56,
  "validatorAddress": "eip155:84532:0x4baf957...",
  "createdAt": "2025-12-20T05:00:38Z",
  "validationType": "zkml-proof",
  "reasoning": "Verify model execution"
}
```

---

## Glossary

| Term | Definition |
|------|------------|
| A2A | Agent-to-Agent Protocol (a2a.to) |
| AgentURI | Offchain metadata URI referenced by ERC-8004 NFTs |
| CAIP-2 | Blockchain ID format: `namespace:chainId` |
| CAIP-10 | Account address format: `namespace:chainId:address` |
| CID | Content Identifier (IPFS hash) |
| ERC-8004 | NFT-based standard for agent identity, reputation, and validation |
| Groth16 | zkSNARK proof system with efficient verification |
| Identity Registry | Smart contract managing agent NFTs and metadata URIs |
| MCP | Model Context Protocol (modelcontextprotocol.io) |
| OASF | Open Agent Skills Framework taxonomy (github.com/agntcy/oasf) |
| Reputation Registry | Smart contract managing feedback and reputation signals |
| TEE | Trusted Execution Environment |
| Validation Registry | Smart contract managing validation requests and responses |
| x402 | Payment protocol for agents using HTTP 402 |
| zkML | Zero-Knowledge Machine Learning |

---

## Production Statistics (8004scan, Dec 2025)

- **4,725 agents** on Base Sepolia
- URI formats: IPFS 51%, HTTP 22%, Data URI 18%
- Endpoint adoption: A2A 85%, MCP 75%, OASF 60%, agentWallet 45%
- Parse success: 58.1% perfect, 69.9% total (including warnings)
- 8,219 validation records (100% Groth16 zkSNARK)
- 99.4% of agent metadata < 2KB
