# Mediator

## General Types

```solidity
struct SafeProtocolAction {
    address to,
    uint256 value,
    bytes data
}
```

Safe transaction (invoked via `call`)

```solidity
struct SafeTransaction {
    address safe,
    SafeProtocolAction[] actions,
    uint256 nonce,
    bytes32 metaHash
}
```

Safe root access (invoked via `delegatecall`)

```solidity
struct SafeRootAccess {
    address safe,
    SafeProtocolAction action,
    uint256 nonce,
    bytes32 metaHash
}
```

Both `SafeTransaction` and `SafeRootAccess` MUST have a unique id, which is the EIP-712 hash of the object.

## Interface

```solidity
interface ISafeProtocolMediator {
    function executeTransaction(Safe safe, SafeTransaction tx) external view returns (bytes[] memory data);
    function executeRootAccess(Safe safe, SafeRootAccess rootAccess) external view returns (bytes memory data);
}
```

To handle return data for `executeTransaction` (as it is an array of actions)
- The size of `data` must equal the size of the actions array.
- Each element in `data` corresponds to the action that has been executed i.e. `data[i]` is the result of `action[i]`.
- If execution of action(s) fail, the transaction must revert.

## Uniqueness

As mentioned before it is required that both `SafeTransaction` and `SafeRootAccess` can be uniquely identified. An example where this is important is tooling related to indexing and querying information for the components. For this purpose a `nonce` field is present in the structs, which allows to make the hash calculated for these is unique.

Note: It is the responsibility of the component (i.e. module) to ensure that each of structs can be uniquely identified.

## Flow Charts

## Automatic Enforcements

## Upgradeability  

It is inevitable that more features will be added to the Safe Protocol (e.g. new components). As the Mediator is the central part of this setup, it is important to consider a path for integrating these new features. Using an upgradeable proxy for the mediator would introduce unacceptable security concerns. Separating too much of the functionality into separate contract to allow reusability (i.e. the list of enabled components) would increase the gas costs, and so is also not practical. A common and acceptable pattern is to allow a new version to load information from the previous version and thereby allow an information migration.