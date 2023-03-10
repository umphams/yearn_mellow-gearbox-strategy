# 🏄‍♀️⚙️ Yearn Mellow-Gearbox_wETH Strategy | _(AKA Mellowed-Out wETH Strategy)_
## **Strategy Details**

This strategy simply deploys wETH from Yearn wETH Vault to the Mellow Fearless Gearbox wETH Strategy.

There are attractive yields on the Gearbox protocol only accessible to approved credit managers, and Mellow is one of them. Yearn will/is explore being approved by Gearbox protocol, but in the interim strives to offer yield to Yearn users by accessing Gearbox yields via Mellow Protocol.

> 🔍 Additional reasoning can be found in the Due Diligence document [here](https://hackmd.io/@yaSscTAySm-9M5DU5kgpIA/BkeZ00eKi).

> 🚨 Please note, this repo is still under development and the code is not ready, tested, or audited. See active project tasks for most up to date areas of focus.

💬 Feel free to open up any issues as you see fit!

---

## Unique Aspects of this Strategy

**`harvest()` sequence**

Typically, yearn strategies have three asset flow scenarios:

1. Increase DR - vault function `deposit()` is used to deposit funds to strategy. Strategy gets adjusted next time bot calls `harvest()` which also acts as an accounting event.
   - `harvest()` calls `adjustPosition()` which then deposits `wantToken` into strategy assuming `_debtOutstanding` = 0.
   
2. Decrease DR - vault function `withdraw()` is used to withdraw a specified `_debtOutstanding` from strategy. Strategy calls `prepareReturn(_debtOutstanding)` which then does the accounting to see if a profit was made or not in this `harvest`. From there it uses `liquidate(_debtOutstanding)` to exit the strategy with required amount of `wantToken`

3. Keep DR and `tend()` yields: this is not used as much.

From the above three sequences, we can see a pattern needed in this strategy specifically:

1. For withdrawals: Each time `harvest(_debtOutstanding)` is called, the sequence is delayed by one `harvest()` AT LEAST (it really is based on when the `period` ends in the mellow-gearbox strategy). Okay, so `harvest()` calls `prepareReturn()` which calls `estimatedTotalAssets()` to report a profit or loss. `estimateTotalAssets()` needs to only count **claimable** `wantToken` per `harvest`. This is because `harvest()` ends up being an accounting event && a readjustment event for the strategy from the vault. 
- estimatedTotalAssets() = wantToken.balanceOf(address(this)) + <conversion of mellow LPTs in strategy.sol to wantTokens in mellow strategy>


<details markdown='1'><summary>  README Details from storming0x Template Repo
 </summary>

# Yearn Strategy Foundry Mix

## What you'll find here

- Basic Solidity Smart Contract for creating your own Yearn Strategy ([`Strategy.sol`](src/Strategy.sol))

- Configured github template with Foundry framework for starting your yearn strategy project.

- Sample test suite. ([`tests`](src/test/))

## How does it work for the User

Let's say Alice holds 100 DAI and wants to start earning yield % on them.

For this Alice needs to `DAI.approve(vault.address, 100)`.

Then Alice will call `Vault.deposit(100)`.

Vault will then transfer 100 DAI from Alice to itself, and mint Alice the corresponding shares.

Alice can then redeem those shares using `Vault.withdrawAll()` for the corresponding DAI balance (exchanged at `Vault.pricePerShare()`).

## Installation and Setup

1. To install with [Foundry](https://github.com/gakonst/foundry).

2. Fork this repository (easier) or create a new repository using it as template. [Create from template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template)

3. Clone your newly created repository recursively to include modules.

```sh
git clone --recursive https://github.com/myuser/foundry-yearn-strategy

cd foundry-yearn-strategy
```

NOTE: if you create from template you may need to run the following command to fetch the git submodules (.gitmodules for exact releases) `git submodule init && git submodule update`

4. Build the project.

```sh
make build
```

5. Sign up for [Infura](https://infura.io/) and generate an API key and copy your RPC url. Store it in the `ETH_RPC_URL` environment variable.
NOTE: you can use other services.

6. Use .env file
  1. Make a copy of `.env.example`
  2. Add the values for `ETH_RPC_URL`, `ETHERSCAN_API_KEY` and other example vars
     NOTE: If you set up a global environment variable, that will take precedence.

7. Run tests
```sh
make test
```

## Basic Use

To deploy the demo Yearn Strategy in a development environment:

TODO

## Implementing Strategy Logic

[`src/Strategy.sol`](contracts/Strategy.sol) is where you implement your own logic for your strategy. In particular:

- Create a descriptive name for your strategy via `Strategy.name()`.
- Invest your want tokens via `Strategy.adjustPosition()`.
- Take profits and report losses via `Strategy.prepareReturn()`.
- Unwind enough of your position to payback withdrawals via `Strategy.liquidatePosition()`.
- Unwind all of your positions via `Strategy.exitPosition()`.
- Fill in a way to estimate the total `want` tokens managed by the strategy via `Strategy.estimatedTotalAssets()`.
- Migrate all the positions managed by your strategy via `Strategy.prepareMigration()`.
- Make a list of all position tokens that should be protected against movements via `Strategy.protectedTokens()`.

## Testing

Tests run in fork environment, you need to complete [Installation and Setup](#installation-and-setup) step 6 to be able to run these commands.

```sh
make test
```
Run tests with traces (very useful)

```sh
make trace
```
Run specific test contract (e.g. `test/StrategyOperation.t.sol`)

```sh
make test-contract contract=StrategyOperationsTest
```
Run specific test contract with traces (e.g. `test/StrategyOperation.t.sol`)

```sh
make trace-contract contract=StrategyOperationsTest
```

See here for some tips on testing [`Testing Tips`](https://book.getfoundry.sh/forge/tests.html)

## Deploying Contracts

You can Deploy and verify your strategies if PRIV_KEY and ETHERSCAN_API_KEY are both set in the .env by using the command

```sh
make deploy
```

Before deploying, update the constructor-args variable within the Makefile to include any parameters applicable. Make sure to seperate each argument only by a space, no commas.

The deploy script is coded to deploy the "Strategy" contract within the Strategy.sol file. This can be updated by simply updating to src/YourContract.sol:YourContract.

## GitHub Actions

This template comes with GitHub Actions pre-configured. Your contracts will be linted and tested on every push to
`master` and `develop` branch.

Note though that to make this work, you must set your `INFURA_API_KEY` and your `ETHERSCAN_API_KEY` as GitHub secrets.

# Resources

- Yearn [Discord channel](https://discord.com/invite/6PNv2nF/)
- [Getting help on Foundry](https://github.com/gakonst/foundry#getting-help)
- [Forge Standard Lib](https://github.com/brockelmore/forge-std)
- [Awesome Foundry](https://github.com/crisgarner/awesome-foundry)
- [Foundry Book](https://book.getfoundry.sh/)
- [Learn Foundry Tutorial](https://www.youtube.com/watch?v=Rp_V7bYiTCM)

 </details>

