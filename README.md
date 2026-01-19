# X402 Protocol ⚡

### Decentralized AI Signal Execution Layer

The X402 Protocol is a proprietary API standard built on the HTTP 402 "Payment Required" framework. It provides developers and institutional traders with a structured way to access Emperor Signal's 5-AI Oracle consensus system and real-time market signals using a pay-per-use micropayment model.

## 🌟 Key Features

- 🧩 **Native HTTP 402 integration**: Purpose-built for decentralized value exchange
- 🧠 **AI Consensus Engine**: Verified signals from 5 specialized AI models (DeepSeek, Claude, Gemini, Grok, GPT-4o)
- ⏱️ **Sub-second Latency**: Real-time signal delivery via high-performance WebSockets
- 💰 **Micropayment Model**: No monthly subscriptions required—pay only for the signals you consume
- 🔗 **Multi-Chain Support**: Native support for Solana (SPL) and EVM-compatible networks

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

### 1. Create an Account
Sign up at [Emperor Signal](https://emperorsignal.com) and navigate to the X402 Protocol page.

### 2. Generate API Keys
Navigate to your [X402 Dashboard](https://emperorsignal.com/x402) and click "Create Key" to generate a new API key.

### 3. Fund Your Protocol Wallet
X402 operates on a credit-based system. Add credits via:
- **Stripe**: Pay with credit/debit card
- **Crypto**: Deposit USDC or IMPERIUM tokens on Solana

## 📡 API Reference

### Base URL
```
https://us-central1-emperiorsignal.cloudfunctions.net
```

### Authentication
All requests require a valid Firebase Auth token:
```
Authorization: Bearer <your_firebase_id_token>
```

Or use an API key:
```
X-API-Key: <your_api_key>
```

### Endpoints

| Endpoint | Method | Cost | Description |
|----------|--------|------|-------------|
| `/x402TrendSignal` | POST | 1 credit | Get trend signal for a symbol |
| `/x402MegaSignal` | POST | 2 credits | Get mega signal (multi-timeframe) |
| `/x402Gate` | POST | varies | Check/deduct credits for custom services |

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

def get_trend_signal(symbol="BTCUSDT", id_token=None):
    """Fetch trend signal for a given symbol"""
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

# Example usage
signal = get_trend_signal("BTCUSDT", id_token="your_firebase_id_token")
if signal and signal.get("success"):
    print(f"Signal: {signal['data']['signal']}")
    print(f"Confidence: {signal['data']['confidence']}%")
```

### Node.js Example

```javascript
const axios = require('axios');

const BASE_URL = 'https://us-central1-emperiorsignal.cloudfunctions.net';

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
        
        console.log('Signal:', response.data.data.signal);
        console.log('Confidence:', response.data.data.confidence);
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

// Example usage
getMegaSignal('ETHUSDT', 'your_firebase_id_token');
```

### cURL Example

```bash
# Get Trend Signal
curl -X POST https://us-central1-emperiorsignal.cloudfunctions.net/x402TrendSignal \
  -H "Authorization: Bearer YOUR_FIREBASE_ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"symbol": "BTCUSDT"}'

# Response (success)
{
  "success": true,
  "data": {
    "symbol": "BTCUSDT",
    "signal": "BUY",
    "confidence": 85,
    "timestamp": "2026-01-20T10:30:00Z"
  },
  "remainingCredits": 99
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
