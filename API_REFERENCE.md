# Hyperliquid Python SDK - Complete API Reference

A comprehensive guide to the Hyperliquid Python SDK, covering all API endpoints, WebSocket subscriptions, functions, usage examples, and best practices.

## Table of Contents

- [Installation](#installation)
- [Authentication & Setup](#authentication--setup)
- [Trading APIs](#trading-apis)
  - [Order Management](#order-management)
  - [Position Management](#position-management)
  - [Order Types & Parameters](#order-types--parameters)
- [Market Data APIs](#market-data-apis)
  - [Price Data](#price-data)
  - [Order Book](#order-book)
  - [Historical Data](#historical-data)
- [Account APIs](#account-apis)
  - [User State](#user-state)
  - [Open Orders](#open-orders)
  - [Trade History](#trade-history)
- [Transfers & Withdrawals](#transfers--withdrawals)
- [Spot Trading](#spot-trading)
- [WebSocket/Real-time Data](#websocketreal-time-data)
- [Advanced Features](#advanced-features)
  - [Agent Trading](#agent-trading)
  - [Multi-Signature Wallets](#multi-signature-wallets)
  - [Staking & Delegation](#staking--delegation)
- [Utility Functions](#utility-functions)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Installation

```bash
pip install hyperliquid-python-sdk
```

For development:
```bash
git clone https://github.com/hyperliquid-dex/hyperliquid-python-sdk.git
cd hyperliquid-python-sdk
make install  # Requires Python 3.10+
```

## Authentication & Setup

### Basic Configuration

```python
from hyperliquid.utils import constants
from hyperliquid.info import Info
from hyperliquid.exchange import Exchange
from eth_account import Account

# Configuration
config = {
    "secret_key": "your_private_key_hex",  # Without 0x prefix
    "account_address": "",  # Optional, defaults to wallet address
    "base_url": constants.MAINNET_API_URL  # or TESTNET_API_URL
}

# Initialize account
account = Account.from_key(config["secret_key"])
address = config["account_address"] or account.address

# Create API instances
info = Info(config["base_url"], skip_ws=True)
exchange = Exchange(account, config["base_url"], account_address=address)
```

### API Endpoints

- **Mainnet**: `https://api.hyperliquid.xyz`
- **Testnet**: `https://api.hyperliquid-testnet.xyz`
- **Local**: `http://localhost:3001`

## Trading APIs

### Order Management

#### Place Order

Place a single limit or trigger order.

```python
def order(
    name: str,           # Asset name (e.g., "ETH") or spot pair (e.g., "PURR/USDC")
    is_buy: bool,        # True for buy, False for sell
    sz: float,           # Order size
    limit_px: float,     # Limit price
    order_type: Dict,    # Order type configuration
    reduce_only: bool = False,
    cloid: Optional[Cloid] = None,  # Client order ID
    builder: Optional[BuilderInfo] = None
) -> Dict
```

**Example Request:**
```python
# Limit buy order
order_result = exchange.order(
    "ETH",
    True,
    0.2,
    3400,
    {"limit": {"tif": "Gtc"}},  # Good Till Cancelled
    reduce_only=False,
    cloid=Cloid.from_int(12345)
)
```

**Example Response:**
```json
{
    "status": "ok",
    "response": {
        "data": {
            "statuses": [{
                "resting": {
                    "oid": 123456789,
                    "cloid": "0x0000000000000000000000000000000000000000000000000000000000003039"
                }
            }]
        }
    }
}
```

**Order Types:**
- **Limit Orders**: `{"limit": {"tif": "Gtc"}}`, `{"limit": {"tif": "Alo"}}`, `{"limit": {"tif": "Ioc"}}`
- **Trigger Orders**: `{"trigger": {"triggerPx": 3500, "isMarket": true, "tpsl": "tp"}}`

#### Place Multiple Orders

```python
def bulk_orders(
    order_requests: List[OrderRequest],
    builder: Optional[BuilderInfo] = None
) -> Dict
```

**Example:**
```python
orders = [
    {
        "a": 2,  # Asset index
        "b": True,  # is_buy
        "p": "3400",  # price
        "s": "0.1",  # size
        "r": False,  # reduce_only
        "t": {"limit": {"tif": "Gtc"}},
        "c": "0x0000000000000000000000000000000000000000000000000000000000000001"
    },
    {
        "a": 2,
        "b": True,
        "p": "3350",
        "s": "0.2",
        "r": False,
        "t": {"limit": {"tif": "Gtc"}}
    }
]
result = exchange.bulk_orders(orders)
```

#### Cancel Order

```python
# Cancel by order ID
cancel_result = exchange.cancel("ETH", oid=123456789)

# Cancel by client order ID
cancel_result = exchange.cancel_by_cloid("ETH", cloid=Cloid.from_int(12345))
```

**Response:**
```json
{
    "status": "ok",
    "response": {
        "data": {
            "statuses": ["success"]
        }
    }
}
```

#### Modify Order

```python
modify_result = exchange.modify_order(
    oid=123456789,
    name="ETH",
    is_buy=True,
    sz=0.3,  # New size
    limit_px=3450,  # New price
    order_type={"limit": {"tif": "Gtc"}},
    reduce_only=False
)
```

#### Market Orders

```python
# Market buy with slippage tolerance
market_result = exchange.market_open(
    name="ETH",
    is_buy=True,
    sz=0.05,
    px=None,  # Uses current mid price
    slippage=0.01  # 1% slippage
)

# Market close position
close_result = exchange.market_close(
    coin="ETH",
    sz=None,  # Closes entire position
    px=None,
    slippage=0.05
)
```

### Position Management

#### Update Leverage

```python
leverage_result = exchange.update_leverage(
    leverage=10,
    name="ETH",
    is_cross=True  # True for cross margin, False for isolated
)
```

**Response:**
```json
{
    "status": "ok",
    "response": {
        "data": null
    }
}
```

#### Update Isolated Margin

```python
margin_result = exchange.update_isolated_margin(
    amount=100.0,
    name="ETH"
)
```

### Order Types & Parameters

#### Time in Force (TIF) Options
- **Gtc** (Good Till Cancelled): Order remains active until filled or cancelled
- **Alo** (Add Liquidity Only): Only posts to orderbook, cancelled if would take
- **Ioc** (Immediate or Cancel): Fills immediately or cancels

#### Order Type Structure
```python
# Limit order
order_type = {
    "limit": {
        "tif": "Gtc"  # or "Alo", "Ioc"
    }
}

# Trigger order (stop/take-profit)
order_type = {
    "trigger": {
        "triggerPx": 3500.0,
        "isMarket": True,  # Execute as market order when triggered
        "tpsl": "tp"  # "tp" for take-profit, "sl" for stop-loss
    }
}
```

## Market Data APIs

### Price Data

#### Get All Mid Prices

```python
mids = info.all_mids()
```

**Response:**
```json
{
    "ETH": "3400.50",
    "BTC": "65432.10",
    "ARB": "1.2345",
    "ATOM": "11.2585"
}
```

### Order Book

#### L2 Snapshot

```python
l2_data = info.l2_snapshot("ETH")
```

**Response:**
```json
[
    [  // Bids
        {"px": "3400.0", "sz": "1.5", "n": 3},
        {"px": "3399.5", "sz": "2.0", "n": 5}
    ],
    [  // Asks
        {"px": "3401.0", "sz": "1.2", "n": 2},
        {"px": "3401.5", "sz": "3.0", "n": 4}
    ]
]
```

### Historical Data

#### Candles

```python
candles = info.candles_snapshot(
    name="ETH",
    interval="1h",  # 1m, 5m, 15m, 1h, 4h, 1d
    startTime=1684811870000,  # Unix timestamp in ms
    endTime=1684898270000
)
```

**Response:**
```json
[
    {
        "t": 1684811870000,  // Timestamp
        "o": "3380.5",       // Open
        "h": "3420.0",       // High
        "l": "3375.0",       // Low
        "c": "3410.5",       // Close
        "v": "125.5"         // Volume
    }
]
```

#### Funding History

```python
funding = info.funding_history(
    name="ETH",
    startTime=1684811870000,
    endTime=None  # Optional
)
```

**Response:**
```json
[
    {
        "coin": "ETH",
        "fundingRate": "0.0001",
        "premium": "0.0002",
        "time": 1684811870000
    }
]
```

## Account APIs

### User State

#### Get User Trading State

```python
user_state = info.user_state(address)
```

**Response:**
```json
{
    "assetPositions": [
        {
            "position": {
                "coin": "ETH",
                "entryPx": "3400.50",
                "leverage": {
                    "type": "cross",
                    "value": 10
                },
                "liquidationPx": "3100.25",
                "marginUsed": "340.05",
                "positionValue": "3400.50",
                "returnOnEquity": "0.15",
                "szi": "1.0",
                "unrealizedPnl": "50.25"
            },
            "type": "oneWay"
        }
    ],
    "marginSummary": {
        "accountValue": "10000.50",
        "totalMarginUsed": "340.05",
        "totalNtlPos": "3400.50",
        "totalRawUsd": "9660.45"
    },
    "withdrawable": "9660.45"
}
```

#### Get Spot Balances

```python
spot_state = info.spot_user_state(address)
```

**Response:**
```json
{
    "balances": [
        {
            "coin": "USDC",
            "hold": "100.0",
            "token": 0,
            "total": "1000.0"
        },
        {
            "coin": "PURR",
            "hold": "0.0",
            "token": 1,
            "total": "500.0"
        }
    ]
}
```

### Open Orders

```python
open_orders = info.open_orders(address)
```

**Response:**
```json
[
    {
        "coin": "ETH",
        "limitPx": "3300.0",
        "oid": 123456789,
        "side": "B",
        "sz": "0.5",
        "timestamp": 1684811870000,
        "origSz": "0.5",
        "cloid": "0x0000000000000000000000000000000000000000000000000000000000003039"
    }
]
```

### Trade History

#### Get User Fills

```python
fills = info.user_fills(address)
```

**Response:**
```json
[
    {
        "coin": "ETH",
        "px": "3400.50",
        "sz": "0.2",
        "side": "B",
        "time": 1684811870000,
        "startPosition": "0.0",
        "dir": "Open Long",
        "closedPnl": "0.0",
        "hash": "0x...",
        "oid": 123456789,
        "crossed": true,
        "fee": "0.68",
        "liquidation": false,
        "cloid": "0x0000000000000000000000000000000000000000000000000000000000003039"
    }
]
```

#### Get Fills by Time Range

```python
fills_by_time = info.user_fills_by_time(
    address,
    start_time=1684811870000,
    end_time=1684898270000
)
```

## Transfers & Withdrawals

### USD Transfer

```python
transfer_result = exchange.usd_transfer(
    amount=100.0,
    destination="0x0000000000000000000000000000000000000000"
)
```

**Response:**
```json
{
    "status": "ok",
    "response": {
        "data": null
    }
}
```

### Spot Token Transfer

```python
spot_transfer_result = exchange.spot_transfer(
    amount=50.0,
    destination="0x0000000000000000000000000000000000000000",
    token="USDC"
)
```

### Withdraw from Bridge

```python
withdraw_result = exchange.withdraw_from_bridge(
    amount=1000.0,
    destination="0x0000000000000000000000000000000000000000"
)
```

### Internal Transfers

#### Between Spot and Perp

```python
# Transfer from perp to spot
class_transfer = exchange.usd_class_transfer(
    amount=500.0,
    to_perp=False  # False = to spot, True = to perp
)
```

#### Sub-account Transfers

```python
# Transfer to sub-account
sub_transfer = exchange.sub_account_transfer(
    sub_account_user="0x...",
    is_deposit=True,  # True = deposit to sub, False = withdraw from sub
    usd=100  # Amount in cents
)
```

## Spot Trading

### Spot Order Format

Spot trading uses the same order functions with spot pair notation:

```python
# Using pair notation
spot_order = exchange.order(
    "PURR/USDC",  # Spot pair format
    True,         # is_buy
    100,          # size
    0.5,          # price
    {"limit": {"tif": "Gtc"}}
)

# Using index notation
spot_order = exchange.order(
    "@8",  # Asset index from spot metadata
    True,
    100,
    0.5,
    {"limit": {"tif": "Gtc"}}
)
```

### Get Spot Metadata

```python
spot_meta = info.spot_meta()
```

**Response:**
```json
{
    "tokens": [
        {
            "name": "USDC",
            "szDecimals": 6,
            "weiDecimals": 6,
            "index": 0,
            "tokenId": "0x...",
            "isCanonical": true
        },
        {
            "name": "PURR",
            "szDecimals": 18,
            "weiDecimals": 18,
            "index": 1,
            "tokenId": "0x...",
            "isCanonical": false
        }
    ],
    "universe": [
        {
            "name": "PURR/USDC",
            "tokens": [1, 0],
            "index": 8,
            "isCanonical": true
        }
    ]
}
```

## WebSocket/Real-time Data

### Available Subscriptions

```python
# 1. All mid prices
info.subscribe({"type": "allMids"}, callback)

# 2. Best bid/offer for a coin
info.subscribe({"type": "bbo", "coin": "ETH"}, callback)

# 3. L2 orderbook updates
info.subscribe({"type": "l2Book", "coin": "ETH"}, callback)

# 4. Trade feed
info.subscribe({"type": "trades", "coin": "ETH"}, callback)

# 5. User events (fills, liquidations, etc.)
info.subscribe({"type": "userEvents", "user": address}, callback)

# 6. User fills only
info.subscribe({"type": "userFills", "user": address}, callback)

# 7. Candlestick data
info.subscribe({"type": "candle", "coin": "ETH", "interval": "1m"}, callback)

# 8. Order updates
info.subscribe({"type": "orderUpdates", "user": address}, callback)

# 9. User funding payments
info.subscribe({"type": "userFundings", "user": address}, callback)

# 10. Non-funding ledger updates
info.subscribe({"type": "userNonFundingLedgerUpdates", "user": address}, callback)

# 11. Web data feed
info.subscribe({"type": "webData2", "user": address}, callback)
```

### WebSocket Message Format

```python
def handle_message(message):
    print(f"Channel: {message['channel']}")
    print(f"Data: {message['data']}")
    
# Example message
{
    "channel": "l2Book",
    "data": {
        "coin": "ETH",
        "time": 1684811870000,
        "levels": [
            [{"px": "3400.0", "sz": "1.5", "n": 3}],
            [{"px": "3401.0", "sz": "1.2", "n": 2}]
        ]
    }
}
```

### Managing WebSocket Connection

```python
# Start WebSocket (if not using skip_ws)
info = Info(base_url, skip_ws=False)

# Subscribe to channels
subscription_id = info.subscribe({"type": "trades", "coin": "ETH"}, callback)

# Unsubscribe
info.unsubscribe({"type": "trades", "coin": "ETH"}, subscription_id)

# Disconnect
info.disconnect_websocket()
```

## Advanced Features

### Agent Trading

Agents can trade on behalf of users but cannot transfer funds.

```python
# Approve agent and get agent key
agent_result, agent_key = exchange.approve_agent()

# Create agent account
agent_account = Account.from_key(agent_key)
agent_exchange = Exchange(
    agent_account, 
    base_url, 
    account_address=address  # Original user's address
)

# Agent can now trade
agent_order = agent_exchange.order("ETH", True, 0.1, 3400, {"limit": {"tif": "Gtc"}})
```

### Multi-Signature Wallets

#### Convert to Multi-Sig

```python
multi_sig_result = exchange.convert_to_multi_sig_user(
    authorized_users=[
        "0x1111111111111111111111111111111111111111",
        "0x2222222222222222222222222222222222222222"
    ],
    threshold=2  # Number of signatures required
)
```

#### Execute Multi-Sig Action

```python
# Each signer creates their signature
inner_action = {
    "type": "order",
    "orders": [{
        "a": 2,  # Asset index
        "b": True,
        "p": "3400",
        "s": "0.1",
        "r": False,
        "t": {"limit": {"tif": "Gtc"}}
    }]
}

# Collect signatures from required signers
signatures = [sig1, sig2]

# Execute multi-sig transaction
result = exchange.multi_sig(
    multi_sig_user=multi_sig_address,
    inner_action=inner_action,
    signatures=signatures,
    nonce=current_nonce
)
```

### Staking & Delegation

#### Delegate Tokens

```python
# Delegate to validator
delegate_result = exchange.token_delegate(
    validator="0x...",
    wei=1000000000000000000,  # 1 token in wei
    is_undelegate=False
)
```

#### Query Staking Info

```python
# User staking summary
staking_summary = info.user_staking_summary(address)

# Staking delegations
delegations = info.user_staking_delegations(address)

# Staking rewards history
rewards = info.user_staking_rewards(address)
```

### Spot Token Deployment

```python
# Register new token
register_result = exchange.spot_deploy_register_token(
    token_name="MYTOKEN",
    sz_decimals=18,
    wei_decimals=18,
    max_gas=200000,
    full_name="My Token"
)

# Deploy genesis with initial distribution
genesis_result = exchange.spot_deploy_user_genesis(
    token=token_index,
    user_and_wei=[
        ["0x111...", "1000000000000000000000"],  # 1000 tokens
        ["0x222...", "500000000000000000000"]    # 500 tokens
    ],
    existing_token_and_wei=[[0, "100000000"]]  # 100 USDC
)
```

## Utility Functions

### Client Order ID (Cloid)

```python
from hyperliquid.utils.types import Cloid

# Create from integer
cloid = Cloid.from_int(12345)

# Create from hex string
cloid = Cloid.from_str("0x3039")

# Use in order
order_result = exchange.order("ETH", True, 0.1, 3400, {"limit": {"tif": "Gtc"}}, cloid=cloid)
```

### Constants

```python
from hyperliquid.utils import constants

# API URLs
mainnet_url = constants.MAINNET_API_URL  # https://api.hyperliquid.xyz
testnet_url = constants.TESTNET_API_URL  # https://api.hyperliquid-testnet.xyz
local_url = constants.LOCAL_API_URL      # http://localhost:3001
```

### Signing Utils

```python
from hyperliquid.utils.signing import sign_typed_data

# Sign typed data (used internally)
signature = sign_typed_data(typed_data, account)
```

## Error Handling

### Error Types

```python
from hyperliquid.utils.error import ClientError, ServerError

try:
    result = exchange.order("ETH", True, 0.1, 3400, {"limit": {"tif": "Gtc"}})
except ClientError as e:
    # Client errors (4xx)
    print(f"Client error: {e}")
except ServerError as e:
    # Server errors (5xx)
    print(f"Server error: {e}")
```

### Common Error Responses

```python
# Insufficient margin
{
    "status": "ok",
    "response": {
        "data": {
            "statuses": [{
                "error": "Insufficient margin for order"
            }]
        }
    }
}

# Invalid price
{
    "status": "ok",
    "response": {
        "data": {
            "statuses": [{
                "error": "Price precision exceeded"
            }]
        }
    }
}

# Rate limit
{
    "status": "error",
    "response": {
        "error": "Rate limit exceeded"
    }
}
```

### Error Handling Pattern

```python
def safe_order(exchange, coin, is_buy, sz, px, order_type):
    try:
        # Check account value first
        user_state = info.user_state(exchange.address)
        account_value = float(user_state["marginSummary"]["accountValue"])
        
        if account_value < sz * px * 0.1:  # 10x leverage check
            raise ValueError("Insufficient account value for order")
        
        # Place order
        result = exchange.order(coin, is_buy, sz, px, order_type)
        
        if result["status"] != "ok":
            raise Exception(f"Order failed: {result}")
            
        # Check order status
        for status in result["response"]["data"]["statuses"]:
            if "error" in status:
                raise Exception(f"Order error: {status['error']}")
            elif "resting" in status:
                return status["resting"]["oid"]
            elif "filled" in status:
                return status["filled"]
                
    except ClientError as e:
        print(f"Client error: {e}")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None
```

## Rate Limiting

### Rate Limits

- **Orders**: 10 requests per second
- **Cancels**: 100 requests per second
- **Info requests**: 20 requests per second
- **WebSocket subscriptions**: 10 per connection

### Rate Limit Handling

```python
import time
from typing import List

class RateLimiter:
    def __init__(self, max_requests: int, time_window: float):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests: List[float] = []
    
    def wait_if_needed(self):
        now = time.time()
        # Remove old requests
        self.requests = [t for t in self.requests if now - t < self.time_window]
        
        if len(self.requests) >= self.max_requests:
            sleep_time = self.time_window - (now - self.requests[0])
            if sleep_time > 0:
                time.sleep(sleep_time)
        
        self.requests.append(now)

# Usage
order_limiter = RateLimiter(10, 1.0)  # 10 requests per second

for order in orders:
    order_limiter.wait_if_needed()
    exchange.order(...)
```

## Best Practices

### 1. Connection Management

```python
# Use context manager pattern
class HyperliquidClient:
    def __init__(self, config):
        self.info = None
        self.exchange = None
        self.config = config
        
    def __enter__(self):
        account = Account.from_key(self.config["secret_key"])
        self.info = Info(self.config["base_url"], skip_ws=False)
        self.exchange = Exchange(account, self.config["base_url"])
        return self
        
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.info:
            self.info.disconnect_websocket()

# Usage
with HyperliquidClient(config) as client:
    # Use client.info and client.exchange
    pass
```

### 2. Order Precision

```python
# Proper decimal handling
from decimal import Decimal, ROUND_DOWN

def format_price(price: float, tick_size: str = "0.1") -> str:
    """Format price according to tick size"""
    tick = Decimal(tick_size)
    price_decimal = Decimal(str(price))
    return str(price_decimal.quantize(tick, rounding=ROUND_DOWN))

def format_size(size: float, sz_decimals: int) -> str:
    """Format size according to asset decimals"""
    multiplier = 10 ** sz_decimals
    return str(int(size * multiplier) / multiplier)
```

### 3. Idempotent Orders

```python
import uuid

def place_order_idempotent(exchange, coin, is_buy, sz, px):
    """Place order with idempotency"""
    # Generate unique client order ID
    cloid = Cloid.from_int(int(uuid.uuid4().hex[:16], 16))
    
    # First, check if order already exists
    open_orders = info.open_orders(exchange.address)
    for order in open_orders:
        if order.get("cloid") == cloid.to_raw():
            return order["oid"]
    
    # Place new order
    result = exchange.order(coin, is_buy, sz, px, {"limit": {"tif": "Gtc"}}, cloid=cloid)
    return result
```

### 4. WebSocket Reconnection

```python
import asyncio

class ReconnectingWebSocket:
    def __init__(self, info, subscriptions):
        self.info = info
        self.subscriptions = subscriptions
        self.running = True
        
    async def maintain_connection(self):
        while self.running:
            try:
                # Resubscribe to all channels
                for sub, callback in self.subscriptions:
                    self.info.subscribe(sub, callback)
                    
                # Wait for disconnect
                await asyncio.sleep(30)  # Heartbeat check
                
            except Exception as e:
                print(f"WebSocket error: {e}")
                await asyncio.sleep(5)  # Reconnect delay
                
                # Recreate connection
                self.info = Info(self.info.base_url, skip_ws=False)
```

### 5. Position Risk Management

```python
def check_position_risk(info, address, coin):
    """Check position risk metrics"""
    user_state = info.user_state(address)
    
    for asset_position in user_state["assetPositions"]:
        position = asset_position["position"]
        if position["coin"] == coin:
            leverage = position["leverage"]["value"]
            liq_px = float(position["liquidationPx"])
            entry_px = float(position["entryPx"])
            current_px = float(info.all_mids()[coin])
            
            # Calculate distance to liquidation
            if float(position["szi"]) > 0:  # Long
                liq_distance = (current_px - liq_px) / current_px
            else:  # Short
                liq_distance = (liq_px - current_px) / current_px
            
            return {
                "leverage": leverage,
                "liquidation_distance": liq_distance,
                "unrealized_pnl": position["unrealizedPnl"],
                "margin_used": position["marginUsed"]
            }
    
    return None
```

## Troubleshooting

### Common Issues

#### 1. Authentication Errors

**Problem**: "Invalid signature" errors

**Solution**:
```python
# Ensure private key is correctly formatted
private_key = "your_key_here"  # Without 0x prefix
if private_key.startswith("0x"):
    private_key = private_key[2:]

# Verify account creation
account = Account.from_key(private_key)
print(f"Address: {account.address}")
```

#### 2. Insufficient Margin

**Problem**: Orders rejected due to insufficient margin

**Solution**:
```python
# Check available margin before ordering
user_state = info.user_state(address)
margin_summary = user_state["marginSummary"]
available = float(margin_summary["accountValue"]) - float(margin_summary["totalMarginUsed"])

# Calculate required margin
required_margin = order_size * price / leverage
if available < required_margin:
    print(f"Need ${required_margin:.2f}, have ${available:.2f}")
```

#### 3. WebSocket Connection Issues

**Problem**: WebSocket disconnects frequently

**Solution**:
```python
# Use skip_ws=True for short operations
info = Info(base_url, skip_ws=True)

# Or implement reconnection logic
def reconnect_websocket(info):
    try:
        info.disconnect_websocket()
    except:
        pass
    
    # Create new instance
    return Info(base_url, skip_ws=False)
```

#### 4. Order Precision Errors

**Problem**: "Price precision exceeded" errors

**Solution**:
```python
# Use proper decimal places
def get_price_decimals(coin):
    # Perps use 6 decimals
    if "/" not in coin:
        return 6
    # Spot uses 8 decimals
    return 8

decimals = get_price_decimals("ETH")
formatted_price = f"{price:.{decimals}f}".rstrip("0").rstrip(".")
```

#### 5. Rate Limit Errors

**Problem**: "Rate limit exceeded" errors

**Solution**:
```python
# Implement exponential backoff
import time

def retry_with_backoff(func, max_retries=3):
    for i in range(max_retries):
        try:
            return func()
        except ServerError as e:
            if "rate limit" in str(e).lower():
                wait_time = 2 ** i
                time.sleep(wait_time)
            else:
                raise
    raise Exception("Max retries exceeded")
```

### Debug Mode

Enable detailed logging:

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Log all requests/responses
class DebugExchange(Exchange):
    def post(self, endpoint, payload):
        logging.debug(f"Request: {endpoint} - {payload}")
        result = super().post(endpoint, payload)
        logging.debug(f"Response: {result}")
        return result
```

---

Generated with [Claude Code](https://claude.ai/code)