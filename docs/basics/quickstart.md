---
title: Kadena Quick Start Guide
description: Setting up a Kadena Development Environment
---

import PageRef from '@components/PageRef'
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Setting up a Kadena Development Environment

Quick Start Guide.

---

Kadena's Chainweb is a horizontally scalable proof-of-work blockchain that together with the Pact smart contract programming language offers a complete platform for builders to develop and launch state-of-the-art decentralized applications.

Chainweb is a public network and anyone can invoke write and read operations to and from the blockchain. Developers can write software called smart contracts that are deployed to Chainweb's network of 20 chains.

In order to write an application for Kadena, you need to have some knowledge of smart contract architecture and understanding of basic cryptography concepts such as digital signatures, public-private key pairs and hash functions.

## Kadena Development Key Concepts

You only need a few minutes to setup your development environment for Kadena but it's important to have a basic understanding of some general concepts before jumping to writing code.

1. **Pact** - Turing-incomplete smart contract language that has been purpose-built with blockchains first in mind. Pact focuses on facilitating transactional logic with the optimal mix of functionality in authorization, data management, and workflow.

2. **Pact-Lang-Api.js** - A Javascript library for web browsers and Node.js that enables developers to easily read and write to the Kadena blockchain.

3. **Pact Local Server** - A full REST API HTTP server and SQLite database implementation that simulates a single-node blockchain environment, with the same API supported by the Kadena Chainweb blockchain.

4. **Atom IDE** - Atom editor together with `language-pact` package offers a full-fledged IDE experience to seamlessly write Pact code.


### Installing Pact

Pact is an interpreted programming language which makes it human-readable allowing anyone to check any smart contract deployed on Kadena blockchain.

Installing Pact can be done using Homebrew on Mac or Nix for Linux distributions. If you're on Mac simply run `brew install pact` in your terminal. To build with Nix follow the [instructions from the official Pact repository](https://github.com/kadena-io/pact#building-with-nix).

Once you have installed it you can use it on the command line to execute Pact code:

```bash
pact> (+ 1 2)
3
```

### Installing pact-lang-api.js

`pact-lang-api.js` is a library to interact with Kadena blockchain. To use it in a web page, you can import it directly using a CDN and `Pact` will be available as a global variable:

 ```js
 <script src="https://cdn.jsdelivr.net/npm/pact-lang-api@4.1.2/pact-lang-api-global.min.js"></script>
 ```

To install it for a Node.js script/backend application or for a built front-end project, you can use NPM or Yarn.

```bash

npm install pact-lang-api

# or

yarn add pact-lang-api
```

If you used the CDN approach, there's nothing else to do, you can directly use the lib in your code.

To import `pact-lang-api` into your Node.js project or Browserify front-end app use the following line:

```js
const Pact = require('pact-lang-api);
```

### Running Pact Server

Pact Local Server is a great tool for Kadena development and testing. You can use it to deploy smart contracts and interact with them as if they were on a real network.
Pact Server is part of the `pact` command line tool that you [already installed](#installing-pact). To run the server you just need a configuration file. Copy the example below and save it as `config.yaml`.

```bash
# Config file for pact http server. Launch with `pact -s config.yaml`

# HTTP server port
port: 8080

# directory for HTTP logs
logDir: log

# persistence directory
persistDir: log

# SQLite pragmas for pact back-end
pragmas: []

# verbose: provide log output
verbose: True
```

Now create the logs directory and start the server:

```bash
mkdir log
pact -s config.yaml
```

### Installing Atom and language-pact

Pact is supported by a variety of editors like Atom, Emacs or Vim. For a full-fledged IDE experience, install the [Atom](https://atom.io) editor along with `language-pact` package using the [atom package manager](https://flight-manual.atom.io/using-atom/sections/atom-packages/).

You can also check the Pact official repository for [instructions on using other editors](https://github.com/kadena-io/pact#supported-editors).

## Building Your First DApp

Our development environment is ready and we have an understanding of basic Kadena concepts, it's now time to write a first Dapp.

After you complete this guide, you can follow a mode advanced tutorial to build a voting Dapp.

### Configure Your Project And Local Development Environment

Create a directory on your local machine for your Dapp project files.

```bash
mkdir first-kadena-dapp
cd first-kadena-dapp
mkdir pact
```

### Writing a Smart Contract with Pact

We are going to write our smart contract in a file called `hello.pact`.

```bash
touch pact/hello.pact
```

Open the Pact file with Atom or another editor of your choice for writing code. Here's the code for our contract:

```clojure
(module Hello GOVERNANCE

    (defcap GOVERNANCE ()
        (enforce false "Module is not upgradable"))

    (defun hello:string ()
        (format "Hello from Kadena!"))
)
```

The contract has a `hello` function that only returns the "Hello from Kadena!" message. Save the file once you are done.

### Testing With REPL

### Deploying to Pact Local Server

### Writing the Front-End

### Running Your DApp
