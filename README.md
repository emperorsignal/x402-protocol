# X402 Protocol ⚡

### Decentralized AI Signal Execution Layer

The X402 Protocol is a proprietary API standard built on the HTTP 402 "Payment Required" framework. It provides developers and institutional traders with a structured way to access Emperor Signal's 5-AI Oracle consensus system and real-time market signals using a pay-per-use micropayment model.

## 🌟 Key Features

- 🧩 **Native HTTP 402 integration**: Purpose-built for decentralized value exchange
- 🧠 **AI Consensus Engine**: Verified signals from 5 specialized AI models (DeepSeek, Claude, Gemini, Grok, GPT-4o)
- ⏱️ **Sub-second Latency**: Real-time signal delivery via high-performance WebSockets
- 💰 **Micropayment Model**: No monthly subscriptions required—pay only for the signals you consume
- 🔗 **Payments Network**: Pay-As-You-Go uses Solana USDC (SPL). EVM payments are not enabled yet.
- 🤖 **Accountless Pay-As-You-Go**: Bots can pay on-chain per request—no account needed!

## 🔒 Security Features

- **Replay Protection**: Each transaction signature can only be used once
- **Time-Limited Payments**: Transactions must be within 10 minutes of block time
- **On-Chain Verification**: Real-time Solana RPC validation of USDC transfers
- **Atomic Transactions**: Credit deductions use Firestore transactions to prevent race conditions
- **Dynamic Pricing**: Prices sync instantly from admin configuration

## 🚀 Capabilities

### 📡 Signal Streams
Access real-time Mega and Trend signals with granular metadata:
- Confidence scores (0-100%)
- Timeframe alignment counts
- Technical indicator backing (MRI, SuperTrend, etc.)

### 🔮 AI Oracle Queries
Programmatically query the Emperor Signal Oracle for custom market analysis:
- Consensus-based reasoning
- Multi-model voting breakdowns
- Contextual market sentiment analysis

### 📈 Institutional Data
Retrieve high-fidelity market data across 100+ top assets:
- Live price feeds
- Volume & momentum analytics
- Fear & Greed index integration

## 🛠️ Getting Started

X402 Protocol supports **two modes**:

### Mode 1: Account-Based Credits (Recommended for high-volume)
1. Create account at [Emperor Signal](https://emperorsignal.com)
2. Top up credits via Stripe or crypto
3. Use API key or Bearer token for requests

### Mode 2: Pay-As-You-Go (Accountless/Trustless)
1. Make a request → Get 402 with payment info
2. Pay on-chain (USDC to treasury wallet)
3. Retry with `X-Payment-Signature` header → Get signal

**No account needed for pay-as-you-go!**

---

## 🤖 Agent Integration (Server-Side Payer)
This pattern is designed for AI agents (including OpenClaw and Moltbots) and backend services.

### Recommended Flow
1. Call `/x402TrendSignal` with no auth
2. Receive `402 Payment Required` with `paymentInfo`
3. Pay USDC on Solana using a **server-side wallet**
4. Retry with `X-Payment-Signature` header

### Security Checklist
- Keep private keys **server-side only**
- Encrypt keys at rest (KMS or vault)
- **Never store PINs** in Firestore or client storage
- Rate-limit payment attempts per user/session
- Log tx signatures for audit + replay protection

### Minimal Example (Server-Side Payer)
```javascript
// 1) Attempt request (no auth)
const first = await fetch("https://us-central1-emperiorsignal.cloudfunctions.net/x402TrendSignal", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ symbol: "BTC" })
});

if (first.status === 402) {
    const { paymentInfo } = await first.json();
    const txSig = await payUSDC(paymentInfo.wallet, paymentInfo.amount);

    const paid = await fetch("https://us-central1-emperiorsignal.cloudfunctions.net/x402TrendSignal", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "X-Payment-Signature": txSig
        },
        body: JSON.stringify({ symbol: "BTC" })
    });

    const signal = await paid.json();
    console.log(signal);
}
```

---

## 🚀 Pay-As-You-Go Flow (Accountless)

The true X402 protocol - trustless, on-chain micropayments:

```
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│   Bot/AI    │───────▶│   Server    │◀──────▶│  Blockchain │
└─────────────┘        └─────────────┘        └─────────────┘
      │                      │                      │
      │  1. POST /signal     │                      │
      │─────────────────────▶│                      │
      │                      │                      │
      │  2. 402 + wallet     │                      │
      │◀─────────────────────│                      │
      │                      │                      │
      │  3. Pay USDC         │                      │
      │──────────────────────────────────────────▶ │
      │                      │                      │
      │  4. POST + tx sig    │                      │
      │─────────────────────▶│  5. Verify on-chain │
      │                      │─────────────────────▶│
      │                      │◀─────────────────────│
      │  6. Signal data      │                      │
      │◀─────────────────────│                      │
```

### Python Example (Pay-As-You-Go)

```python
import requests
from solana.rpc.api import Client
from solders.keypair import Keypair
from solders.pubkey import Pubkey
from spl.token.instructions import transfer_checked, TransferCheckedParams

BASE_URL = "https://us-central1-emperiorsignal.cloudfunctions.net"
TREASURY = "AAMLmYgPCSNQHB8mLPjkKpdHkqNu8pJxS58HSe9vKwFA"
USDC_MINT = "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"

def get_signal_pay_as_you_go(symbol="BTC"):
    """
    Get trend signal using pay-as-you-go (no account needed).
    
    IMPORTANT: Use symbol without 'USDT' suffix!
    Examples: 'BTC', 'ETH', 'SOL', 'PEPE' (NOT 'BTCUSDT')
    """
    # Step 1: Make request (no auth needed)
    response = requests.post(
        f"{BASE_URL}/x402TrendSignal",
        json={"symbol": symbol}
    )
    
    if response.status_code != 402:
        return response.json()  # Already have credits or other error
    
    # Step 2: Get payment info from 402 response
    payment_info = response.json().get("paymentInfo", {})
    amount_usdc = payment_info.get("amount", 0.01)
    wallet = payment_info.get("wallet")
    
    print(f"Payment required: ${amount_usdc} USDC to {wallet}")
    
    # Step 3: Pay on-chain (you need a funded wallet)
    tx_signature = pay_usdc(amount_usdc, wallet)  # Your payment function
    
    # Step 4: Retry with payment signature
    response = requests.post(
        f"{BASE_URL}/x402TrendSignal",
        json={"symbol": symbol},
        headers={"X-Payment-Signature": tx_signature}
    )
    
    return response.json()

# Example output
signal = get_signal_pay_as_you_go("BTC")  # Use 'BTC' not 'BTCUSDT'!
print(f"Signal: {signal['signal']}")
print(f"Confidence: {signal['confidence']}%")
```

### Node.js Example (Pay-As-You-Go)

```javascript
const axios = require('axios');
const { Connection, PublicKey, Transaction } = require('@solana/web3.js');
const { getOrCreateAssociatedTokenAccount, createTransferInstruction } = require('@solana/spl-token');

const BASE_URL = 'https://us-central1-emperiorsignal.cloudfunctions.net';
const TREASURY = 'AAMLmYgPCSNQHB8mLPjkKpdHkqNu8pJxS58HSe9vKwFA';

async function getSignalPayAsYouGo(symbol, wallet) {
    // Step 1: Try request (no auth)
    try {
        const response = await axios.post(`${BASE_URL}/x402TrendSignal`, { symbol });
        return response.data;
    } catch (error) {
        if (error.response?.status !== 402) throw error;
        
        // Step 2: Get payment info
        const paymentInfo = error.response.data.paymentInfo;
        console.log(`Payment required: $${paymentInfo.amount} USDC`);
        
        // Step 3: Pay on-chain
        const txSignature = await payUSDC(wallet, paymentInfo.amount, TREASURY);
        
        // Step 4: Retry with signature
        const retryResponse = await axios.post(
            `${BASE_URL}/x402TrendSignal`,
            { symbol },
            { headers: { 'X-Payment-Signature': txSignature } }
        );
        
        return retryResponse.data;
    }
}
```

---

## 💳 Account-Based Credits

For high-volume users, you can pre-purchase credits:

### 1. Create an Account
Sign up at [Emperor Signal](https://emperorsignal.com) and navigate to the X402 Protocol page.

### 2. Generate API Keys
Navigate to your [X402 Dashboard](https://emperorsignal.com/x402) and click "Create Key" to generate a new API key.

### 3. Top Up Credits
- **Stripe**: Pay with credit/debit card (minimum $5)
- **Crypto**: Send USDC to treasury wallet (minimum $5)

### 4. Use Your Credits

```python
# With API Key (use symbol WITHOUT 'USDT' suffix!)
response = requests.post(
    f"{BASE_URL}/x402TrendSignal",
    json={"symbol": "BTC", "apiKey": "x402_your_api_key_here"}
)

# With Bearer Token (Firebase Auth)
response = requests.post(
    f"{BASE_URL}/x402TrendSignal",
    json={"symbol": "SOL"},  # NOT 'SOLUSDT'!
    headers={"Authorization": f"Bearer {firebase_id_token}"}
)
```

---

## 📡 API Reference

### Base URL
```
https://us-central1-emperiorsignal.cloudfunctions.net
```

### Authentication Options

| Method | Where | Use Case |
|--------|-------|----------|
| Pay-As-You-Go | Header: `X-Payment-Signature: <tx_sig>` | Accountless, per-request |
| API Key | JSON body: `apiKey` | External apps, bots |
| Bearer Token | Header: `Authorization: Bearer <token>` | Web apps with Firebase Auth |

### Endpoints

| Endpoint | Method | Cost | Description |
|----------|--------|------|-------------|
| `/x402TrendSignal` | POST | 1 credit ($0.01) | Get trend signal for a symbol |
| `/x402MegaSignal` | POST | 2 credits ($0.02) | Get mega signal (multi-timeframe) |
| `/x402Gate` | POST | varies | Check/deduct credits for custom services |
| `/x402GetPricing` | GET | free | Get current pricing and service availability |
| `/x402GetCredits` | GET | free | Get credit balance (auth required) |
| `/x402VerifyPayment` | POST | free | Verify top-up and add credits |

### ⚠️ Symbol Format

**IMPORTANT**: Use symbols WITHOUT the 'USDT' suffix!

```json
// ✅ CORRECT
{"symbol": "BTC"}
{"symbol": "ETH"}
{"symbol": "SOL"}

// ❌ WRONG
{"symbol": "BTCUSDT"}
{"symbol": "ETHUSDT"}
```

### 📊 Supported Symbols (124 total)

**Major Coins:**
```
BTC ETH BNB SOL XRP DOGE ADA AVAX DOT LINK
LTC BCH ETC XLM XMR ATOM NEAR UNI AAVE MKR
```

**DeFi & Layer 2:**
```
ARB OP MATIC IMX STX SUI APT SEI TIA INJ
JUP JTO ONDO PENDLE LDO DYDX GMX CRV SNX COMP
```

**Meme & Trending:**
```
PEPE SHIB DOGE WIF BONK FLOKI MEME TURBO NEIRO BOME
```

**AI & Gaming:**
```
FET AGIX RNDR TAO OCEAN AXS SAND MANA GALA ENJ
```

**Full List:**
```
1INCH AAVE ADA AGIX ALGO ANKR APE APT AR ARB
ATOM AVAX AXS BAND BAT BCH BLUR BNB BOME BONK
BTC CAKE CELO CELR CFX CHZ CKB COMP CRV CTSI
CVX DASH DOGE DOGS DOT DYDX EGLD ENA ENJ ENS
EOS ETC ETH FET FIL FLOKI FLOW FTM GALA GMX
GRT HBAR HOT ICP IMX INJ IOTA IOTX JTO JUP
KAVA KLAY KSM LDO LINK LRC LTC MANA MASK MATIC
MEME MINA MKR NEAR NEIRO NEO NOT OCEAN OGN ONDO
ONE ONT OP ORDI PENDLE PEOPLE PEPE PYTH QNT RNDR
ROSE RUNE SAND SC SEI SHIB SKL SNX SOL STORJ
STRK STX SUI SUSHI TAO THETA TIA TON TRX TURBO
UNI VET WAVES WIF WLD XLM XMR XRP XTZ YFI
ZEC ZEN ZIL ZRX
```

### Pricing

| Service | Credits | USD Value |
|---------|---------|-----------|
| Trend Signal | 1 | $0.01 |
| Mega Signal | 2 | $0.02 |
| Data Query | 0.5 | $0.005 |
| Streaming/hr | 200 | $2.00 |
| Bulk/100 | 50 | $0.50 |

## 💻 Code Examples

### Python Example

```python
import requests
import firebase_admin
from firebase_admin import auth

# Initialize Firebase (get your service account from Firebase Console)
cred = firebase_admin.credentials.Certificate("serviceAccountKey.json")
firebase_admin.initialize_app(cred)

BASE_URL = "https://us-central1-emperiorsignal.cloudfunctions.net"

def get_trend_signal(symbol="BTC", id_token=None):
    """
    Fetch trend signal for a given symbol.
    
    IMPORTANT: Use symbol without 'USDT' suffix!
    Examples: 'BTC', 'ETH', 'SOL', 'PEPE'
    """
    headers = {
        "Authorization": f"Bearer {id_token}",
        "Content-Type": "application/json"
    }
    
    response = requests.post(
        f"{BASE_URL}/x402TrendSignal",
        headers=headers,
        json={"symbol": symbol}
    )
    
    if response.status_code == 402:
        # X402 Payment Required
        payment_info = response.json().get("paymentInfo", {})
        print(f"Insufficient credits. Top up at: {payment_info.get('topUpUrl')}")
        return None
    
    return response.json()

# Example usage - use 'BTC' not 'BTCUSDT'!
signal = get_trend_signal("BTC", id_token="your_firebase_id_token")
if signal and signal.get("success"):
    print(f"Signal: {signal['signal']}")
    print(f"Confidence: {signal['confidence']}%")
```

### Node.js Example

```javascript
const axios = require('axios');

const BASE_URL = 'https://us-central1-emperiorsignal.cloudfunctions.net';

/**
 * Get mega signal for a symbol.
 * IMPORTANT: Use symbol without 'USDT' suffix!
 * Examples: 'BTC', 'ETH', 'SOL', 'PEPE'
 */
async function getMegaSignal(symbol, idToken) {
    try {
        const response = await axios.post(
            `${BASE_URL}/x402MegaSignal`,
            { symbol },
            {
                headers: {
                    'Authorization': `Bearer ${idToken}`,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        console.log('Signal:', response.data.direction);
        console.log('Confidence:', response.data.confidence);
        console.log('Is Mega:', response.data.isMegaSignal);
        return response.data;
        
    } catch (error) {
        if (error.response?.status === 402) {
            // X402 Payment Required
            const paymentInfo = error.response.data.paymentInfo;
            console.log('Insufficient credits!');
            console.log(`Wallet: ${paymentInfo.wallet}`);
            console.log(`Min Top-up: $${paymentInfo.minTopUp} ${paymentInfo.currency}`);
            console.log(`Top up at: ${paymentInfo.topUpUrl}`);
        }
        throw error;
    }
}

// Example usage - use 'ETH' not 'ETHUSDT'!
getMegaSignal('ETH', 'your_firebase_id_token');
```

### cURL Example

```bash
# Get Trend Signal (use symbol without USDT suffix!)
curl -X POST https://us-central1-emperiorsignal.cloudfunctions.net/x402TrendSignal \
  -H "Content-Type: application/json" \
  -H "X-Payment-Signature: YOUR_TX_SIGNATURE" \
  -d '{"symbol": "BTC"}'

# Response (success)
{
  "success": true,
  "symbol": "BTC",
  "signal": "SELL",
  "strength": 0.65,
  "confidence": 65.1,
  "atr": {
    "percent": 0.75,
    "value": 696.68
  },
  "x402": {
    "payAsYouGo": true,
    "txSignature": "3UX7tcv...gWtX",
    "amountPaid": 0.01
  }
}

# Response (insufficient credits - HTTP 402)
{
  "success": false,
  "allowed": false,
  "remainingCredits": 0,
  "cost": 1,
  "paymentInfo": {
    "wallet": "AAMLmYgPCSNQHB8mLPjkKpdHkqNu8pJxS58HSe9vKwFA",
    "amount": 0.01,
    "minTopUp": 5.00,
    "currency": "USDC",
    "network": "solana",
    "topUpUrl": "https://emperorsignal.com/x402"
  }
}
```

## 🔐 X402 Protocol Headers

When credits are insufficient, the API returns HTTP 402 with these headers:

| Header | Value | Description |
|--------|-------|-------------|
| `X-Payment-Required` | `x402` | Protocol identifier |
| `X-Payment-Wallet` | `AAMLmY...` | Solana treasury wallet |
| `X-Payment-Amount` | `1` | Credits required |
| `X-Payment-Currency` | `USDC` | Payment currency |
| `X-Payment-Network` | `solana` | Blockchain network |

## 💳 Top-Up Options

### Via Web Dashboard
Visit [emperorsignal.com/x402](https://emperorsignal.com/x402) and click "Top Up Credits"

### Via Crypto
Send USDC to the treasury wallet:
```
AAMLmYgPCSNQHB8mLPjkKpdHkqNu8pJxS58HSe9vKwFA
```
Minimum: $5.00 USDC on Solana

## 📄 Documentation

For full endpoint specifications, rate limits, and authentication details, visit the [Official Documentation](https://emperorsignal.com/documentation).

## ⚖️ License

Usage of the X402 Protocol is subject to the [Emperor Signal Terms of Service](https://emperorsignal.com/terms-of-service).

---

> "Empowering the next generation of algorithmic trading."

**Emperor Signal** | [Website](https://emperorsignal.com) | [Twitter](https://twitter.com/emperorsignal) | [Discord](https://discord.gg/emperorsignal)
