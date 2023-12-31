---
arc: 0080
title: Explicitly Reject Program Address as Record Owner
authors: randomsleep
discussion: https://github.com/AleoHQ/ARCs/discussions/56
topic: Application
status: Draft
created: 2023-12-29
---

## Abstract

The current Aleo system permits the assignment of a `Record` to any address, including program addresses. However, programs cannot spend a `Record`. Consequently, any `Record` assigned to a program becomes irretrievable, leading to unintended asset loss in Aleo, which can include credits, tokens, NFTs, and more. With the introduction of arc-0030, which enables Aleo users to transfer assets to a program, the frequency of such incidents is likely to increase.

This proposal seeks to explicitly prohibit the creation of a `Record` with a program as the owner, significantly reducing the potential for asset loss.


## Specification

The proposed solution involves modifying the `Record` creation process to reject any attempt to assign a `Record` to a program address. This can be achieved by adding a validation step in the `Record` creation process to check the type of the address. If the address belongs to a program, the process should throw an error and abort the `Record` creation.

For example, if a user tries to create a `Record` with a program address like `aleo1qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqquzlzl`, the system should return an error message like "Invalid owner address: A program cannot own a Record."

### Test Cases

The following test cases should be considered:

1. Attempt to create a `Record` with a program address - This should fail with an appropriate error message.
2. Attempt to create a `Record` with a non-program address - This should succeed.
3. Attempt to transfer a `Record` to a program address - This should fail with an appropriate error message.


Considering the following program.

```
import token.leo;

program arc_0080_test.aleo {
    transition show_address() -> (address, address) {
        let signer: address = self.signer;
        let caller: address = token.leo/get_self_address();
        return (signer, caller);
    }

    transition mint_public(public amount: u64) {
        token.leo/self_mint_public(amount);
    }

    transition mint_public_then_transfer(amount: u64) {
        let receiver: address = self.signer;
        token.leo/self_mint_public(amount);
        token.leo/transfer_public(receiver, amount);
    }

    transition mint_private(amount: u64) -> token.leo/token {
        return token.leo/self_mint_private(amount);
    }

    transition mint_private_then_transfer(amount: u64) -> (token.leo/token, token.leo/token) {
        let receiver: address = self.signer;
        let sender: token = token.leo/self_mint_private(amount);
        return token.leo/transfer_private(sender, receiver, amount);
    }
}
```

Run `mint_public` should succeed.
Run `mint_public_then_transfer` should succeed.
Run `mint_private` must fail.
Run `mint_private_then_transfer` must fail.


## Dependencies

This proposal directly affects the Aleo and snarkVM repositories as they contain the logic for `Record` creation and validation.

### Backwards Compatibility

This proposal does not introduce any backwards incompatibility as it adds a new validation step in the `Record` creation process.


## Security & Compliance

This proposal enhances the security of the Aleo platform by preventing unintended asset loss. It does not introduce any new regulatory concerns.
