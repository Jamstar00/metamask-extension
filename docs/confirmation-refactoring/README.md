# Confirmation Pages Refactoring

The following pages document the ongoing refactoring efforts of confirmation pages. They describe the current (2023) code and proposed changes.

1. [Signature Request Pages](./signature-request/README.md)

2. [Confirmation Pages Routing](./confirmation-pages-routing/README.md)

3. [Confirmation Page Structure](./confirmation-page=structure/README.md)

4. [Confirmation State Management](./confirmation-state-management/README.md)

5. [Confirmation Backend Architecture](./confirmation-backend-architecture/README.md)

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
