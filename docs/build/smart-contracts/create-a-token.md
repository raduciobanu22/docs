---
title: Create a token
description: Create a token using fungible-v2 standard
---

import PageRef from '@components/PageRef'
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

# Create a Token Using Pact

---

## Fungible-v2

Fungible-v2 defines a common list of traits that all fungible tokens in Kadena ecosystem should adhere to. This ensures interoperability between tokens and allows developers to easily integrate any token in their projects as long as it is following the rules.

If you are familiar with the ERC-20 token standard, **fungible-v2** is the equivalent for Pact and Kadena and was introduced with [KIP-0005](https://github.com/kadena-io/KIPs/blob/master/kip-0005.md).

Below is the `fungible-v2` interface that tokens must implement:

  ```clojure
  (interface fungible-v2

  " Standard for fungible coins and tokens as specified in KIP-0002. "

   ; ----------------------------------------------------------------------
   ; Schema

   (defschema account-details
    @doc "Schema for results of 'account' operation."

    account:string
    balance:decimal
    guard:guard)

  ; ----------------------------------------------------------------------
   ; Caps

   (defcap TRANSFER:bool
     ( sender:string
       receiver:string
       amount:decimal
     )
     @doc " Managed capability sealing AMOUNT for transfer from SENDER to \
          \ RECEIVER. Permits any number of transfers up to AMOUNT."
     @managed amount TRANSFER-mgr
     )

   (defun TRANSFER-mgr:decimal
     ( managed:decimal
       requested:decimal
     )
     @doc " Manages TRANSFER AMOUNT linearly, \
          \ such that a request for 1.0 amount on a 3.0 \
          \ managed quantity emits updated amount 2.0."
     )

   ; ----------------------------------------------------------------------
   ; Functionality


  (defun transfer:string
    ( sender:string
      receiver:string
      amount:decimal
    )
    @doc " Transfer AMOUNT between accounts SENDER and RECEIVER. \
         \ Fails if either SENDER or RECEIVER does not exist."
    @model [ (property (> amount 0.0))
             (property (!= sender ""))
             (property (!= receiver ""))
             (property (!= sender receiver))
           ]
    )

   (defun transfer-create:string
     ( sender:string
       receiver:string
       receiver-guard:guard
       amount:decimal
     )
     @doc " Transfer AMOUNT between accounts SENDER and RECEIVER. \
          \ Fails if SENDER does not exist. If RECEIVER exists, guard \
          \ must match existing value. If RECEIVER does not exist, \
          \ RECEIVER account is created using RECEIVER-GUARD. \
          \ Subject to management by TRANSFER capability."
     @model [ (property (> amount 0.0))
              (property (!= sender ""))
              (property (!= receiver ""))
              (property (!= sender receiver))
            ]
     )

   (defpact transfer-crosschain:string
     ( sender:string
       receiver:string
       receiver-guard:guard
       target-chain:string
       amount:decimal
     )
     @doc " 2-step pact to transfer AMOUNT from SENDER on current chain \
          \ to RECEIVER on TARGET-CHAIN via SPV proof. \
          \ TARGET-CHAIN must be different than current chain id. \
          \ First step debits AMOUNT coins in SENDER account and yields \
          \ RECEIVER, RECEIVER_GUARD and AMOUNT to TARGET-CHAIN. \
          \ Second step continuation is sent into TARGET-CHAIN with proof \
          \ obtained from the spv 'output' endpoint of Chainweb. \
          \ Proof is validated and RECEIVER is credited with AMOUNT \
          \ creating account with RECEIVER_GUARD as necessary."
     @model [ (property (> amount 0.0))
              (property (!= sender ""))
              (property (!= receiver ""))
              (property (!= sender receiver))
              (property (!= target-chain ""))
            ]
     )

   (defun get-balance:decimal
     ( account:string )
     " Get balance for ACCOUNT. Fails if account does not exist."
     )

   (defun details:object{account-details}
     ( account: string )
     " Get an object with details of ACCOUNT. \
     \ Fails if account does not exist."
     )

   (defun precision:integer
     ()
     "Return the maximum allowed decimal precision."
     )

   (defun enforce-unit:bool
     ( amount:decimal )
     " Enforce minimum precision allowed for transactions."
     )

   (defun create-account:string
     ( account:string
       guard:guard
     )
     " Create ACCOUNT with 0.0 balance, with GUARD controlling access."
     )

   (defun rotate:string
     ( account:string
       new-guard:guard
     )
     " Rotate guard for ACCOUNT. Transaction is validated against \
     \ existing guard before installing new guard. "
     )

)
```

:::info
If you are not sure what is an interface is you can read more about this concept and how it's implemented in Pact [here](https://pact-language.readthedocs.io/en/latest/pact-reference.html?highlight=interface#interface-declaration).
:::

Next we're going to explain each function and the capabilities defined in **fungible-v2**.

### Schema

```clojure
(defschema account-details account:string balance:decimal guard:guard)
```
Schema of the object returned by `details` function.

### Capabilities

```clojure
(defcap TRANSFER:bool ( sender:string receiver:string amount:decimal)
    @managed amount TRANSFER-mgr
)
```

Managed capability used to allow the transfer of `amount` from `sender` to `receiver` in any number of payments.

---

```clojure
(defun TRANSFER-mgr:decimal ( managed:decimal requested:decimal ))
```

The manager function which keeps track of the remaining amount that can be transferred. Once `managed` is 0, the TRANSFER capability cannot be brought into scope anymore.

### Functions

```clojure
(defun transfer:string ( sender:string receiver:string amount:decimal ))
```

Simple transfer of `amount` from `sender` to `receiver`. Does not specify guard of receiving account. It is not safe to use this method if `receiver` is a vanity account and you are not sure the account already exists.

---

```clojure
(defun transfer-create:string ( sender:string receiver:string receiver-guard:guard amount:decimal ))
```
Transfer `amount` from `sender` to `receiver` while also specifying the `receiver-guard`. If account does not exist, it is created using `receiver-guard`, otherwise if the account exists and guard does not match, transfer will fail.

---

```clojure
(defpact transfer-crosschain:string
    ( sender:string
      receiver:string
      receiver-guard:guard
      target-chain:string
      amount:decimal
    )
)
```

Defines a 2-step pact that transfers `amount` from `sender` to `receiver` from current chain to `target-chain`. First step debits the amount from `sender` and yields `receiver`, `receiver-guard` and amount on `target-chain`. Second step validates the proof sent with SPV and credits the amount to receiver. If account does not exist on `target-chain` it is created using `receiver-guard`. If the account exists and the guard does not match, transaction will fail.

---

```clojure
(defun get-balance:decimal ( account:string ))
```

Returns the balance of an account, fails if account doesn't exist.

---

```clojure
(defun details:object{account-details} ( account: string ))
```

Returns an `account-details` object, fails if account doesn't exist.

---

```clojure
(defun precision:integer ())
```

Returns the token's maximum decimal precision.

---

```clojure
(defun enforce-unit:bool ( amount:decimal ))
```

Utility function used to enforce the minimum precision allowed for token transactions.

---

```clojure
(defun create-account:string ( account:string guard:guard))
```

Creates a new `account` using `guard` to control access.

---

```clojure
(defun rotate:string ( account:string new-guard:guard))
```

Validates the existing guard and installs the `new-guard` of `account`.

## Deploy a new token

