# RFC-0003: Asynchronous Circuits in `o1js`

- **Intent**: To introduce asynchronous circuit in `o1js`, allowing for improved zkApp design patterns and improved program concurrency in circuit development.
- **Submitted by**: Theodore Pender (Github: @teddyjfpender, email: theodore.pender@minaprotocol.com, Twitter/X: @franklyteddy)
- **Submitted on**: Monday, November 27, 2023

## Abstract

This RFC proposes the implementation of asynchronous circuits and thus asynchronous circuit witnesses in `o1js`, addressing the limitations of sequential execution in zkApp development. The proposed feature enables circuits to pause and await data fetched asynchronously, thus enhancing the richness of zkApp design patterns and improving the efficiency and concurrency of program execution.

## Introduction

Circuits built in `o1js` currently cannot await the result of asynchronous computations and are constrained by sequential execution, limiting the ability to integrate dynamic data not known at the start of execution. This RFC proposes adding  asynchronous circuit as a solution, permitting circuits to pause their execution and resume upon receiving necessary data, be it from API calls, other circuits, or other sources of data.

Problem: Circuits cannot await the result of asynchronous computations and therefore `o1js` users must find solutions outside that are potentially error-prone and hinder brilliant developer experiences. 
Solution: Implementing asynchronous circuits in `o1js` to allow circuits to pause and resume, integrating data from external sources.

## Objectives

- Demonstrate how this feature enhances program concurrency and expands zkApp design possibilities.
- Discuss the potential impact on zkApp & circuit development and the wider `Mina` ecosystem.

## Motivation and Rationale

- Current Limitations: The inability to integrate asynchronous data sources in zk-circuit development restricts program efficiency and design flexibility.
- Proposed Benefits:
    - Enhanced Efficiency: By allowing circuits to pause and resume, execution time can be optimized, avoiding unnecessary delays.
    - Richer Design Patterns: The feature enables more complex and dynamic zkApp designs, leveraging real-time data integration.
- Community Feedback:
    - [Issue #735](https://github.com/o1-labs/o1js/issues/735) in o1js GitHub repository highlights the demand for this feature among developers.

## Scenarios and Use Cases

### Example Scenario 1: Dynamic Data Integration in a Smart Contract Circuit

Consider a generic smart contract circuit that requires real-time data integration for its operation. This data could be anything from user profiles to environmental metrics, which are not known at the start of the contract's execution. With the current synchronous model, the contract must rely on pre-fetched data, which may quickly become outdated or incomplete. Implementing asynchronous circuit witnesses allows the contract to fetch the required data in real-time during its execution.

| Aspect           | Description |
|------------------|-------------|
| **Description**  | A smart contract that needs to dynamically integrate various types of real-time data during its execution; potentially from provable-programs who's execution result is not known upfront. |
| **Requirements** | The smart contract's circuit should be able to pause its execution, perform an asynchronous call to retrieve the necessary data from an external source (like a database or API), and then resume once the data has been resolved. |
| **Expected Outcome** | The contract becomes more flexible, capable of operating with the most current and relevant data, thereby improving its functionality and the developer experience. |
| **Impact Analysis** | Introducing asynchronous circuits in this generic scenario would allow developers to utilize new design patterns and extend the functionality of smart contracts, allowing for a wide range of dynamic and responsive applications across different use-cases. |

### Example Scenario 2: Real-Time Player Attributes in Games

Player attributes such as health, experience points, owned items can change frequently based on in-game actions and events. These attributes would previously needed to have been pre-fetched and compiled for the game's smart contract or provable-programs, leading to potential lags in reflecting real-time changes; the system complexity manifests in the user experience. With asynchronous circuits, the contract (i.e. the circuit) could fetch and current player attributes dynamically during gameplay.

| Aspect           | Description |
|------------------|-------------|
| **Description**  | A smart contract circuit for an RPG game that dynamically fetches updated player attributes like health and experience points during gameplay. |
| **Requirements** | The smart contract circuit must be able to pause its execution, perform an asynchronous fetch for the latest player attributes (possibly from a separate database, API, or provable-program execution), and then resume with the updated data. |
| **Expected Outcome** | Players experience a more seamless and interactive game, where their actions and the game events are reflected in real-time. |
| **Impact Analysis** | Asynchronous circuits would improve the end-user experience as the system complexity does not manifest in the user experience, therefore more appropriately matching the end-user's mental models of the system they are interacting with. |

## Open Issues and Discussion Points

- Discuss implementation details of asynchronous circuits in `o1js`.
- Invite feedback on the use-cases and requirements.