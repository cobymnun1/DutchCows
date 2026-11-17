# Dutch CoWs: Programmatic Dutch Auctions for CoW Protocol

Applicant: Coby ([@Cobymnun](https://x.com/Cobymnun))
Code: [github.com/cobymnun1/DutchCow](https://github.com/cobymnun1/DutchCow)
Docs: `docs/` in the repo (integration guide and full reference)
Grant type: Milestone-based
Duration: 45 days
Requested: $4800 USD equivalent in xDAI or COW, paid on Ethereum Mainnet per CIP-82

Fields in [brackets] are mine to fill in before posting.

## Summary

Dutch auctions, where orders start priced high and fall accross a fixed schedule and range, are the most popular order type across existing DEXs. Dutch CoWs lets a smart contract swap a token on CoW Protocol through a dutch auction in one transaction; allowing developers to build safe and efficiently routed orders directly into their on-chain applciations without running any off-chain infra. Target usecases include treasuries, DAOs, vaults, funds, and any other complicated and sensitive protocols / dapp that want to keep all their logic on-chain 

A live, working version is currently deployed on Base Mainnet, but only I have used it. This grant aims to fund a small paid beta and developer outreach campaign to get users and developers testing it under both friendly and adversarial conditions. Further steps not included in the scope of this grant are an independent audit and/or further developer outreach. In the case of a sucessful beta, progression is expected to be scoped based on committee input.

## Problem

Many routers are easily accessible from a frontend but much harder to reach from a contract. This hinders developer ability to ensure full composability and forces use of bots, keepers, or API clients running off-chain, introducing trust assumptions and vulnerabilities. 

Since a contract can't sign an order or poll an orderbook on its own, developers have historically needed these dependencies to avoid the economic and security failures of virtually all other options. Recent developments, most notably Uniswap X, have introduced solid intra-protocol options for developers, but CoW protocol allows Dutch CoWs to fill across virtually any pool on any protocol at optimal prices. ComposableCoW and the watch tower make this possible, but Dutch CoWs maximizes developer accessability through a dead simple interface and familiar order structure that abstracts away the conditional order handler, ERC-1271, and all the rest of composable CoW's quirks and complexities.

With the rapidly expanding adoption of on-chain hooks and callbacks, Dutch CoWs can help CoW take advantage of the new fine-grained programability they offer. This is exactly the case Dutch Cows arose from, my personal need for safe on-chain swapping inside of a uniswap v4 hook-based index fund protocol. I wanted to support non-uniswap v2-4 tokens without breaking fees or rebalancing and without leaving the chain, something virtually impossible today without Dutch CoWs. 

## What exists now

Contracts are in `src/` and deployed on Base (chain 8453):

| Contract | Role |
| --- | --- |
| `DutchCoWAuctioneer` | Entry point. Validates params, deploys one escrow per auction, registers the order. |
| `DutchAuction` | ComposableCoW handler. Computes the current price and verifies settlements. |
| `DutchCowEscrow` | Per-auction clone. Holds the sell token, signs via ERC-1271, refunds after expiry. |

Integration, from the contract that holds the sell token:

```solidity
// Client contract Predicts their auction's escrow address
address escrow = auctioneer.predictEscrow(params);

// Client then approves that address for the amount it wants to auction
IERC20(sellToken).approve(escrow, sellAmount);

// Cliet places orders with predefined params (see below)
auctioneer.placeOrder(params);

//Auctioneer's required parameters
AuctionParams memory params = AuctionParams({
    sellToken: sellToken,       // address
    buyToken: buyToken,         // address
    sellAmount: sellAmount,     // uint256
    maxAmount: maxAmount,       // uint256
    receiver: address(this),    // address
    startBlock: block.number,   // uint256 
    validFor: validFor,         // uint32
    slippageBps: slippageBps    // uint32
});
```

The price holds at `maxAmount` for 5 blocks (equivelent to 1 second on Base today), then falls in equal steps to a floor set by `slippageBps` over `validFor` blocks. A solver may pay more than the current price but never less.

Fee: 0.1% of volume as a CoW partner fee in the order's app data is the only fee charged on a sucessful swap. 

### Safety properties

- **One escrow per auction.** Auctions never share a balance, so a bug or a weird token in one auction can't touch another.
- **No admin.** The escrow is non-upgradeable with no owner key, pause, or fee switch. `initialize` can be called once and only by the auctioneer.
- **Parameters are immutable clone args** and are checked against what the auctioneer passes in.
- **Settlement is constrained field by field.** `verify` rejects any order whose tokens, receiver, amount, app data, kind, fee, or balance type differ from the auction. The only freedom a solver has is paying more.
- **Funds come back.** After the auction window anyone can call `releaseFunds`. It removes the conditional order first, then pays out to the original caller in a separate call, so a blacklisted or paused token can't block the removal. A 0.1% tip goes to whoever triggers the release as to incentivise third parties to set up bots to facilitate this process on client contract's behalf. 
- **The receiver must be the caller.** An order can't send proceeds to a third party.

## Testing so far ------ Edited to here so far 

So far I have tested a variety tokens through the swap via a few simple call forwarding contracts (0x0dE8769698fAf248E696058fF6900bAD09a790B1, 0xe537e7206526486A2b4fBe7679Ee779734462437, 0x897FB2145CBd860aF27d08d39cFB1d8f5c1B6B4e) variety of different types of tokens as well as run many usage and  It has not been independently audited. The full log is [`cow-grant/tests performed.md`](tests%20performed.md), and every Base transaction is a row in [`cow-grant/base-mainnet-tests.csv`](base-mainnet-tests.csv). New results go in those two files first. Foundry reruns replace the one-line result in this section and the "Latest automated suite" section of [`test/DUTCH_AUCTION_TESTS.md`](../test/DUTCH_AUCTION_TESTS.md). Older runs stay in that report.

The latest recorded Foundry run of `test/*.t.sol` was **30 passed, 0 failed, 1 skipped**. The skip is the Base fork test, which only runs when a fork URL is set. Those 30 tests cover the price schedule, the place and release rules, example integrators, and mock tokens with odd decimals, transfer fees, rebasing, blacklist, pause, and lying balances. A separate local Anvil fork posted decaying orders through a watch tower into a mock orderbook. That was not a production fill.

Twenty-three live auctions ran on Base and twenty-two filled at an average cost of 0.13%. One USDC to WETH auction started too far above market and did not allocate enough slippage to fill. It's tokens were succesfully reclaimed from escrow through `releaseFunds` with a .01% tip going to the caller. Theses tests covered dollar and euro stablecoins, wrapped-BTC, staked-ETH, Aave aToken, multiple protocol governance tokens, multiple different launchpad tokens, some meme coins, and a few long-tail tokens. Total USD value sold in the orders filled was **$22.9702** and total value of tokens recieved was **$22.9394**. That is a total of **$0.0308** under market, or **0.13%**. Four fills came back above the market: Fren Pet by **$0.0305** (3.05%), cbBTC by $0.0029 (0.23%), cbETH by $0.0027 (0.26%), and BNKR by $0.0012 (0.12%). The largest miss was PEPTAI, $0.0121 under the market, or 1.16%. Amounts, addresses, transaction hashes, posted quotes, and DefiLlama prices at fill time are in [`base-mainnet-tests.csv`](base-mainnet-tests.csv).

## Plan

### Milestone 0: Working Deployment & Docs (Already Done) 

- Dutchcows.fyi serves as the core implementation guide and documentation repository. Further documentation exists in the github repository under /docs and /test. 

### Milestone 1: Beta setup (weeks 1) - Funds unlocked on completion: $3000 in beta funding.

- Build a web interface for users to connect a wallet and runs an auction through a forwarding contract to emulate the behavior of a real contract interaction. This is the lowest paying option and exists so people can try the full flow without writing any code.
- Write the beta rules, a short integration tutorial, and a bug report template.
- Publish an agents.md that testers can pass into their own AI coding agent along with their prompt. It describes the system and its rules but contains no contract code, so each integration must be further prompted by the tester to produce a full contract. This is done to ensure developers integrate acorss differentiated use cases in testing.

Deliverable: Live, working web interface, beta contract verified, rules & materials published.

### Milestone 2: Paid beta (weeks 2 though 5) - Funds unlocked on completion: $1200 in personal compensation 

- Get real usage on the Base deployment by paying individuals to test more tokens swaps and try more contract integrations. Rewards will be paid in USDC on base. Under either case we intend one reward per person per tier (see below) with up to 2 specific integration case bonuses. Payments are capped by grant allocation budget and will be payed out first come first served.

| Tier | What you do | Reward |
| --- | --- | --- |
| Web tester | Run a complete auction through the web interface and submit feedback, an email address, and optionally a sm handle. | $5 |
| Integrator | Write a small contract that calls the auctioneer, in any form, and run an auction with it on Base. | $20 |
| Hook bonus | Do the integration from inside a hook, such as a Uniswap v4 hook, a CoW pre or post hook, or a vault or fund callback. | +$5 |
| Odd-token test | Run an auction with a token that has quirks (fee-on-transfer, rebasing, no-return `approve`, blacklistable) and report both the quirks in question and what happened. | $10 |
| Refund test | Let an auction expire unfilled and recover the funds through `releaseFunds`, then report. | $5 |
| Docs fix | Correct something wrong or unclear in the guide or reference, accepted as a PR. | $10 |
| Bug report | A valid bug. Low severity (wrong revert, documentation error, ui bug) up to $25. Medium severity (funds stuck temporarily, price curve wrong) up to $100. High severity (loss or theft of funds) up to $500, decided with the Steward. | $20 to $500 |

A few notes on running it:

- Duplicate wallets or copy-pasted contracts will be treated as multiple entries and result in a reduced reward or disqualification at my discretion.  
- There is little to no oppourtunity for reward farming, fund release bots are the exception and will reviece a maximum of .01% of unfilled orders. This is intentional as community run release bots are encouraged. 
- Testers keep the buy tokens they or their contracts receive and are paid out seperatley. 
- Everything found goes in a public issue list.
- We seek commitee and steward input on paymout structure given experience with similar beta programs on other experimental CoW features. This list is ammenable.  

Deliverable: at least 100 web testers, 20 integrations, of which at least 15 fulfill a secondary reward criteria and 10 fulfill 2. We hope for more and based the budget allocation on 250 total testers at these proportions. Aditionally a public log of rewards paid and a list of issues found and fixed will be published to DutchCoWs.fyi, X, the CoW DAO discord server, and anywhere else requested by comitee or users. 

### Milestone 3: Beta report and next steps (weeks 5 to 6) - Funds onlocked on completion: TBD based on comittee feedback. Minimum expected: $600 for promotion of 3-6 X posts targeting developers. 

I'll fix what the beta found, publish a report, and then decide the next step with the committee. I see two sensible options and want input before I commit to either.

**Option A: audit.** Commission an independent audit of the three final core contracts (Auctioneer, Escrow Implementation, Handler) and their deployment bytecode, fix and findings, and publish the report. 

**Option B: developer outreach.** Develop and/or contract an integration library, market feature to developers, and work with any interested devs or teams to integrate.

My personal opinion is that the audit is the ideal path but may be prohibitiably expensive. An audit minimizes user risk and outreach effectiveness is impinged if early integration uncovers missed bugs.

Deliverable: Public beta report, fixes released, and a decision on next steps.

### Milestone 4 (optional): Other chains

If the beta goes well and/or an audit is underwent, deployments to Ethereum mainnet and other CoW-supported chains would be a natural progression direction. The only necessary changes are around the block-time parameters given different avg block times across evm chains. Real fills linked for each. I'd only start this after Milestone 3 and I'm happy to leave it out of the first grant.

## Budget

All beta rewards are paid in USDC out of the pool and capped. Numbers are my best guess and I expect the committee to push on them.

| Item | Basis | Amount |
| --- | --- | --- |
| Web tester rewards | 100 - 200 x $5 | $500 - $1000 |
| Integrator rewards | 10 - 50 x $25 | $250 - $1250 |
| Specific Base Bonus| 7 - 35 x $5 | $35 - $175 |
| Bug bounty pool | reserved | $575|
| Personal Compensation | my time building DutchCoWs, fixing it per beta findings, and overseeing testing process | $1200 |
| **Beta subtotal** | | **$4200** |
| First Promotional Campaign | 3-6 postes promoted with $100-$200 each | $600 |
| **Total** | | $4800 |

The bug bounty pool is only spent if bugs are found. Whatever is left unspent at the end goes to paying more testers, refunded into the grant, or used for other purposes with approval from comitee and/or steward. Per the process guide, each milestone is paid after review.

## How this helps CoW

- **Volume that doesn't come from a frontend.** Each contract that integrates is a recurring source of orders.
- **Hook-friendly order flow.** Hooks and funds are a natural user of CoW and don't have an easy way to reach it today.
- **Developer onboarding.** A contract developer can reach CoW with three calls, which is a better first experience than building a custom handler.
- **A Dutch auction primitive on CoW.** Teams coming from UniswapX have a familiar order type on a settlement layer with better protection.
- **Real testing.** Dozens of independent integrations before anyone's treasury relies on it.

I'd measure success by the number of unique testers, the number of substantially different integrations, fill rate and time to fill by token pair, bugs found, and volume. I'll report these in the forum thread after each milestone.

## Fee and sustainability

The 0.1% partner fee is the only income the project has. It's set in app data and goes to the recipient address in the docs. I'm not asking the DAO to subsidize DutchCoWs long term. The grant covers getting it tested and trusted, and after that the fee covers maintenance.

If the committee would rather the fee go somewhere else for the grant period or implemented a different way I'm open to discussing it.

## Risks

- **Low participation.** Paying $10 to $25 may not be enough to attract many testers. If so, I'll report that and spend less. Unspent rewards are returned.
- **Beta abuse.** One reward per person enforcement requires careful consideration and manual review. 
- **Fills depend on solvers and the watch tower.** The contracts don't run either. If solvers don't find a pair worth filling, the auction expires and funds are refunded, which is the designed failure mode. I'll report fill rates.
- **Unaudited code.** Testers should only use amounts they are fine losing. The beta rules say so up front.
- **Token quirks** are handled where possible and documented where not.

## Background and context

I built DutchCoWs specifically for my own multi-token fund protocol, [openINDEX](https://github.com/cobymnun1/openINDEX), which needs a way to sell the many tokens it collects without keepers. It is a separate project and not part of this application. The design of Dutch CoWs is inspired by an earlier handler version developed by @kayibal(https://github.com/kayibal) at an eth event in 2024 but is a different approach in many different ways.

## Payment address

0x785faa8d709d7caad63d5fbdb750d4a22600d30d - Cobymnun.base.eth on base chain, wallet accepts Ethereum mainnet as well. 

## Links

- Source: [github.com/cobymnun1/DutchCow](https://github.com/cobymnun1/DutchCow)
- Base deployment:
  - Auctioneer: [`0xE105d00b8E13Cb7f33Ea22cA2c2320f40A0E2c63`](https://basescan.org/address/0xE105d00b8E13Cb7f33Ea22cA2c2320f40A0E2c63)
  - Handler: [`0xB417180c2FD563bE2fB7833E95908B4D243bB62b`](https://basescan.org/address/0xB417180c2FD563bE2fB7833E95908B4D243bB62b)
  - Escrow implementation: [`0x2607A4712ed8C0BE50A92f71f32f0cBE0C366de9`](https://basescan.org/address/0x2607A4712ed8C0BE50A92f71f32f0cBE0C366de9)
- Grants process: [CoW Grants Process Guide](https://forum.cow.fi/t/grants-committee-for-cow-grantees-process-guide-feb-2025/2830)
