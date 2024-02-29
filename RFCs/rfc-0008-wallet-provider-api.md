# RFC-0008: Mina Provider JavaScript API

- **Intent**: Introduces a first, extensible standard JavaScript Mina Provider API for consistency across clients and applications.
- **Submitted by**: Theodore Pender (Github: @teddyjfpender, email: theodore.pender@minaprotocol.com, Twitter/X: @franklyteddy)
- **Submitted on**: Monday, January 15, 2024

## Abstract

This RFC looks to formalizes a Mina Provider API to promote wallet interoperability within the Mina ecosystem. The API is designed to be minimal, event-driven, and agnostic of transport and RPC protocols.

## Specification

### Definitions

- **Provider**: A JavaScript object that provides access to Mina by means of a Client.
- **Client**: An endpoint that receives Remote Procedure Call (RPC) requests from the Provider and returns their results.
- **Wallet**: An application that manages private keys and acts as a middleware between the Provider and the Client.

### Connectivity

The Provider is "connected" when it can service RPC requests to at least one chain.

### API

#### request

The `request` method is a transport- and protocol-agnostic wrapper function for Remote Procedure Calls (RPCs).

```typescript
interface RequestArguments {
  readonly method: string;
  readonly params?: readonly unknown[] | object;
}

Provider.request(args: RequestArguments): Promise<unknown>;
```

### Supported RPC Methods

A "supported RPC method" is any RPC method that may be called via the Provider.

All supported RPC methods MUST be identified by unique strings. The unique identifiers must be consistent across all wallet providers.

Providers MAY support whatever RPC methods required to fulfill their purpose, standardized or otherwise.

If an RPC method defined in a finalized RFC (or MIP) is not supported, it SHOULD be rejected with a `4200` error per the Provider Errors section below, or an appropriate error per the RPC method's specification.

### RPC Errors

```typescript
interface ProviderRpcError extends Error {
  message: string;
  code: number;
  data?: unknown;
}
```

#### Error Standards

`ProviderRpcError` codes and messages should follow these conventions:ß◊
- `4001`: User Rejected Request
- `4100`: Unauthorized
- `4200`: Unsupported Method
- `4900`: Disconnected
- `4901`: Chain Disconnected

### Events

The Provider **MUST** implement the following event handling methods:
- `on` 
- `removeListener` methods. 

These methods MUST be implemented per the Node.js EventEmitter API.


It also should handle the `mina_message` and `connect` events.

#### mina_message

This event is for arbitrary notifications. When emitted, the `mina_message` event must be with an object argument:

```typescript
interface ProviderMessage {
  readonly type: string;
  readonly data: unknown;
}
```

#### connect

The connect event must be emitted with an object:

```typescript
interface ProviderConnectInfo {
  readonly chainId: string;
}
```

#### disconnect

The Provider is said to be "disconnected" when it cannot service RPC requests to any chain at all.

If the Provider becomes disconnected from all chains, the Provider MUST emit the event named `disconnect` with value error: ProviderRpcError, per the interfaced defined in the RPC Errors section. The value of the error's code property MUST follow the status codes for CloseEvent.

#### chainChanged

If the chain the Provider is connected to changes, the Provider MUST emit the event named `chainChanged` with value `chainId: string`, specifying the integer ID of the new chain as a hexadecimal string, per the `mina_chainId` RPC method.

#### accountsChanged

If the accounts available to the Provider change, the Provider MUST emit the event named `accountsChanged` with value `accounts: string[]`, containing the account addresses per the `mina_accounts` RPC method.

The "accounts available to the Provider" change when the return value of `mina_accounts` changes.

### Rationale
The purpose of a Provider is to _provide_ a consumer with access to Mina. In general, a Provider must enable a Mina web application to do two things:
- Query Mina & the wallet with requests
- Respond to state changes in the Provider's Mina chain, Client, and Wallet

The Provider API specification consists of a single method and five events. The request method and the message event alone, are sufficient to implement a complete Provider. They are designed to make arbitrary RPC requests and communicate arbitrary messages, respectively.

The remaining four events can be separated into two categories:

- Changes to the Provider's ability to make RPC requests
    - `connect`
    - `disconnect`
- Common Client and/or Wallet state changes that any non-trivial application must handle
    - `chainChanged`
    - `accountsChanged`

### Security Considerations

The Provider is intended to pass messages between a Mina Client and a Mina application. It is not responsible for private key or account management; it merely processes RPC messages and emits events. Consequently, account security and user privacy need to be implemented in middlewares between the Provider and its Mina Client. In practice, we call these middleware applications "Wallets," and they usually manage the user's private keys and accounts. The Provider can be thought of as an extension of the Wallet, exposed in an untrusted environment, under the control of some third party (e.g. a website).

#### Handling Adversarial Behavior

Since it is a JavaScript object, consumers can generally perform arbitrary operations on the Provider, and all its properties can be read or overwritten. Therefore, it is best to treat the Provider object as though it is controlled by an adversary. It is paramount that the Provider implementer protects the user, Wallet, and Client by ensuring that:

- The Provider does not contain any private user data.
- The Provider and Wallet programs are isolated from each other.
- The Wallet and/or Client rate-limit requests from the Provider.
- The Wallet and/or Client validate all data sent from the Provider.

### Provider RPC Methods
This section lists the standard RPC methods that can be supported by the Provider, along with their return types. Each method follows the `mina_{methodName}` naming convention.

#### mina_accounts
Returns a list of addresses owned by client.

##### Parameters
None

##### Returns
An array of hexadecimals as strings representing the addresses owned by the client.

```typescript
Provider.request({ 
  method: 'mina_accounts'
}): Promise<string[]>;
```

#### mina_chainId
Returns the chain ID of the current network.

##### Parameters
None

##### Returns
A hexadecimal of the current chain ID.

```typescript
Provider.request({
    method: 'mina_chainId'
}): Promise<string>;
```

#### mina_sign
Returns a signed message.

##### Parameters
`message`: [Required] a string

##### Returns
A signed message.

```typescript
type SignedMessage = {
    publicKey: string;
    data: string;
    signature: {
        field: string;
        scalar: string;
    };
};

Provider.request({
    method: 'mina_sign',
    params: { message: "hi bob" }
}): Promise<SignedMessage>;
```

#### mina_signFields
Returns a signed message.

##### Parameters
`fields`: [Required] an array of numbers or strings

##### Returns

```typescript
type SignedFieldsData = {
    publicKey: string;
    data: (string | number)[];
    signature: {
        field: string;
        scalar: string;
    };
};

Provider.request({
    method: 'mina_signFields',
    params: { fields: [1, 5, 9, 2] }
}): Promise<SignedFieldsData>;
```

#### mina_signTransaction
Returns a signed transaction.

##### Parameters
`transaction`: [Required] a well-formed transaction body.

##### Returns
A signed transaction body.

```typescript
Provider.request({
    method: 'mina_signTransaction',
    params: { transaction: tx}
}): Promise<SignedTransaction>;
```

#### mina_sendTransaction
Submits a signed transaction to the network.

##### Parameters
- `signedTransaction`: [Required] the signed transaction body
- `transactionBody`: [Required] the transaction body
- `transactionType`: [Required] the type of the transaction (e.g. `payment`, `delegation`, `zkapp`)

```typescript
type TransactionReceipt = {
    hash: string
};

Provider.request({
    method: 'mina_sendTransaction',
    params: { signedTransaction: signedTx, transactionBody: txBody, transactionType: 'payment'}
}): Promise<TransactionReceipt>
```
#### mina_getBalance
Returns the balance of the the client's current account.

##### Parameters
None.

##### Returns
```typescript
Provider,request({
    method: 'mina_getBalance'
}): Promise<number>
```

#### mina_createNullifier
Returns a nullifier.

##### Parameters
`message`: [Required] an array of `bigint`s

##### Returns
```typescript
type Nullifier = {
    publicKey: Group;
    public: {
        nullifier: Group;
        s: Field;
    };
    private: {
        c: Field;
        g_r: Group;
        h_m_pk_r: Group;
    };
};

Provider.request({
    method: 'mina_createNullifier',
    params: { message: [1n] }
}): Promise<Nullifier>
```


## Appendix I: Consumer-Facing API Documentation

### request

Makes a Mina RPC method call.

```typescript
interface RequestArguments {
  readonly method: string;
  readonly params?: readonly unknown[] | object;
}

Provider.request(args: RequestArguments): Promise<unknown>;
```

The returned Promise resolves with the method's result or rejects with a [`ProviderRpcError`](#errors). For example:

```javascript
Provider.request({ method: 'mina_accounts' })
  .then((accounts) => console.log(accounts))
  .catch((error) => console.error(error));
```

### Events

Events follow the conventions of the Node.js [`EventEmitter` API](https://nodejs.org/api/events.html).

#### connect

The Provider emits `connect` when it:

- first connects to a chain after being initialized.
- first connects to a chain, after the `disconnect` event was emitted.

```typescript
interface ProviderConnectInfo {
  readonly chainId: string;
}

Provider.on('connect', listener: (connectInfo: ProviderConnectInfo) => void): Provider;
```

The event emits an object with a hexadecimal string `chainId` per the `mina_chainId` Mina GraphQL query result, and other properties as determined by the Provider.

#### disconnect

The Provider emits `disconnect` when it becomes disconnected from all chains.

```typescript
Provider.on('disconnect', listener: (error: ProviderRpcError) => void): Provider;
```

This event emits a [`ProviderRpcError`](#rpc-errors). The error `code` follows the table of [`CloseEvent` status codes](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent#Status_codes).

#### chainChanged

The Provider emits `chainChanged` when connecting to a new chain.

```typescript
Provider.on('chainChanged', listener: (chainId: string) => void): Provider;
```

The event emits a hexadecimal string `chainId` per the `mina_chainId` Mina GraphQL query result.

#### accountsChanged

The Provider emits `accountsChanged` if the accounts returned from the Provider (`mina_accounts`) change.

```typescript
Provider.on('accountsChanged', listener: (accounts: string[]) => void): Provider;
```

The event emits with `accounts`, an array of account addresses, per the `mina_accounts` RPC method.

#### message

The Provider emits `message` to communicate arbitrary messages to the consumer.
Messages may include JSON-RPC notifications, GraphQL subscriptions, and/or any other event as defined by the Provider.

```typescript
interface ProviderMessage {
  readonly type: string;
  readonly data: unknown;
}

Provider.on('message', listener: (message: ProviderMessage) => void): Provider;
```
