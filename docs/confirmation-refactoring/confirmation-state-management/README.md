# Confirmation Pages - Frontend State Management

State Management is very important piece to keep frontend confirmation code simplified. Currently state management is fragmented over places and is complicated. Following guidelines will be useful for designing State Magagement:

1. Use state obtained from backend (redux store `state.metamask`) as single source of truth
2. For state derived from the backend state hooks can be written, these will internally use backend state
3. For temporary UI state shared across multiple components React Context can be used, minimise the scope of the context to just the components that need them (this is useful to avoid un-necessary re-rendering cycles in the app)
4. Confirmation React components fall into 2 categories:
   - Smart components: state access should go here
   - Dumb components: they are used for layout mainly and should ideally have required state data passed to them via props
5. Redux state is a good candidate for implementing state machine on frontend, if require anywhere in confirmation pages. Though currently transient state is mostly confined to single component state machine may not be needed.

Refactorings:

- There are confirmations related ducks [here](https://github.com/MetaMask/metamask-extension/tree/develop/ui/ducks):
  - [confirm-transaction](https://github.com/MetaMask/metamask-extension/tree/develop/ui/ducks/confirm-transaction): this is redundant and we should be able to get rid of it.
  - [gas](https://github.com/MetaMask/metamask-extension/tree/develop/ui/ducks/gas): this is not used anywhere and can be removed.
  - [send](https://github.com/MetaMask/metamask-extension/tree/develop/ui/ducks/send): this duck is important state machine for send flow and we should continue to maintain.
- [gasFeeContext](https://github.com/MetaMask/metamask-extension/blob/develop/ui/contexts/gasFee.js) is huge context written on top of [gasFeeInput](https://github.com/MetaMask/metamask-extension/tree/develop/ui/hooks/gasFeeInput) hook. The context / hook provides about 20 different values used in different places in confirmation pages. We need to break this down:

  - Context is required only to provide temporary UI state for confirmation pages which includes:

    - `transaction` - active transaction on confirmation pages
    - `editGasMode` - cancel, speedup, swap or default, this is also temporary UI state

    The context can be included in `/pages/confirm-transaction-base` and around `TokenAllowance` in `/pages/confirm-approve`.

  - Hooks can be created for values derived from values derived from above context and metamask state. This include:
    - `maxFeePerGas`
    - `maxPriorityFeePerGas`
    - `supportEIP1559`
    - `balanceError`
    - `minimumCostInHexWei`
    - `maximumCostInHexWei`
    - `hasSimulationError`
    - `estimateUsed`
  - Values which can be obtained from metamask state using selectors should be removed from this context. This includes:
    - `gasFeeEstimates`
    - `isNetworkBusy`
  - `minimumGasLimitDec` is a constant value 21000 should be removed from the context, this can be moved to constants file.
  - Create separate hook for transaction functions [here](https://github.com/MetaMask/metamask-extension/blob/develop/ui/hooks/gasFeeInput/useTransactionFunctions.js), this hook can consume GasFeeContext.
  - Setters and manual update functions are only used by legacy gas component [edit-gas-fee-popover](https://github.com/MetaMask/metamask-extension/tree/develop/ui/components/app/edit-gas-popover). This component uses useGasFeeInputs hook. We need to create a smaller hook just for this component using the above context and hooks.

* [confirm-transaction-base.container.js](https://github.com/MetaMask/metamask-extension/blob/develop/ui/pages/confirm-transaction-base/confirm-transaction-base.container.js) and [confirm-transaction-base.component.js](https://github.com/MetaMask/metamask-extension/blob/develop/ui/pages/confirm-transaction-base/confirm-transaction-base.component.js) has a lot of code to derive values from state and selected transactions. This can be simplified by using hooks that will he created.
* We will have a lot of hooks for transaction related fields, these can be grouped into same file / folder.

As we work on the components we will be able to identify more areas of improvement.

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
