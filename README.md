# OdinTrade API Reference

This documentation provides complete information about the API endpoints which can be used for interacting with OdinTrade's offchain orderbook.

The WebSocket API is available at: https://socket.odin.trade

## Events

### funds

Emitted when there are changes in user.balances, similar in structure to user.balances.

### orders

Emitted when an order is created or removed, similar in structure to market.orders.

### trades

Emitted when a trade happens, similar in structure to market.trades.

## Get all available markets

```
getMarkets
```

Return all available markets and their current prices

**Sample request:**

```javascript
socket.emit("getMarkets");
socket.on("markets", m => {
	console.log(m);
});
```

**Sample response:**

```javascript
[
	{
		symbol: "ETH_REM",
		bid: 0.00002392,
		ask: 0.00002412
	}
	{
		symbol: "ETH_DELTA",
		bid: 0.00000025,
		ask: 0.00000028
	},
	...
];
```

## Get data of a specific market

```
getMarket { symbol, user (address | optional) }
```

Return market information and user data related to the market

**Sample request:**

```javascript
socket.emit("getMarket", { symbol, user });
socket.on("market", m => {
	console.log(m);
});
```

**Sample response:**

```javascript
{
	market: {
		pair: "ETH_REM",
		high: 0.000024909999983821,
		low: 0.000022576272324766,
		ask: 0.000023399999999977,
		bid: 0.00002276,
		percentChange: -6.47382142,
		volume: 357.803939065858605263,
		orders: {
			sell: {
				[
					{
						maker: "0x8a37b79e54d69e833d79cac3647c877ef72830e1",
						price: 0.00002321,
						amount: 39990.184,
						total: 0.94791529,
						timestamp: 1534949733008,
						sell: true,
						hash: "0xa6fd3f8fd7331b2b41d47d60a93f5a11e3b94a0974a58f17b56a66861ff5d770"
					},
					...
				],
			},
			buy: {
				[
					{
						maker: "0x8a37b79e54d69e833d79cac3647c877ef72830e1",
						price: 0.00002321,
						amount: 39990.184,
						total: 0.94791529,
						timestamp: 1534949733008,
						sell: true,
						hash: "0xa6fd3f8fd7331b2b41d47d60a93f5a11e3b94a0974a58f17b56a66861ff5d770"
					},
					...
				],
			}
		}
		trades: [
			{
				buyer: "0x8a37b79e54d69e833d79cac3647c877ef72830e1",
				seller: "0x4e30dba9762aba125f5ab81647edebff9f9df7a7",
				price: 0.00002321,
				amount: 39990.184,
				total: 0.94791529,
				timestamp: 1534949733008,
				sell: true
			},
			...
		]
	},
	user: {
		balances: {
			baseTotal: 0.18946627,
			baseAvailable: 0.09221427,
			baseReserve: 0.097252,
			tokenTotal: 21000,
			tokenAvailable: 15000,
			tokenReserve: 6000,
		}
	}
}
```

## Withdraw

```
withdraw { tokenAddress, amount, account, nonce, v, r, s }
```

Submit a withdraw request.

**Parameters:**

- `tokenAddress`: the address of the token to be withdrawn
- `amount`: the amount to be withdrawn
- `account`: the address of the account to withdraw from
- `nonce`: current timestamp
- `v, r, s`: the keccak256 result of the above, signed by `account`

**Example of obtaining the v, r, s for a withdraw message:**

```javascript
const msg = Web3Utils.soliditySha3(tokenAddress, amount, account, nonce);
const signedMsg = web3.eth.sign(account, msg);
const { v, r, s } = eutil.fromRpcSig(signedMsg);
```

**Sample request:**

```javascript
socket.emit("withdraw", {
	tokenAddress,
	amount,
	account,
	nonce,
	v,
	r,
	s
});
```

## Placing limit orders

```
order { maker, giveToken, giveAmount, takeToken, takeAmount, nonce, expiry, v, r, s }
```

Submit an order to the orderbook.

**Parameters:**

- `maker`: the address of the account creating the order
- `giveToken`: the address of the token to trade away
- `takeToken`: the address of the token to receive
- `giveAmount`: the amount to trade away
- `takeAmount`: the amount to receive
- `nonce`: current timestamp
- `expiry`: expiry time in blocks
- `v, r, s`: the keccak256 result of the above, signed by `maker`

**Example of obtaining the v, r, s for an order:**

```javascript
const order = Web3Utils.soliditySha3(
	maker,
	giveToken,
	giveAmount,
	takeToken,
	takeAmount,
	makerNonce,
	expiry
);
const signedOrder = web3.eth.sign(maker, order);
const { v, r, s } = eutil.fromRpcSig(signedOrder);
```

**Sample request:**

```javascript
socket.emit("order", {
	maker,
	giveToken,
	giveAmount,
	takeToken,
	takeAmount,
	nonce,
	expiry,
	v,
	r,
	s
});
```

## Trade

```
trade [{ orderHash, taker, amount, nonce, v, r, s }]
```

Trade or "fill" an order or mutiple orders, this event is emitted with an array, each object in the array represents a trade and has to be signed individually. This operation is atomic, meaning when the array contains multiple trades, all is completed or none is completed.

**Parameters:**

- `orderHash`: the hash of the order to fill
- `taker`: the address of the account making the trade
- `amount`: the amount to be traded
- `nonce`: current timestamp
- `v, r, s`: the keccak256 result of the above, signed by `taker`

**Example of obtaining the v, r, s:**

```javascript
const trade = Web3Utils.soliditySha3(orderHash, taker, amount, takerNonce);
const signedTrade = web3.eth.sign(taker, trade);
const { v, r, s } = eutil.fromRpcSig(signedTrade);
```

**Sample request:**

```javascript
socket.emit("trade", {
	orderHash,
	taker,
	amount,
	nonce,
	v,
	r,
	s
});
```

## Cancelling orders

```
cancel { orderHash, account, nonce, v, r, s }
```

Cancel an order.

**Parameters:**

- `orderHash`: the hash of the order to fill
- `account`: the address of the account owns the order
- `nonce`: current timestamp
- `v, r, s`: the keccak256 result of `orderHash` and `nonce`, signed by `account`

**Example of obtaining the v, r, s:**

```javascript
const cancelMsg = Web3Utils.soliditySha3(orderHash, nonce);
const signedCancelMsg = web3.eth.sign(account, cancelMsg);
const { v, r, s } = eutil.fromRpcSig(signedCancelMsg);
```

**Sample request:**

```javascript
socket.emit("cancel", {
	orderHash,
	account,
	nonce,
	v,
	r,
	s
});
```
