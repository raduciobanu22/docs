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

Next run the following command to create a `package.json` file that will keep track of our project's dependencies:

```bash
npm init -y

# or

yarn init -y
```

Earlier we've already mentioned how to [install pact-lang-api](#installing-pact-lang-apijs), use the same commands to install it as a dependency to our new project. Additionally we'll also add the `http-server` package:

```bash
npm install http-server
```

Now we've got everything we need to build our first Dapp.

### Writing a Smart Contract with Pact

We are going to write our smart contract in a file called `kapy.pact`.

```bash
touch pact/kapy.pact
```

Open the Pact file with Atom or another editor of your choice for writing code. Here's the code for our contract:

```clojure
(module kapy GOVERNANCE

    (defcap GOVERNANCE ()
        (enforce false "Module is not upgradable"))

    (defun hello:string ()
        (format "Hello from Kadena ⛓🕸!" []))
)
```

The contract has a `hello` function that only returns the "Hello from Kadena!" message. Save the file once you are done.

### Testing With REPL

Next we'll use the Pact built-in REPL tool to test the contract that we just wrote. Create a new file `kapy.repl` and add the following code:

```clojure

(begin-tx)

(load "kapy.pact")

(use kapy)

(expect "Received hello" (kapy.hello) "Hello from Kadena ⛓🕸!")

(commit-tx)

```
Save the file and run it using the pact command line tool:

```bash
pact>(load "kapy.repl")
```

If successful you will see the output in your terminal:

```bash
"Loading pact/kapy.repl..."
"Begin Tx 0"
"Loading kapy.pact..."
"Loaded module kapy, hash KUmsdwEMt_ZWphXBQkrlo3v0BZcwWl2HdwQeil1zKbA"
"Using kapy"
"Expect: success: Received a hello message"
"Commit Tx 0"
```

### Deploying to Pact Local Server

We know our contract is working correctly so it's time to deploy it. In a separate terminal create a new file called `deploy.js`:

```bash
touch deploy.js
```

Add the following code to the file. Running this script will deploy our smart contract to the Pact local server , which should be [running](#running-pact-server) in a separate terminal:

```js

const Pact = require('pact-lang-api');
const fs = require('fs');

const API_HOST = 'http://localhost:8080';
const KEY_PAIR = Pact.crypto.genKeyPair();
const pactCode = fs.readFileSync('./pact/kapy.pact', 'utf8');

deployContract(pactCode);

async function deployContract(pactCode) {
  const cmd = {
    keyPairs: KEY_PAIR,
    pactCode: pactCode
  };
  const response = await Pact.fetch.send(cmd, API_HOST);
  const txResult = await Pact.fetch.listen({ listen: response.requestKeys[0] }, API_HOST);
  console.log(txResult);
};

```

Save the file and run it using the following command:

```bash
node deploy.js
```

You should see an output similar to the one below which confirms the `kapy` module was successfully deployed:

```bash
{
  gas: 0,
  result: {
    status: 'success',
    data: 'Loaded module kapy, hash KUmsdwEMt_ZWphXBQkrlo3v0BZcwWl2HdwQeil1zKbA'
  },
  reqKey: 'V0rVJdygJvnij0jPd0bekxFDtlATqC3bWhOowtf1NFI',
  logs: '56n7Msx_zFcEUEExWy1jt1g28zPIF8MFefP2WM_Wv7A',
  metaData: null,
  continuation: null,
  txId: 0
}
```

### Writing the Front-End

On the front-end part, we will call our contract method `hello` from the browser and display the result. Our Dapp does not write anything to the blockchain so we don't need a wallet to sign transactions.

Create an HTML file in your project:

```bash
touch index.html
```

We'll add the UI and use `pact-lang-api` to call the smart contract:

```html

<!DOCTYPE html>
<html>
<head>
  <title></title>
</head>
<body>

  <h1>First Kadena Dapp</h1>
  <label>Contract call:</label>
  <span>kapy.hello</span>
  <br />
  <label>Result:</label>
  <span id="message"></span>

</body>
<script src="https://cdn.jsdelivr.net/npm/pact-lang-api@4.1.2/pact-lang-api-global.min.js"></script>

<script>

const API_HOST = 'http://localhost:8080';
const KEY_PAIR = Pact.crypto.genKeyPair();
const htmlLabel = document.getElementById('message');

const cmd = {
  keyPairs: KEY_PAIR,
  pactCode: '(kapy.hello)'
};

Pact.fetch.local(cmd, API_HOST).then((response) => {
  htmlLabel.innerHTML = response.result.data;
})

</script>
</html>


```

### Running Your DApp

Our Dapp is now ready! We just need to start the HTTP server that will serve our HTML file:

```bash
npx http-server

Starting up http-server, serving ./

http-server version: 14.1.0

....

Available on:
  http://127.0.0.1:8081
  http://172.16.172.16:8081
  http://192.168.1.41:8081
  http://10.10.206.91:8081
```
You can notice that it automatically chose a different port `8081` since Pact local server is already running on `8080`. We can reach the server at `http://localhost:8081` so open the address in your browser and if everything works correctly you see something similar to the screenshot below:

![first-kadena-dapp](/img/docs/basics/quickstart/first-kadena-dapp.png)

When the page is loaded, the "Hello from Kadena ⛓🕸!" message is retrieved as a result of calling the `kapy.hello` smart contract function that we've deployed to our local pact server.

Congratulations! You've done it, our first Kadena Dapp is completed!. We hope this guide was useful to quickly learn the basics of Kadena Dapp development. For more advanced tutorials visit our [tutorials section](#).

