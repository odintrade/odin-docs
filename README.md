# OdinTrade API Reference

This documentation provides complete information about the API endpoints which can be used for interacting with OdinTrade's offchain orderbook.

The WebSocket API is available at: https://socket.odin.trade

## Emitting events

### ticks

Emitted when a new tick is recorded with an empty payload to notify the clients to reload the datafeed.

### `marketAddress`, eg: 0xA39071f6...

Emitted when a new market-related information becomes available, such as new orders or trades, response is similar to `getMarket`.

### `userAddress`, eg: 0xA39071f6...

Emitted when a new user-related information becomes available, such as new deposits or withdrawals, response is similar to `getUser`.

## Listening events

### Get all available markets

```
getMarkets
```

Return all available markets

**Sample request:**

```javascript
socket.emit("getMarkets");
socket.on("markets", res => {
	console.log(res);
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
		address: "0x8a37b79E54D69e833d79Cac3647C877Ef72830E1"
	},
	...
};
```

### Get data of a specific market

```
getMarket { symbol }
```

Return information of a specific market

**Sample request:**

```javascript
socket.emit("getMarket", { symbol: "MKR" });
socket.on("MKR", res => {
	console.log(res);
});
```

**Sample response:**

```javascript
{
	market: {
		symbol: "MKR",
		name: "Maker",
		address: "0x76a86b8172886DE0810E61A75aa55EE74a26e76f",
		orders: [
			{
				user: "0x41088A3d46884444705d7603B9Ef47fD67C3541E",
				amount: 3000000000000000,
				price: 1400000000000000000,
				next: 45,
				prev: 40,
				sell: true,
				id: 30
			},
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

### Fetch user data

```
getUser { address }
```

Return an object contains the data of a given user.

**Sample request:**

```javascript
socket.emit("getUser", { address });
socket.on("0x76a86b8172886DE0810E61A75aa55EE74a26e76f", res => {
	console.log(res);
});
```

**Sample reponse:**

```javascript
{
	user: {
		id: 1,
		address: "0x8a37b79E54D69e833d79Cac3647C877Ef72830E1",
		transfers: [
			{
				type: "deposit",
				date: "2018-08-25 21:48:29",
				asset: "VINA",
				name: "VietTrade"
				amount: "0.1",
				status: "pending",
				transactionHash: "0xb844692c9c29ae7d7cb246bacac84f8a435a402d2074a85c37bbf03af928f60f",
				blockHash: "0x55d9972705ab92ed16dcbc5491e282df2456131a9404f4b812457c23cffb535c",
				blockNumber: 371
			},
			...
		]
		wallets: {
			"VINA": "0.12345678",
			"BNB": 0.12345678
			...
		}
	},
	type: "getUser"
}
```

### Withdraw

```
withdraw { tokenAddress, amount, user, nonce, v, r, s }
```

Submit a withdraw request.

**Parameters:**

- `tokenAddress`: the address of the token to be withdrawn
- `amount`: the amount to be withdrawn
- `user`: the address of the user to withdraw from
- `nonce`: the current timestamp in milliseconds, a nonce should only be used once
- `v, r, s`: the keccak256 result of all the above, signed by `user`

**Example of obtaining the v, r, s for a withdraw message:**

```javascript
const msg = web3.utils.soliditySha3(exchangeAddress, tokenAddress, amount, user, nonce);
const sig = web3.eth.sign(user, msg);
const { v, r, s } = eutil.fromRpcSig(sig);
```

**Sample request:**

```javascript
socket.emit("withdraw", {
	tokenAddress,
	amount,
	user,
	nonce,
	v,
	r,
	s
});
socket.on("0x8a37b79E54D69e833d79Cac3647C877Ef72830E1", res => {
	console.log(res)	
})
```

**Sample response:**

```javascript
{
	user: {
		address: "0x8a37b79E54D69e833d79Cac3647C877Ef72830E1",
		"VINA": {
			available: 0.09221427,
			reserve: 0.097252
		},
		...
	},
	type: "withdraw"
}
```

### Placing limit orders

```
createOrder { maker, giveToken, giveAmount, takeToken, takeAmount, nonce, expiry, v, r, s }
```

Submit an order to the orderbook.

**Parameters:**

- `exchangeAddress`: the address of the exchange's contract
- `user`: the address of the user creating the order
- `giveToken`: the address of the token to trade away
- `takeToken`: the address of the token to receive
- `giveAmount`: the amount to trade away
- `takeAmount`: the amount to receive
- `nonce`: the current timestamp in milliseconds, a nonce should only be used once
- `expiry`: expiry time in blocks
- `v, r, s`: the keccak256 result of the above, signed by `maker`

**Example of obtaining the v, r, s for an order:**

```javascript
const order = Web3Utils.soliditySha3(
	exchangeAddress,
	user,
	giveToken,
	giveAmount,
	takeToken,
	takeAmount,
	nonce,
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

### Cancelling orders

```
cancelOrder { orderHash, user, nonce, v, r, s }
```

Cancel an order.

**Parameters:**

- `orderHash`: the hash of the order to cancel
- `user`: the address of the order's owner
- `nonce`: the current timestamp in milliseconds, a nonce should only be used once
- `v, r, s`: the keccak256 result of `orderHash` and `nonce`, signed by `user`

**Example of obtaining the v, r, s:**

```javascript
const cancelMsg = Web3Utils.soliditySha3(orderHash, nonce);
const signedCancelMsg = web3.eth.sign(user, cancelMsg);
const { v, r, s } = eutil.fromRpcSig(signedCancelMsg);
```

**Sample request:**

```javascript
socket.emit("cancel", {
	orderHash,
	user,
	nonce,
	v,
	r,
	s
});
```
