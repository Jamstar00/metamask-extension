# Confirmation Background Architecture and Code Cleanup

## Current Implementation:

Current confirmation implementation in the background consists of following pieces:

1. `TransactionController` and utility, helper classes used by it:
   `TransactionController` is very important piece in transaction processing. It is described [here](https://github.com/MetaMask/metamask-extension/tree/develop/app/scripts/controllers/transactions#transaction-controller). It consists of 4 important parts:
   - `txStateManager`: responsible for the state of a transaction and storing the transaction
   - `pendingTxTracker`: watching blocks for transactions to be include and emitting confirmed events
   - `txGasUtil`: gas calculations and safety buffering
   - `nonceTracker`: calculating nonces
2. `MessageManagers`:
   There are 3 different message managers responsible for processing signature requests. These are detailed [here](https://github.com/MetaMask/metamask-extension/tree/develop/docs/refactoring/signature-request#proposed-refactoring).
3. `MetamaskController `:
   `MetamaskController ` is responsible for gluing together the different pieces in transaction processing. It is responsible to inject dependencies in `TransactionController`, `MessageManagers`, handling different events, responses to DAPP requests, etc.

## Areas of Code Cleanup:

1. Migrating to `@metamask/transaction-controller`. `TransactionController` in extension repo should eventually get replaced by core repo [TransactionController](https://github.com/MetaMask/core/tree/main/packages/transaction-controller). This controller is maintained by core team and also used in Metamask Mobile App.
2. Migrating to `@metamask/message-manager`. Message Managers in extension repo should be deprecated in favor of core repo [MessageManagers](https://github.com/MetaMask/core/tree/main/packages/message-manager).
3. Cleanup Code in `MetamaskController`. [Metamaskcontroller](https://github.com/MetaMask/metamask-extension/blob/develop/app/scripts/metamask-controller.js) is where `TransactionController` and different `MessageManagers` are initialized. It is responsible for injecting required dependencies. Also, it is responsible for handling incoming DAPP requests and invoking appropriate methods in these background classes. Over the period of time lot of code that should have been part of `TransactionController` and `MessageManagers` has ended up in `MetamaskController`. We need to cleanup this code and move to the appropriate classes.
   - Code [here](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L3097) to check if `eth_sign` is enabled in preferences and perform other validation on the incoming request should be part of [MessageManager](https://github.com/MetaMask/metamask-extension/blob/develop/app/scripts/lib/message-manager.js)
   - Method to sign messages [signMessage](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L3158), [signPersonalMessage](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L3217), [signTypedMessage](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L3470) can be simplified by injecting `KeyringController` into `MessageManagers`.
   - There are about 11 different methods to `add`, `approve`, `reject` different types of signature requests. These can probably be moved to a helper class, thus reducing lines of code from `MetamaskController `.
   - This [code](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L959) can better be placed in `TransactionController`.
   - A lot of other methods in `MetamaskController` which are related to `TransactionController` and the state of `TransactionController` can be moved into `TransactionController` itself like [method1](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L1179), [method2](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L3570), [method3](https://github.com/MetaMask/metamask-extension/blob/bc19856d5d9ad1831e1722c84fe6161bed7a0a5a/app/scripts/metamask-controller.js#L4349), etc.

### Using ApprovalController for Confirmations

[ApprovalController](https://github.com/MetaMask/core/tree/main/packages/approval-controller) is written as a helper to `PermissionController`. Its role is to manage requests that require user approval. It can also be used in confirmation code to launch UI. Thus the use of `showUserConfirmation` function in `MetamaskController ` can be removed.
But `ApprovalController` will need some changes to be able to use it for confirmations, for example, it does not support multiple parallel requests from the same origin.

## Result Pages

Result pages are an essential part of the confirmation process. They provide users with feedback on the outcome of their actions, such as approving or rejecting a transaction. Result pages help ensure that users are informed about the status of their requests and any necessary next steps.

### Purpose and Structure

The purpose of result pages is to display the outcome of a user's action in a clear and concise manner. They typically include the following elements:

1. **Header**: A brief message indicating the result of the action (e.g., "Transaction Approved" or "Transaction Rejected").
2. **Details**: Information about the action that was taken, such as transaction details, gas fees, and any relevant metadata.
3. **Next Steps**: Guidance on what the user should do next, if applicable (e.g., "You can now view your transaction in the activity tab").
4. **Visual Feedback**: Icons or images that help convey the result of the action (e.g., a checkmark for approval or an X for rejection).

### Usage in the Confirmation Process

Result pages are used in the confirmation process to provide users with immediate feedback on the outcome of their actions. When a user approves or rejects a transaction, they are redirected to a result page that displays the relevant information. This helps ensure that users are aware of the status of their requests and can take any necessary follow-up actions.

In the context of the MetaMask extension, result pages are typically displayed after a user interacts with a confirmation page. For example, after approving a token transfer, the user would be redirected to a result page that confirms the approval and provides details about the transaction.
