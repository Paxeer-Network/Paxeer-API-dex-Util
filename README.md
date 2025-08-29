# PaxDex Protocol - Frontend Integration Guide

This comprehensive guide provides all the necessary information for integrating with the PaxDex Protocol API and smart contracts.

## 📋 Table of Contents

1. [API Base URL](#api-base-url)
2. [Authentication](#authentication)
3. [REST API Endpoints](#rest-api-endpoints)
4. [WebSocket Connection](#websocket-connection)
5. [Smart Contract Integration](#smart-contract-integration)
6. [Supported Tokens](#supported-tokens)
7. [Error Handling](#error-handling)
8. [Rate Limits](#rate-limits)

---

## 🌐 API Base URL

**Production API:** `https://dex-api.paxeer.app`

All API endpoints should be prefixed with the base URL.

---

## 🔐 Authentication

Currently, the API does not require authentication for read operations. All endpoints are publicly accessible.

---

## 📡 REST API Endpoints

### 1. Health Check

**Endpoint:** `GET /api/health`

**Description:** Check API health status

**Response:**
```json
{
  "success": true,
  "message": "PaxDex Engine is healthy",
  "timestamp": 1693234567890,
  "uptime": 12345.67,
  "services": {
    "priceService": true,
    "oracleService": true,
    "websocketService": true,
    "cacheService": true,
    "databaseService": true
  }
}
```

### 2. Get All Token Prices

**Endpoint:** `GET /api/prices`

**Description:** Retrieve current prices for all supported tokens

**Response:**
```json
{
  "success": true,
  "data": {
    "0x96465d06640aff1a00888d4b9217c9eae708c419": {
      "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
      "symbol": "WBTC",
      "name": "Wrapped Bitcoin",
      "decimals": 8,
      "price": 45230.50,
      "adjustedPrice": 45230500000000000,
      "change24h": 2.45,
      "lastUpdated": 1693234567,
      "timestamp": 1693234567890
    }
  },
  "count": 12,
  "timestamp": 1693234567890,
  "cached": false
}
```

### 3. Get Single Token Price

**Endpoint:** `GET /api/prices/:address`

**Description:** Get price data for a specific token

**Parameters:**
- `address` (string): Token contract address

**Response:**
```json
{
  "success": true,
  "data": {
    "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
    "symbol": "WBTC",
    "name": "Wrapped Bitcoin",
    "decimals": 8,
    "price": 45230.50,
    "adjustedPrice": 45230500000000000,
    "change24h": 2.45,
    "lastUpdated": 1693234567,
    "timestamp": 1693234567890
  },
  "timestamp": 1693234567890
}
```

### 4. Get Token Metadata

**Endpoint:** `GET /api/tokens`

**Description:** Retrieve metadata for all supported tokens

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
      "symbol": "WBTC",
      "name": "Wrapped Bitcoin",
      "decimals": 8,
      "coinGeckoId": "wrapped-bitcoin"
    }
  ],
  "count": 12,
  "timestamp": 1693234567890
}
```

### 5. Get Swap Token Data (Frontend Optimized)

**Endpoint:** `GET /api/tokens/swap`

**Description:** Get enhanced token data optimized for swap interfaces

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
      "symbol": "WBTC",
      "name": "Wrapped Bitcoin",
      "decimals": 8,
      "price": 45230.50,
      "marketCap": 890456000000,
      "volume24h": 12345678900,
      "priceChange24h": 2.45,
      "image": "https://assets.coingecko.com/coins/images/7598/small/wrapped_bitcoin_wbtc.png",
      "rank": 15
    }
  ],
  "count": 12,
  "timestamp": 1693234567890,
  "cached": true
}
```

### 6. Get Price History

**Endpoint:** `GET /api/prices/:address/history`

**Description:** Get historical price data for a token

**Query Parameters:**
- `timeframe` (string): Time period ('1h', '24h', '7d', '30d') - default: '24h'
- `limit` (number): Maximum number of data points - default: 100

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "timestamp": 1693234567890,
      "price": 45230.50,
      "adjustedPrice": 45230500000000000
    }
  ],
  "count": 100,
  "timeframe": "24h",
  "token": {
    "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
    "symbol": "WBTC",
    "name": "Wrapped Bitcoin"
  }
}
```

### 7. Get System Status

**Endpoint:** `GET /api/status`

**Description:** Get detailed system status and metrics

**Response:**
```json
{
  "success": true,
  "data": {
    "server": {
      "uptime": 12345.67,
      "memory": {
        "rss": 123456789,
        "heapTotal": 87654321,
        "heapUsed": 65432109
      }
    },
    "services": {
      "priceService": {
        "status": "running",
        "lastUpdate": 1693234567890,
        "updateInterval": 30000
      },
      "oracleService": {
        "status": "connected",
        "contractAddress": "0x6e5da6d7a89c6B7cB0e5c64fcf326292F76A0352",
        "lastUpdate": 1693234567890
      }
    },
    "database": {
      "status": "connected",
      "totalTokens": 12,
      "totalPriceRecords": 123456
    }
  }
}
```

---

## 🔌 WebSocket Connection

### Connection Details

**WebSocket URL:** `wss://dex-api.paxeer.app:3001`

### Connection Example

```javascript
const ws = new WebSocket('wss://dex-api.paxeer.app:3001');

ws.onopen = function(event) {
    console.log('Connected to PaxDex WebSocket');
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Received update:', data);
};

ws.onclose = function(event) {
    console.log('WebSocket connection closed');
};

ws.onerror = function(error) {
    console.error('WebSocket error:', error);
};
```

### Message Types

#### 1. Price Updates
```json
{
  "type": "price_update",
  "data": {
    "0x96465d06640aff1a00888d4b9217c9eae708c419": {
      "address": "0x96465d06640aff1a00888d4b9217c9eae708c419",
      "symbol": "WBTC",
      "price": 45230.50,
      "change24h": 2.45,
      "timestamp": 1693234567890
    }
  },
  "timestamp": 1693234567890
}
```

#### 2. Connection Status
```json
{
  "type": "connection",
  "status": "connected",
  "clientId": "unique-client-id",
  "timestamp": 1693234567890
}
```

---

## 🔗 Smart Contract Integration

### Contract Addresses

- **Vault Contract:** `0x49B0f9a0554da1A7243A9C8ac5B45245A66D90ff`
- **Oracle Contract:** `0x6e5da6d7a89c6B7cB0e5c64fcf326292F76A0352`

### Vault Contract ABI

```json
[
  {
    "inputs": [
      {"internalType": "address", "name": "_tokenIn", "type": "address"},
      {"internalType": "address", "name": "_tokenOut", "type": "address"},
      {"internalType": "uint256", "name": "_amountIn", "type": "uint256"},
      {"internalType": "uint256", "name": "_minAmountOut", "type": "uint256"}
    ],
    "name": "swapExactTokensForTokens",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "anonymous": false,
    "inputs": [
      {"indexed": true, "internalType": "address", "name": "user", "type": "address"},
      {"indexed": true, "internalType": "address", "name": "tokenIn", "type": "address"},
      {"indexed": true, "internalType": "address", "name": "tokenOut", "type": "address"},
      {"indexed": false, "internalType": "uint256", "name": "amountIn", "type": "uint256"},
      {"indexed": false, "internalType": "uint256", "name": "amountOut", "type": "uint256"}
    ],
    "name": "Swap",
    "type": "event"
  },
  {
    "inputs": [],
    "name": "SWAP_FEE_BPS",
    "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "BPS_DENOMINATOR", 
    "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
    "stateMutability": "view",
    "type": "function"
  }
]
```

### Swap Implementation Example

```javascript
import { ethers } from 'ethers';

const VAULT_ADDRESS = '0x49B0f9a0554da1A7243A9C8ac5B45245A66D90ff';
const VAULT_ABI = [/* ABI from above */];

async function executeSwap(tokenIn, tokenOut, amountIn, minAmountOut) {
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  const signer = provider.getSigner();
  const vaultContract = new ethers.Contract(VAULT_ADDRESS, VAULT_ABI, signer);

  try {
    // First approve token spending
    const tokenContract = new ethers.Contract(tokenIn, ERC20_ABI, signer);
    const approveTx = await tokenContract.approve(VAULT_ADDRESS, amountIn);
    await approveTx.wait();

    // Execute swap
    const swapTx = await vaultContract.swapExactTokensForTokens(
      tokenIn,
      tokenOut,
      amountIn,
      minAmountOut
    );
    
    const receipt = await swapTx.wait();
    return receipt;
  } catch (error) {
    console.error('Swap failed:', error);
    throw error;
  }
}
```

### Fee Calculation

The vault charges a 0.3% fee on swaps (30 basis points):

```javascript
// Get fee rate from contract
const swapFeeBps = await vaultContract.SWAP_FEE_BPS(); // Returns 30
const bpsDenominator = await vaultContract.BPS_DENOMINATOR(); // Returns 10000

// Calculate fee
const fee = (amountIn * swapFeeBps) / bpsDenominator;
const amountAfterFee = amountIn - fee;
```

---

## 🪙 Supported Tokens

| Symbol | Name | Address | Decimals | CoinGecko ID |
|--------|------|---------|----------|--------------|
| WBTC | Wrapped Bitcoin | `0x96465d06640aff1a00888d4b9217c9eae708c419` | 8 | wrapped-bitcoin |
| wstETH | Wrapped stETH | `0xeb2c4ae6fe90f9bf25c94269236cb5408e00e188` | 18 | wrapped-steth |
| WETH | Wrapped Ethereum | `0xd0c1a714c46c364dbdd4e0f7b0b6ba5354460da7` | 18 | weth |
| BNB | Binance Coin | `0xb947bcd6bcce03846ac716fc39a3133c4bf0108e` | 18 | binancecoin |
| UNI | Uniswap | `0x90a271d104aea929b68867b3050efacbc1a28e84` | 18 | uniswap |
| DOT | Polkadot | `0xdccec2b62dd102276b7ba689405a5cd7504a8cd3` | 10 | polkadot |
| TON | The Open Network | `0x9d60b394276e67a44f2d80e1ab7cfafa4e151f02` | 9 | the-open-network |
| CRO | Crypto.com Chain | `0xa1956408cbeb4c0d2c257be394b9bdf4c9e1a061` | 8 | crypto-com-chain |
| stETH | Lido Staked Ether | `0xbbf11b964ac48bd11109b68dffe129b45671e34e` | 18 | staked-ether |
| USDT | Tether USD | `0x2a401fe7616c4aba69b147b4b725ce48ca7ec660` | 6 | tether |
| SOL | Solana | `0x7100cf39ff0e845d7751fb56198b8dd16c6ecb2a` | 9 | solana |
| USDC | USD Coin | `0x29e1f94f6b209b57ecdc1fe87448a6d085a78a5a` | 6 | usd-coin |

---

## 🚨 Error Handling

### Standard Error Response Format

All API errors follow this structure:

```json
{
  "success": false,
  "message": "Error description",
  "error": "Detailed error information",
  "timestamp": 1693234567890
}
```

### Common Error Codes

| HTTP Code | Description | Cause |
|-----------|-------------|-------|
| 400 | Bad Request | Invalid parameters or malformed request |
| 404 | Not Found | Endpoint or resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Service temporarily down |

### Error Handling Example

```javascript
async function fetchTokenData() {
  try {
    const response = await fetch('https://dex-api.paxeer.app/api/tokens/swap');
    const data = await response.json();
    
    if (!data.success) {
      throw new Error(data.message);
    }
    
    return data.data;
  } catch (error) {
    console.error('API Error:', error.message);
    // Handle error appropriately
    return [];
  }
}
```

---

## ⚡ Rate Limits

- **REST API:** 100 requests per minute per IP
- **WebSocket:** No rate limits on connections, but message throttling may apply
- **Cached Data:** API responses include cache information to help optimize requests

### Rate Limit Headers

The API includes rate limit information in response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1693234567
```

---

## 🛠️ Integration Examples

### React Hook for Price Data

```javascript
import { useState, useEffect } from 'react';

export const usePaxDexPrices = () => {
  const [prices, setPrices] = useState({});
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let ws;
    
    const fetchInitialPrices = async () => {
      try {
        const response = await fetch('https://dex-api.paxeer.app/api/prices');
        const data = await response.json();
        setPrices(data.data);
        setLoading(false);
      } catch (err) {
        setError(err.message);
        setLoading(false);
      }
    };

    const connectWebSocket = () => {
      ws = new WebSocket('wss://dex-api.paxeer.app:3001');
      
      ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        if (message.type === 'price_update') {
          setPrices(prev => ({ ...prev, ...message.data }));
        }
      };
    };

    fetchInitialPrices();
    connectWebSocket();

    return () => {
      if (ws) ws.close();
    };
  }, []);

  return { prices, loading, error };
};
```

### Swap Calculation Helper

```javascript
export class PaxDexSwapCalculator {
  static SWAP_FEE_BPS = 30;
  static BPS_DENOMINATOR = 10000;

  static calculateOutputAmount(inputAmount, inputPrice, outputPrice, inputDecimals, outputDecimals) {
    // Convert to standard 18 decimal format
    const standardInputAmount = inputAmount * Math.pow(10, 18 - inputDecimals);
    
    // Calculate USD value
    const usdValue = (standardInputAmount * inputPrice) / Math.pow(10, 18);
    
    // Apply fee
    const feeAmount = (usdValue * this.SWAP_FEE_BPS) / this.BPS_DENOMINATOR;
    const usdAfterFee = usdValue - feeAmount;
    
    // Convert to output token
    const outputAmount = (usdAfterFee / outputPrice) * Math.pow(10, 18);
    
    // Convert to token's actual decimals
    const finalAmount = outputAmount / Math.pow(10, 18 - outputDecimals);
    
    return Math.floor(finalAmount);
  }

  static calculateMinimumOutput(expectedOutput, slippageBps = 50) {
    return Math.floor(expectedOutput * (this.BPS_DENOMINATOR - slippageBps) / this.BPS_DENOMINATOR);
  }
}
```

### Complete Swap Interface Example

```javascript
import { ethers } from 'ethers';
import { PaxDexSwapCalculator } from './SwapCalculator';

const VAULT_CONTRACT_ADDRESS = '0x49B0f9a0554da1A7243A9C8ac5B45245A66D90ff';
const VAULT_ABI = [/* Full ABI from above */];

export class PaxDexSwapInterface {
  constructor(provider) {
    this.provider = provider;
    this.signer = provider.getSigner();
    this.vaultContract = new ethers.Contract(
      VAULT_CONTRACT_ADDRESS, 
      VAULT_ABI, 
      this.signer
    );
  }

  async executeSwap({
    tokenIn,
    tokenOut,
    amountIn,
    slippageBps = 50,
    deadline = Math.floor(Date.now() / 1000) + 1200 // 20 minutes
  }) {
    try {
      // Get current prices
      const pricesResponse = await fetch('https://dex-api.paxeer.app/api/prices');
      const pricesData = await pricesResponse.json();
      
      const tokenInData = pricesData.data[tokenIn.toLowerCase()];
      const tokenOutData = pricesData.data[tokenOut.toLowerCase()];
      
      if (!tokenInData || !tokenOutData) {
        throw new Error('Token price data not available');
      }

      // Calculate expected output
      const expectedOutput = PaxDexSwapCalculator.calculateOutputAmount(
        amountIn,
        tokenInData.price,
        tokenOutData.price,
        tokenInData.decimals,
        tokenOutData.decimals
      );

      const minAmountOut = PaxDexSwapCalculator.calculateMinimumOutput(
        expectedOutput,
        slippageBps
      );

      // Approve token spending
      const tokenInContract = new ethers.Contract(
        tokenIn,
        ['function approve(address,uint256) returns(bool)'],
        this.signer
      );
      
      const approveTx = await tokenInContract.approve(VAULT_CONTRACT_ADDRESS, amountIn);
      await approveTx.wait();

      // Execute swap
      const swapTx = await this.vaultContract.swapExactTokensForTokens(
        tokenIn,
        tokenOut,
        amountIn,
        minAmountOut
      );

      const receipt = await swapTx.wait();
      
      return {
        success: true,
        transactionHash: receipt.transactionHash,
        expectedOutput,
        minAmountOut,
        receipt
      };

    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
}
```

---

## 📝 Additional Notes

### Network Information
- **Blockchain:** Paxeer Network
- **Chain ID:** 80000
- **RPC URL:** https://rpc-paxeer-network-djjz47ii4b.t.conduit.xyz
- **Gas Considerations:** Swap operations typically use 150,000-250,000 gas

### Best Practices

1. **Caching:** Utilize cached data when available to reduce API calls
2. **Error Handling:** Always implement robust error handling for both API and blockchain interactions
3. **Slippage Protection:** Use appropriate slippage settings (0.5% - 2% recommended)
4. **Price Impact:** Calculate and display price impact for large swaps
5. **Transaction Monitoring:** Monitor transaction status and handle failures gracefully

### Support

For technical support or questions about the API integration:
- **Documentation:** This guide
- **API Status:** `GET https://dex-api.paxeer.app/api/health`
- **WebSocket Status:** Connect to `wss://dex-api.paxeer.app:3001` for real-time updates

---

## 📚 Quick Reference

### Essential URLs
- **API Base:** `https://dex-api.paxeer.app`
- **WebSocket:** `wss://dex-api.paxeer.app:3001`
- **Vault Contract:** `0x49B0f9a0554da1A7243A9C8ac5B45245A66D90ff`
- **Oracle Contract:** `0x6e5da6d7a89c6B7cB0e5c64fcf326292F76A0352`

### Key Endpoints
- **Prices:** `/api/prices`
- **Swap Data:** `/api/tokens/swap`
- **Health Check:** `/api/health`
- **Token Info:** `/api/tokens`
- **Price History:** `/api/prices/:address/history`

### Contract Constants
- **Swap Fee:** 30 BPS (0.3%)
- **BPS Denominator:** 10,000
- **Supported Tokens:** 12 tokens across major DeFi assets

---

*Last Updated: 2025-08-29*
*API Version: 1.0.0*
