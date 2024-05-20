# RFC-0012: Access Control

- **Intent**: Access Control Standards for Single and Multi Signature Access
- **Submitted by**: Chris Cates, <hello@chriscates.ca>, @ChrisCates on Discord
- **Submitted on**: February 13th, 2024

## Abstract

The goal of this RFC is to standardize access control over various functions in Mina Smart Contracts. As of now, there are three core flows:

1. Single Signature

2. Multi Signature

3. Role Based Access Control

The Access Control Standard should take inspiration from Open Zeppelin formatting and standards, and, leverage Typescript decorations in order to make access control among functions easy to implement.

## Introduction

As of today, most contracts prototyped on Mina with O1JS require you to manually design, verify and implement security via signatures. The goal of the Access Control standard is to allow developers to verify access to functions with signatures through decorators. Furthermore, this standardized implementation will improve security practices across the board for all O1JS contracts. Ensuring that key sensitive functions executions are secured via decorators. The decorators are also intended to garauntee that the development experience is simplified as much as possible so that developers aren't confused managing and verifying access to functions.

## Objectives

The core goal of the Access Control Standard is as follows:

**Ensure security and access to a function is achieved with only 1 decorator.**

**Allow developers to manage ownership of a contract in a similar format to Open Zeppelin's Access Control Standard.**

**Allow for single, and multi ownership access control flows, leveraging single and multi signature verification.**

## Motivation and Rationale

The motivation of this RFC is to solidify a security standard for O1JS contracts revolved around decorators. This will enable developers to easily extend parameters of their contract logic using the Access Control interfaces, as well as secure functions with decorators.

Mina's Access Control standard should take inspiration from Open Zeppelin's Solidity Contract standard, but, also take it a step further with multi signature request access control flows.

Assuming an easy to use NPM module is developed. The Access Control standard will allow all O1JS developers to secure their applications faster and more effectively.

## Scenarios and Use Cases

### Scenario 1: An Adminstrative Contract Owner

In this context, a developer may want an adminstrative contract owner. This adminstrator may only be allowed to mint tokens for example, or, modify states in the contract.

It is required that the adminstrative contract leverages the decorators and extends interface for secured methods that require said decorator.

For context of how implementation should be designed. Here is a preview of what Single Ownership Interfaces and Decorators should look like:

```typescript
// @NOTE: Namespaces for Access Control functions should probably be shortened

// Access Control for a Single Owner Interface
export interface SingleAccess {
  signature: Signature
}

// Access Control for a Single Owner Method Decorator
export function SingleAccessDecorator(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.value = (args: SingleAccess) => {
    const signature = args.signature

    // Verify the owner key
    signature.verify(...)
  }
}

export interface SecureEndpointI extends SingleAccess {}

class Test {
  @SingleAccessDecorator
  secureEndpoint(parameters: SecureEndpointI) {
    // Secure Function Logic
  }
}
```

### Scenario 2: Multi Signature Adminstration

In this context, a developer may want multiple adminstrative contract owners instead of one. This scenario would require multiple signatures in order to execute a function.

It is required that the adminstrative contract leverages the decorators and extends interface for secured methods that require said decorator.

For context of how the implementation should be designed. Here is a preview of what Multi Ownership Interfaces and Decorators should look like:

```typescript
// @NOTE: Namespaces for Access Control functions should probably be shortened

// Access Control for a Multi Owner Interface
export interface MultiAccess {
  signatures: Signature[]
}

// Access Control for a Multi Owner Method Decorator
export function MultiAccessDecorator(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.value = (args: SingleAccess) => {
    const signatures = args.signatures
    for (const signature of signatures) {
      // Verify the owner key
      signature.verify(...)
    }
  }
}

export interface SecureEndpointI extends MultiAccess {}

class Test {
  @MultiAccessDecorator
  secureEndpoint(parameters: SecureEndpointI) {
    // Secure Function Logic
  }
}
```

### Scenario 3: Role based Access Control and Adminstration

In this context, there may be different roles with certain permissions in the contract. For example, the admin may have access to all functionality. A minter may only mint tokens. An accountant can only transfer tokens, so on and so forth.

It is required that the role based contract leverages the decorators and extends interface for secured methods that require said decorator.

For context of how the implementation should be designed. Here is a preview of what Role Based Interfaces and Decorators should look like:

```typescript
// @NOTE: Namespaces for Access Control functions should probably be shortened

// The developer defines roles
export type Role = 'admin' | 'minter' | 'accountant'

// Access Control for a Single Owner Interface
export interface RoleAccess {
  role: RoleType
  key: PublicKey
  signature: Signature
}

// Access Control for the Role based Method Decorator
export function RoleAccessDecorator(role: RoleType) {
  return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
   descriptor.value = (args: RoleAccess) => {
      if (this.AccessControl[args.key] === role) {
        const signature = args.signature
        // Verify the owner key
        signature.verify(...)
      } else {
        console.log('Incorrect Role!')
      }
    }
  }
}

export interface SecureEndpointI extends SingleAccess {}

class Test {
  @RoleAccessDecorator('minter')
  secureEndpoint(parameters: SecureEndpointI) {
    // Secure Function Logic
  }
}
```

## Conclusion

Overall, the Access Control standard, the examples and scenarios provided. Should not only **improve** the formatting and standards seen in Open Zeppelin. But, also fit the use case and needs of Mina Contracts, by leveraging signature, multi signature access control flows.

These standards being accessible in an npm module that can be imported will help developers across the board in the Mina and O1JS ecosystem. Making development easier, more accessible and most importantly, more secure!

## Appendices

### Ownership Class Extensions

The code snippets provided simply demonstrate interfaces and method decorators to verify signatures for functions in an O1JS contract.

It would be important to include other functions as seen in Open Zeppelin's Access Control standard namely the functions:

```typescript
owner()
renounceOwnership()
transferOwnership(_address: address)
```

Namespaces may be modified for multi signature access, for example:

```typescript
owners()
renounceOwnership(owner: PublicKey)
transferOwnership(owner: PublicKey, to: PublicKey)
```

These class decorators can be added to an O1JS contract as follows:

```typescript
import { SingleAccessContract } from '@mina/access-control'

export class MyO1Contract extends SingleAccessContract {
  // MyO1Contract can now user owner(), renounceOwnership() etc.
}
```

### Role Based Class Extensions

Similar to Ownership Class Extensions. It would be important to have role based class extensions. Same idea in principle.

It should include the following:

```typescript
hasRole()
grantRole()
revokeRole()
```

```typescript
import { RoleAccessContract } from '@mina/access-control'

export class MyO1Contract extends RoleAccessContract {
  // MyO1Contract can now user hasRole(), grantRole(), revokeRole() etc.
}
```

### Easy Unit Testing

It would be ideal if there was an easy way to generate unit tests for access control. Where you name the functions that have single, multi, role based access control. You name the secured functions. The unit tests will auto regression test and verify said functions are secured.

### Improving Role Based Extensions

Note that the demonstration snippets in the RFC should be improved. For example, the `admin` role should be able to access everything that a `minter` and `accountant` should. Instead of a `type` for the `RoleType` definition. Upgrading it to an interface which explicitly defines what level of access a Role has would be optimal. For example:

```typescript
export interface RoleType {
  name: string
  privileges: string[]
}

export const AdminRoleType: RoleType {
  name: 'admin',
  privileges: ['admin', 'minter', 'accountant']
}
```

## References

- [Open Zeppelin's Access Control Standards on Github](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts/access)

- [Open Zeppelin's Access Control Documentation](https://docs.openzeppelin.com/contracts/3.x/access-control)
