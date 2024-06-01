# fuel-network
Building a Smart Contract

Writing A Sway Smart Contract
Installation
To install the Fuel toolchain, you can use the fuelup-init script. This will install forc, forc-client, forc-fmt, forc-lsp, forc-wallet as well as fuel-core in ~/.fuelup/bin.

curl https://install.fuel.network | sh

Icon ClipboardText
Icon InfoCircle
Having problems? Visit the installation guide or post your question in our forum
Icon Link
.

If you're using VSCode, we recommend installing the Sway extension
Icon Link
.

Zaten yakıt yüklediniz mi?
Zaten yüklediyseniz, en güncel araç zincirinde olduğunuzdan emin olmak için aşağıdaki komutları çalıştırın.fuelup

fuelup self update
fuelup update
fuelup default latest

Icon ClipboardText
Your First Sway Project
İki işlevli basit bir sayaç sözleşmesi oluşturacağız: biri sayacı artırmak için, diğeri sayacın değerini döndürmek için.

Start by creating a new, empty folder. We'll call it fuel-project.

mkdir fuel-project

Icon ClipboardText
Writing the Contract
Move inside of your fuel-project folder:

cd fuel-project

Icon ClipboardText
Then create a contract project using forc:

forc new counter-contract

Icon ClipboardText
You will get this output:

To compile, use `forc build`, and to run tests use `forc test`
----
Read the Docs:
- Sway Book: https://docs.fuel.network/docs/sway
- Forc Book: https://docs.fuel.network/docs/forc
- Rust SDK Book: https://docs.fuel.network/docs/fuels-rs
- TypeScript SDK: https://docs.fuel.network/docs/fuels-ts

Join the Community:
- Follow us @SwayLang: https://twitter.com/SwayLang
- Ask questions on Discourse: https://forum.fuel.network/

Report Bugs:
- Sway Issues: https://github.com/FuelLabs/sway/issues/new

Icon ClipboardText
Here is the project that forc has initialized:

tree counter-contract

Icon ClipboardText
counter-contract
├── Forc.toml
└── src
    └── main.sw

1 directory, 2 files

Icon ClipboardText
forc.toml bildirim dosyasıdır (for Cargo veya for Node'a benzer) ve proje adı ve bağımlılıklar gibi proje meta verilerini tanımlar.Cargo.tomlpackage.json

Projenizi bir kod düzenleyicisinde açın ve ilk satır dışındaki her şeyi silin.src/main.sw

Her Sway dosyası, dosyanın ne tür bir program içerdiğini bildirerek başlamalıdır; Burada, bu dosyanın bir sözleşme olduğunu beyan ettik. Sway program türleri hakkında daha fazla bilgi edinebilirsiniz. Sway Book.

contract;

Icon ClipboardText
Next, we'll define a storage value. In our case, we have a single counter that we'll call counter of type u64 (a 64-bit unsigned integer) and initialize it to 0.

storage {
    counter: u64 = 0,
}

Icon ClipboardText
ABI
ABI, Uygulama İkili Arayüzü anlamına gelir. ABI, bir sözleşme için bir arayüz tanımlar. Bir sözleşme, bir ABI beyannamesi tanımlamalı veya içe aktarmalıdır.

ABI'nizi ayrı bir kitaplıkta tanımlamak ve sözleşmenize aktarmak en iyi uygulama olarak kabul edilir. Bu, sözleşmeyi arayanların ABI'yi daha kolay içe aktarmasına ve kullanmasına olanak tanır.

Kolaylık olması için, ABI'yi doğrudan sözleşme dosyasının kendisinde tanımlayacağız.

abi Counter {
    #[storage(read, write)]
    fn increment();
 
    #[storage(read)]
    fn count() -> u64;
}

Icon ClipboardText
Implement ABI
Below your ABI definition, you will write the implementation of the functions defined in your ABI.

impl Counter for Contract {
    #[storage(read)]
    fn count() -> u64 {
        storage.counter.read()
    }
 
    #[storage(read, write)]
    fn increment() {
        let incremented = storage.counter.read() + 1;
        storage.counter.write(incremented);
    }
}

Icon ClipboardText
Icon InfoCircle
storage.counter.read() is an implicit return and is equivalent to return storage.counter.read();.

Here's what your code should look like so far:

File: ./counter-contract/src/main.sw

contract;
 
storage {
    counter: u64 = 0,
}
 
abi Counter {
    #[storage(read, write)]
    fn increment();
 
    #[storage(read)]
    fn count() -> u64;
}
 
impl Counter for Contract {
    #[storage(read)]
    fn count() -> u64 {
        storage.counter.read()
    }
 
    #[storage(read, write)]
    fn increment() {
        let incremented = storage.counter.read() + 1;
        storage.counter.write(incremented);
    }
}

Icon ClipboardText
Build the Contract
Navigate to your contract folder:

cd counter-contract

Icon ClipboardText
Then run the following command to build your contract:

forc build

Icon ClipboardText
  Compiled library "core".
  Compiled library "std".
  Compiled contract "counter-contract".
  Bytecode size: 84 bytes.

Icon ClipboardText
Let's have a look at the content of the counter-contract folder after building:

tree .

Icon ClipboardText
.
├── Forc.lock
├── Forc.toml
├── out
│   └── debug
│       ├── counter-contract-abi.json
│       ├── counter-contract-storage_slots.json
│       └── counter-contract.bin
└── src
    └── main.sw

3 directories, 6 files

Icon ClipboardText
We now have an out directory that contains our build artifacts such as the JSON representation of our ABI and the contract bytecode.

Testing your Contract with Rust
Icon InfoCircle
Don't want to test with Rust? Skip this section and jump to Deploy the Contract.

We will start by adding a Rust integration test harness using a Cargo generate template. If you don't already have Rust installed, you can install it by running this command:

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

Icon ClipboardText
Next, if you don't already have it installed, let's install 
cargo generate
Icon Link
:

cargo install cargo-generate --locked

Icon ClipboardText
Now, let's generate the default test harness with the following command:

cargo generate --init fuellabs/sway templates/sway-test-rs --name counter-contract

Icon ClipboardText
⚠️   Favorite `fuellabs/sway` not found in config, using it as a git repository: https://github.com/fuellabs/sway.git
🔧   Destination: /home/user/path/to/counter-contract ...
🔧   project-name: counter-contract ...
🔧   Generating template ...
🔧   Moving generated files into: `/home/user/path/to/counter-contract`...
✨   Done! New project created /home/user/path/to/counter-contract

Icon ClipboardText
Let's have a look at the result:

tree .

Icon ClipboardText
.
├── Cargo.toml
├── Forc.lock
├── Forc.toml
├── out
│   └── debug
│       ├── counter-contract-abi.json
│       ├── counter-contract-storage_slots.json
│       └── counter-contract.bin
├── src
│   └── main.sw
└── tests
    └── harness.rs

4 directories, 8 files

Icon ClipboardText
We have two new files!

The Cargo.toml is the manifest for our new test harness and specifies the required dependencies including fuels (the Fuel Rust SDK).
The tests/harness.rs contains some boilerplate test code to get us started, though doesn't call any contract methods just yet.
Open your Cargo.toml file and check the version of fuels used under dev-dependencies. Change the version to 0.62.0 if it's not already:

[dev-dependencies]
fuels = { version = "0.62.0", features = ["fuel-core-lib"] }
tokio = { version = "1.12", features = ["rt", "macros"] }

Icon ClipboardText
Now that we have our default test harness, let's add a useful test to it.

At the bottom of test/harness.rs below the can_get_contract_id() test, add the test_increment test function below to verify that the value of the counter gets incremented:

#[tokio::test]
async fn test_increment() {
    let (instance, _id) = get_contract_instance().await;
 
    // Increment the counter
    instance.methods().increment().call().await.unwrap();
 
    // Get the current value of the counter
    let result = instance.methods().count().call().await.unwrap();
 
    // Check that the current value of the counter is 1.
    // Recall that the initial value of the counter was 0.
    assert_eq!(result.value, 1);
}

Icon ClipboardText
Here is what your file should look like:

File: ./counter-contract/tests/harness.rs

use fuels::{prelude::*, types::ContractId};
 
// Load abi from json
abigen!(Contract(
    name = "MyContract",
    abi = "out/debug/counter-contract-abi.json"
));
 
async fn get_contract_instance() -> (MyContract<WalletUnlocked>, ContractId) {
    // Launch a local network and deploy the contract
    let mut wallets = launch_custom_provider_and_get_wallets(
        WalletsConfig::new(
            Some(1),             /* Single wallet */
            Some(1),             /* Single coin (UTXO) */
            Some(1_000_000_000), /* Amount per coin */
        ),
        None,
        None,
    )
    .await
    .unwrap();
    let wallet = wallets.pop().unwrap();
 
    let id = Contract::load_from(
        "./out/debug/counter-contract.bin",
        LoadConfiguration::default(),
    )
    .unwrap()
    .deploy(&wallet, TxPolicies::default())
    .await
    .unwrap();
 
    let instance = MyContract::new(id.clone(), wallet);
 
    (instance, id.into())
}
 
#[tokio::test]
async fn can_get_contract_id() {
    let (_instance, _id) = get_contract_instance().await;
 
    // Now you have an instance of your contract you can use to test each function
}
 
#[tokio::test]
async fn test_increment() {
    let (instance, _id) = get_contract_instance().await;
 
    // Increment the counter
    instance.methods().increment().call().await.unwrap();
 
    // Get the current value of the counter
    let result = instance.methods().count().call().await.unwrap();
 
    // Check that the current value of the counter is 1.
    // Recall that the initial value of the counter was 0.
    assert_eq!(result.value, 1);
}
Expand

Icon ClipboardText
Run cargo test in the terminal:

cargo test

Icon ClipboardText
If all goes well, the output should look as follows:

  ...
  running 2 tests
  test can_get_contract_id ... ok
  test test_increment ... ok
  test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.25s

Icon ClipboardText
Deploy the Contract
It's now time to deploy . We will show how to do this using forc from the command line, but you can also do it using the Rust SDK or the TypeScript SDK.

In order to deploy a contract, you need to have a wallet to sign the transaction and coins to pay for gas. Fuelup will guide you in this process.

Setting up a local wallet
The forc-wallet plugin is packaged alongside the default distributed toolchains when installed using fuelup, so you should already have this installed if you've followed the instructions above.

To initialize a new wallet with forc-wallet, you can run the command below:

forc wallet new

Icon ClipboardText
After typing in a password, be sure to save the mnemonic phrase that is output.

Next, create a new wallet account with:

forc wallet account new

Icon ClipboardText
With this, you'll get a fuel address that looks something like this: fuel1efz7lf36w9da9jekqzyuzqsfrqrlzwtt3j3clvemm6eru8fe9nvqj5kar8.

If you need to list your accounts, you can run the command below:

forc wallet accounts

Icon ClipboardText
You can get test funds using the faucet
Icon Link
.

Deploy To Testnet
Now, you can deploy the contract to the latest testnet with the forc deploy command.

forc deploy --testnet

Icon ClipboardText
The terminal will ask for the password of the wallet:

Please provide the password of your encrypted wallet vault at "~/.fuel/wallets/.wallet":

Once you have unlocked the wallet, the terminal will show a list of the accounts:

Account 0 -- fuel18caanqmumttfnm8qp0eq7u9yluydxtqmzuaqtzdjlsww5t2jmg9skutn8n:
  Asset ID                                                           Amount
  0000000000000000000000000000000000000000000000000000000000000000 499999940

Icon ClipboardText
Just below the list, you'll see this prompt:

Please provide the index of account to use for signing:

Then you'll enter the number of the account of preference and press Y when prompted to accept the transaction.

Finally, you will get back the network endpoint where the contract was deployed, a Contract ID and the block where the transaction was signed.

Save the Contract ID, as you'll need this later to connect the frontend.

Contract counter-contract Deployed!

Network: https://testnet.fuel.network
Contract ID: 0x8342d413de2a678245d9ee39f020795800c7e6a4ac5ff7daae275f533dc05e08
Deployed in block 0x4ea52b6652836c499e44b7e42f7c22d1ed1f03cf90a1d94cd0113b9023dfa636

Icon ClipboardText
Congrats, you have completed your first smart contract on Fuel ⛽
Here is the repo for this project
Icon Link
. If you run into any problems, a good first step is to compare your code to this repo and resolve any differences.

Tweet us @fuel_network
Icon Link
 letting us know you just built a dapp on Fuel, you might get invited to a private group of builders, be invited to the next Fuel dinner, get alpha on the project, or something 👀.

Need Help?
Get help from the team by posting your question in the Fuel Forum
Icon Link
.
