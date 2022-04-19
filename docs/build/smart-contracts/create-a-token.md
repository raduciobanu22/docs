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

Managed capability used to allow the transfer of `amount` from `sender` to `receiver` in any number of payments. Managed capabilities emit events automatically, in this case a `TRANSFER` event with the specified parameters is emitted when the capability is acquired.

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

Here is a token implementation using `coin` contract as guideline:

```clojure
(define-keyset 'token-admin)

(module token GOVERNANCE

  @doc "Fungible token contract example"

  @model
    [ (defproperty conserves-mass
        (= (column-delta token-table 'balance) 0.0))

      (defproperty valid-account (account:string)
        (and
          (>= (length account) 3)
          (<= (length account) 256)))
    ]

  (implements fungible-v2)
  (implements fungible-xchain-v1)

  ; --------------------------------------------------------------------------
  ; Schemas and Tables

  (defschema token-schema
    @doc "The token schema"
    @model [ (invariant (>= balance 0.0)) ]

    balance:decimal
    guard:guard)

  (deftable token-table:{token-schema})

  ; --------------------------------------------------------------------------
  ; Capabilities

  (defcap GOVERNANCE ()
    @doc " Only admin can upgrade the module. "
    (enforce-keyset 'token-admin))

  (defcap ACCOUNT-OWNER (account:string)
    @doc "Validate the ownership of the account"
    (enforce-guard (at 'guard (details account)))
  )

  (defcap DEBIT (sender:string)
    "Capability for managing debiting operations"
    (enforce-guard (at 'guard (read token-table sender)))
    (enforce (!= sender "") "valid sender"))

  (defcap CREDIT (receiver:string)
    "Capability for managing crediting operations"
    (enforce (!= receiver "") "valid receiver"))

  (defcap ROTATE (account:string)
    @doc "Autonomously managed capability for guard rotation"
    @managed
    true)

  (defcap TRANSFER:bool
    ( sender:string
      receiver:string
      amount:decimal
    )
    @managed amount TRANSFER-mgr
    (enforce (!= sender receiver) "same sender and receiver")
    (enforce-unit amount)
    (enforce (> amount 0.0) "Positive amount")
    (compose-capability (DEBIT sender))
    (compose-capability (CREDIT receiver))
  )

  (defun TRANSFER-mgr:decimal
    ( managed:decimal
      requested:decimal
    )

    (let ((newbal (- managed requested)))
      (enforce (>= newbal 0.0)
        (format "TRANSFER exceeded for balance {}" [managed]))
      newbal)
  )

  (defcap TRANSFER_XCHAIN:bool
    ( sender:string
      receiver:string
      amount:decimal
      target-chain:string
    )

    @managed amount TRANSFER_XCHAIN-mgr
    (enforce-unit amount)
    (enforce (> amount 0.0) "Cross-chain transfers require a positive amount")
    (compose-capability (DEBIT sender))
  )

  (defun TRANSFER_XCHAIN-mgr:decimal
    ( managed:decimal
      requested:decimal
    )

    (enforce (>= managed requested)
      (format "TRANSFER_XCHAIN exceeded for balance {}" [managed]))
    0.0
  )

  (defcap TRANSFER_XCHAIN_RECD:bool
    ( sender:string
      receiver:string
      amount:decimal
      source-chain:string
    )
    @event true
  )

  ; --------------------------------------------------------------------------
  ; Constants

  (defconst ACCOUNT_CHARSET CHARSET_LATIN1
    "The default token contract character set")

  (defconst MINIMUM_PRECISION 12
    "Minimum allowed precision for token transactions")

  (defconst MINIMUM_ACCOUNT_LENGTH 3
    "Minimum account length admissible for token accounts")

  (defconst MAXIMUM_ACCOUNT_LENGTH 256
    "Maximum account name length admissible for token accounts")

  (defconst TREASURY_ACCOUNT 'token-treasury)

  (defconst INITIAL_SUPPLY 1000.0
    "TOTAL_SUPPLY = INITIAL_SUPPLY * NUMBER_OF_CHAINS")

  ; --------------------------------------------------------------------------
  ; Utilities

  (defun enforce-unit:bool (amount:decimal)
    @doc "Enforce minimum precision allowed for token transactions"

    (enforce
      (= (floor amount MINIMUM_PRECISION)
         amount)
      (format "Amount violates minimum precision: {}" [amount]))
    )

  (defun validate-account (account:string)
    @doc "Enforce that an account name conforms to the token contract \
         \minimum and maximum length requirements, as well as the    \
         \latin-1 character set."

    (enforce
      (is-charset ACCOUNT_CHARSET account)
      (format
        "Account does not conform to the coin contract charset: {}"
        [account]))

    (let ((account-length (length account)))

      (enforce
        (>= account-length MINIMUM_ACCOUNT_LENGTH)
        (format
          "Account name does not conform to the min length requirement: {}"
          [account]))

      (enforce
        (<= account-length MAXIMUM_ACCOUNT_LENGTH)
        (format
          "Account name does not conform to the max length requirement: {}"
          [account]))
      )
  )

  ; --------------------------------------------------------------------------
  ; Token Contract

  (defun create-account:string (account:string guard:guard)
    @model [ (property (valid-account account)) ]

    (validate-account account)
    (enforce-reserved account guard)

    (insert token-table account
      { "balance" : 0.0
      , "guard"   : guard
      })
    )

  (defun get-balance:decimal (account:string)
    (with-read token-table account
      { "balance" := balance }
      balance
      )
    )

  (defun details:object{fungible-v2.account-details}
    ( account:string )
    (with-read token-table account
      { "balance" := bal
      , "guard" := g }
      { "account" : account
      , "balance" : bal
      , "guard": g })
    )

  (defun rotate:string (account:string new-guard:guard)
    (with-capability (ROTATE account)
      (with-read token-table account
        { "guard" := old-guard }

        (enforce-guard old-guard)

        (update token-table account
          { "guard" : new-guard }
          )))
    )


  (defun precision:integer
    ()
    MINIMUM_PRECISION)

  (defun transfer:string (sender:string receiver:string amount:decimal)
    @model [ (property conserves-mass)
             (property (> amount 0.0))
             (property (valid-account sender))
             (property (valid-account receiver))
             (property (!= sender receiver)) ]

    (enforce (!= sender receiver)
      "sender cannot be the receiver of a transfer")

    (validate-account sender)
    (validate-account receiver)

    (enforce (> amount 0.0)
      "transfer amount must be positive")

    (enforce-unit amount)

    (with-capability (TRANSFER sender receiver amount)
      (debit sender amount)
      (with-read token-table receiver
        { "guard" := g }

        (credit receiver g amount))
      )
    )

  (defun transfer-create:string
    ( sender:string
      receiver:string
      receiver-guard:guard
      amount:decimal )

    @model [ (property conserves-mass) ]

    (enforce (!= sender receiver)
      "sender cannot be the receiver of a transfer")

    (validate-account sender)
    (validate-account receiver)

    (enforce (> amount 0.0)
      "transfer amount must be positive")

    (enforce-unit amount)

    (with-capability (TRANSFER sender receiver amount)
      (debit sender amount)
      (credit receiver receiver-guard amount))
    )

  (defun debit:string (account:string amount:decimal)
    @doc "Debit AMOUNT from ACCOUNT balance"

    @model [ (property (> amount 0.0))
             (property (valid-account account))
           ]

    (validate-account account)

    (enforce (> amount 0.0)
      "debit amount must be positive")

    (enforce-unit amount)

    (require-capability (DEBIT account))
    (with-read token-table account
      { "balance" := balance }

      (enforce (<= amount balance) "Insufficient funds")

      (update token-table account
        { "balance" : (- balance amount) }
        ))
    )


  (defun credit:string (account:string guard:guard amount:decimal)
    @doc "Credit AMOUNT to ACCOUNT balance"

    @model [ (property (> amount 0.0))
             (property (valid-account account))
           ]

    (validate-account account)

    (enforce (> amount 0.0) "credit amount must be positive")
    (enforce-unit amount)

    (require-capability (CREDIT account))
    (with-default-read token-table account
      { "balance" : -1.0, "guard" : guard }
      { "balance" := balance, "guard" := retg }
      ; we don't want to overwrite an existing guard with the user-supplied one
      (enforce (= retg guard)
        "account guards do not match")

      (let ((is-new
             (if (= balance -1.0)
                 (enforce-reserved account guard)
               false)))

        (write token-table account
          { "balance" : (if is-new amount (+ balance amount))
          , "guard"   : retg
          }))
      ))

  (defun check-reserved:string (account:string)
    " Checks ACCOUNT for reserved name and returns type if \
    \ found or empty string. Reserved names start with a \
    \ single char and colon, e.g. 'c:foo', which would return 'c' as type."
    (let ((pfx (take 2 account)))
      (if (= ":" (take -1 pfx)) (take 1 pfx) "")))

  (defun enforce-reserved:bool (account:string guard:guard)
    @doc "Enforce reserved account name protocols."
    (if (validate-principal guard account)
      true
      (let ((r (check-reserved account)))
        (if (= r "")
          true
          (if (= r "k")
            (enforce false "Single-key account protocol violation")
            (enforce false
              (format "Reserved protocol guard violation: {}" [r]))
            )))))


  (defschema crosschain-schema
    @doc "Schema for yielded value in cross-chain transfers"
    receiver:string
    receiver-guard:guard
    amount:decimal
    source-chain:string)

  (defpact transfer-crosschain:string
    ( sender:string
      receiver:string
      receiver-guard:guard
      target-chain:string
      amount:decimal )

    @model [ (property (> amount 0.0))
             (property (valid-account sender))
             (property (valid-account receiver))
           ]

    (step
      (with-capability
        (TRANSFER_XCHAIN sender receiver amount target-chain)

        (validate-account sender)
        (validate-account receiver)

        (enforce (!= "" target-chain) "empty target-chain")
        (enforce (!= (at 'chain-id (chain-data)) target-chain)
          "cannot run cross-chain transfers to the same chain")

        (enforce (> amount 0.0)
          "transfer quantity must be positive")

        (enforce-unit amount)

        ;; step 1 - debit delete-account on current chain
        (debit sender amount)
        (emit-event (TRANSFER sender "" amount))

        (let
          ((crosschain-details:object{crosschain-schema}
            { "receiver" : receiver
            , "receiver-guard" : receiver-guard
            , "amount" : amount
            , "source-chain" : (at 'chain-id (chain-data))
            }))
          (yield crosschain-details target-chain)
          )))

    (step
      (resume
        { "receiver" := receiver
        , "receiver-guard" := receiver-guard
        , "amount" := amount
        }
        (emit-event (TRANSFER "" receiver amount))
        ;; step 2 - credit create account on target chain
        (with-capability (CREDIT receiver)
          (credit receiver receiver-guard amount))
        ))
  )

  (defun init()
    (with-capability (GOVERNANCE)
      (insert token-table TREASURY_ACCOUNT
        {
          'balance: INITIAL_SUPPLY,
          'guard: (read-keyset 'token-admin)
        }
      )
    )
  )
)

(if (read-msg "upgrade")
  ["upgrade"]
  [
    (create-table token-table)
  ]
)

```

We're using the `init` function to initialize the contract and award the tokens to the initial owner.

```clojure
(defun init()
    (with-capability (GOVERNANCE)
        (insert token-table TREASURY_ACCOUNT
            {
                'balance: INITIAL_SUPPLY,
                'guard: (read-keyset 'token-admin)
            }
        )
    )
)
```

:::info
Here we assume that you already have a dev env setup, if not, please follow the [quickstart guide](#).
:::

Copy the source code in a new file `token.pact` and save it.

The module is generically called `token`, which is also the name of the token itself. You can rename it to anything you want, as long the name doesn't already exist in the same namespace.

To deploy the new token to `testnet` we will use the snippet below:

```js
const Pact = require('pact-lang-api');
const fs = require('fs');

const NETWORK_ID = 'testnet04';
const CHAIN_ID = '1';
const API_HOST = `https://api.testnet.chainweb.com/chainweb/0.0/${NETWORK_ID}/chain/${CHAIN_ID}/pact`;
const CONTRACT_PATH = 'token.pact';

const KEY_PAIR = {
  'publicKey': 'my-public-key',
  'secretKey': 'my-private-key'
}

const pactCode = fs.readFileSync(CONTRACT_PATH, 'utf8');
const creationTime = () => Math.round((new Date).getTime() / 1000) - 15;

deployContract(pactCode);

async function deployContract(pactCode) {
  const cmd = {
    networkId: NETWORK_ID,
    keyPairs: KEY_PAIR,
    pactCode: pactCode,
    envData: {
      'token-admin': [KEY_PAIR['publicKey']],
      'upgrade': false
    },
    meta: {
      creationTime: creationTime(),
      ttl: 28000,
      gasLimit: 65000,
      chainId: CHAIN_ID,
      gasPrice: 0.000001,
      sender: KEY_PAIR.publicKey
    }
  };
  const response = await Pact.fetch.send(cmd, API_HOST);
  console.log(response);
  const txResult = await Pact.fetch.listen({ listen: response.requestKeys[0] }, API_HOST);
  console.log(txResult);
};
```

:::note

Make sure `CONTRACT_PATH` is correct and do not forget to change `KEY_PAIR` values.
You can also change `token-admin` if you would like a different keyset to be the one that has governance rights. Check `GOVERNANCE` capability if you are unsure what this means.
:::

Save the snippet to `deploy.js` and run it in a terminal:

```bash
node deploy.js
```

After the transaction was mined, the contract is ready to be used. You can try it out by sending some tokens to a new account using `transfer-create`.


