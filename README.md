# OdinTrade API Reference

This documentation provides complete information about the API endpoints which can be used for interacting with OdinTrade's offchain orderbook.

The WebSocket API is available at: https://socket.odin.trade

## Events

### transactions

Emitted when there are new deposits/ withdrawals, similar in structure to market.transactions.

### orders

Emitted when an order is created or removed, similar in structure to market.orders.

### trades

Emitted when a trade happens, similar in structure to market.trades.

### ticks

Emitted when a new tick is recorded with an empty payload to notify the clients to reload the datafeed.

## Get all available markets

```
getAssets
```

Return all available assets

**Sample request:**

```javascript
socket.emit("getAssets");
socket.on("assets", m => {
	console.log(m);
});
```

**Sample response:**

```javascript
{
	MKR: {
		symbol: "ETH_MKR",
		name: "Maker",
		address: "0x76a86b8172886DE0810E61A75aa55EE74a26e76f"
	}
	BNB: {
		symbol: "BNB",
		name: "Binance",
		address: "0xA39071f60fa2eC4b03749dBA262dCA7f68a43D1B"
	},
	...
};
```

## Get data of a specific market

```
getMarket { symbol }
```

Return information of a specific market

**Sample request:**

```javascript
socket.emit("getMarket", { symbol: "MKR" });
socket.on("MKR", data => {
	console.log(data);
});
```

**Sample response:**

```javascript
{
	market: {
		symbol: "MKR",
		name: "Maker",
		address: "0x76a86b8172886DE0810E61A75aa55EE74a26e76f",
		sellOrders: [
			{
				user: "0x41088A3d46884444705d7603B9Ef47fD67C3541E",
				amount: 3000000000000000,
				price: 1400000000000000000,
				next: 45,
				prev: 40,
				sell: true,
				id: 30
			},
			...
		],
		buyOrders: [
			{
				user: "0x41088A3d46884444705d7603B9Ef47fD67C3541E",
				amount: 3000000000000000,
				price: 1400000000000000000,
				next: 45,
				prev: 40,
				sell: false,
				id: 30
			},
			...
		],
		trades: [
			{
		  	token: 0x76a86b8172886DE0810E61A75aa55EE74a26e76f
		  	bid: 46
		  	ask: 30
		  	price: 1400000000000000000
		  	amount: 1000000000000000
		  	timestamp: 1537436883
		  	sell: true
	    },
			...
		],
	}
}
```

## Fetch user balance

```
getBalance { userAddress }
```

Return an array in which each element contains the balance information of a token.

**Sample request:**

```javascript
socket.emit("getBalance", { userAddress });
socket.on("balance", m => {
	console.log(m);
});

**Sample reponse:**

```javascript
[
	{
		token: '0x0000000000000000000000000000000000000000',
		total: 0.18946627,
		available: 0.09221427,
		reserve: 0.097252,
	}
	...
]
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
