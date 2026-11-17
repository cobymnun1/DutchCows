Paid Beta test for Dutch CoWs: Programmatic Dutch Auctions for CoW Protocol

Author: @Cobymnun on X (https://x.com/Cobymnun) / Cobymnun1 on Github (https://github.com/cobymnun1)
Code: [github.com/cobymnun1/DutchCow](https://github.com/cobymnun1/DutchCow)
Documentation: dutchcows.fyi and `docs/` in the repo (integration guide and full reference)
Grant type: Hybrid
Duration: 35 days
Requested: $4800 USD equivalent in xDAI or COW, paid on Ethereum Mainnet per CIP-82

Experiences and Qualifications: 
  BS in Economics, Business, and Blockchain Development, experience developing blockchain based services and applications at the Stony Brook University Blockchain Business Lab as well as other projects in my personal time available on my github. 

Summary:

  Dutch auctions, where orders start priced high and fall across a fixed schedule and range, are the most popular order type across existing DEXs. Dutch CoWs lets a smart contract swap a token on CoW Protocol through a dutch auction in one transaction; allowing developers to build safe and efficiently routed orders directly into their on-chain applications without running any off-chain infra. Target use cases include treasuries, DAOs, vaults, funds, and any other complicated and sensitive protocols / dapp that want to keep all their logic on-chain 

  A live, working version is currently deployed on Base Mainnet and this grant aims to fund a small paid beta and developer outreach campaign to get users and developers testing it under both friendly and adversarial conditions. Further steps not included in the scope of this grant are an independent audit and/or further developer outreach. In the case of a successful beta, progression is expected to be scoped based on committee input.

Problem:

  Many routers are easily accessible from a frontend but much harder to reach from a contract. This hinders developer ability to ensure full composability and forces use of bots, keepers, or API clients running off-chain, introducing trust assumptions and vulnerabilities. 

  Since a contract can't sign an order or poll an orderbook on its own, developers have historically needed these dependencies to avoid the economic and security failures of virtually all other options. Some intra-protocol options exist for developers, but CoW protocol allows Dutch CoWs to fill across virtually any pool on any protocol at optimal prices. ComposableCoW and the watch tower make this possible, but Dutch CoWs maximizes developer accessibility through a hyper-simplified interface and familiar order structure that abstracts away the conditional order handler, ERC-1271, and all the rest of composable CoW's quirks and complexities.

  With the rapidly expanding adoption of on-chain hooks and callbacks, Dutch CoWs can help CoW take advantage of the new fine-grained programmability they offer. This is exactly the case Dutch CoWs arose from, my personal need for safe on-chain swapping inside of a uniswap v4 hook-based index fund protocol. I wanted to support tokens with liquidity across as many DEXs and launchpads as possible tokens without breaking fees or rebalancing and without leaving the chain, something virtually impossible today without Dutch CoWs. 

What exists now:

  Contracts are in `src/` and deployed on Base (chain 8453):

  | Contract | Role |
  | --- | --- |
  | `DutchCoWAuctioneer` | Entry point. Validates params, deploys one escrow per auction, registers the order. |
  | `DutchAuction` | ComposableCoW handler. Computes the current price and verifies settlements. |
  | `DutchCowEscrow` | Per-auction clone. Holds the sell token, signs via ERC-1271, refunds after expiry. |

  Integration, from the contract that holds the sell token:

  ```solidity

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

  // Client contract Predicts their auction's escrow address
  address escrow = auctioneer.predictEscrow(params);

  // Client then approves that address for the amount it wants to auction
  IERC20(sellToken).approve(escrow, sellAmount);

  // Client places orders with predefined params (see below)
  auctioneer.placeOrder(params);
  ```

  The price holds at `maxAmount` for 5 blocks (equivalent to 1 second on Base today), then falls in equal steps to a floor set by `slippageBps` over `validFor` blocks. A solver may pay more than the current price but never less.

  Fee: 0.1% of volume as a CoW partner fee in the order's app data is the only fee charged on a successful swap. 

| Milestones | Title | Due Date | Funding Requested |

| Milestone 0 | Working Deployment & Docs | Already Done | Funding Requested: $0 | 

  - Dutchcows.fyi serves as the core implementation guide and documentation repository. Further documentation exists in the github repository under /docs and /test. 

| Milestone 1 | Beta setup | Due 1 Week From Grant Allocation Date | Funding Requested: $600 in personal Compensation for work to this point |

  - Build a web interface for users to connect a wallet and runs an auction through a forwarding contract to emulate the behavior of a real contract interaction. This is the lowest paying option and exists so people can try the full flow without writing any code.
  - Write the beta rules, a short integration tutorial, and a bug report template.
  - Publish an agents.md that testers can pass into their own AI coding agent along with their prompt. It describes the system and its rules but contains no contract code, so each integration must be further prompted by the tester to produce a full contract. This is done to ensure developers integrate across differentiated use cases in testing.

  Deliverables: Live, working web interface, beta contract verified, rules & materials published.

| Milestone 2 | Paid beta | Due 4 Weeks from Grant Allocation Date | Fund Requested: $3000 for Beta Participation Rewards |

  - Get real usage on the Base deployment by paying individuals to test more tokens swaps and try more contract integrations. Rewards will be paid in USDC on base. Under either case we intend one reward per person per tier (see below) with up to 2 specific integration case bonuses. Payments are capped by grant allocation budget and will be paid out first come first served.

  | Tier | What you do | Reward |
  | --- | --- | --- |
  | Web tester | Run a complete auction through the web interface and submit feedback, an email address, and optionally a social media handle. | $5 |
  | Integrator | Write a small contract that calls the auctioneer, in any form, and run an auction with it on Base. | $20 |
  | Hook bonus | Do the integration from inside a hook, such as a Uniswap v4 hook, a CoW pre or post hook, or a vault or fund callback. | +$5 |
  | Odd-token test | Run an auction with a token that has quirks (fee-on-transfer, rebasing, no-return `approve`, blacklistable) and report both the quirks in question and what happened. | $10 |
  | Refund test | Let an auction expire unfilled and recover the funds through `releaseFunds`, then report. | $5 |
  | Docs fix | Correct something wrong or unclear in the guide or reference, accepted as a PR. | $10 |
  | Bug report | A valid bug. Low severity (wrong revert, documentation error, ui bug) up to $25. Medium severity (funds stuck temporarily, price curve wrong) up to $100. High severity (loss or theft of funds) up to $500, decided with the Steward. | $20 to $500 |

| Milestone 3 | Beta Results and Final Deployments | Due 5 Weeks from Grant Allocation Date | Funding Requested: $1200 (50/50 Promotion and Personal Compensation) | 

  - Fix any Bugs exposed by the Beta and write and publish a report on its results
  - Deploy the final version to Base, Ethereum, and maybe other CoW supported evm chains
  - Write and publish 3-6 X posts about Dutch CoWs and pay for $100-$200 in promotion on X

Total Funding Request: $4800 USD Equivalent

  Breakdown: 
    - $3000 in Paid Beta Funding
    - $1200 in Personal Compensation
    - $600 in Social Media Promotion

  All beta rewards are paid in USDC out of the pool and capped. Numbers are a rough estimate of optimistic participation 

  | Item | Basis | Amount |
  | --- | --- | --- |
  | Web tester rewards | 100 - 200 x $5 | $500 - $1000 |
  | Integrator rewards | 10 - 50 x $25 | $250 - $1250 |
  | Specific Base Bonus| 7 - 35 x $5 | $35 - $175 |
  | Bug bounty pool | reserved | $575|
  | Personal Compensation | my time building DutchCoWs, fixing it per beta findings, and overseeing testing process | $1200 |
  | **Beta subtotal** | | **$4200** |
  | First Promotional Campaign | 3-6 posts promoted with $100-$200 each | $600 |
  | **Total** | | $4800 |

  The bug bounty pool is only spent if bugs are found. Whatever is left unspent at the end goes to paying more testers, refunded into the grant, or used for other purposes with approval from committee and/or steward. Per the process guide, each milestone is paid after review.

Payment address:

  0x785faa8d709d7caad63d5fbdb750d4a22600d30d - Cobymnun.base.eth on base chain, wallet accepts Ethereum mainnet as well. 

Existing Safety Properties:

  One escrow per auction, escrow is non-upgradeable with no owner, `initialize` can be called once by auctioneer, Parameters are immutable clone args checked against what auctioneer passes in, `verify` rejects any order that differ from the auction, permissionless `releaseFunds`.

Testing so far:

  So far I have tested a variety of tokens through the swap via a few simple call forwarding contracts the full log is at https://github.com/cobymnun1/DutchCow/blob/master/cow-grant/base-mainnet-tests.csv. A test suite was run in a local base fork prior to this as well and can be found at https://github.com/cobymnun1/DutchCow/tree/master/test with documentation at https://github.com/cobymnun1/DutchCow/blob/master/docs/tests%20performed.md 

  Twenty-three live auctions ran on Base and twenty-two filled at an average cost of 0.13%. Amounts, addresses, transaction hashes, posted quotes, and DefiLlama prices at fill time are in the log csv linked above. 

Notes on running Beta:

  - Duplicate wallets or copy-pasted contracts will be treated as multiple entries and result in a reduced reward or disqualification at my discretion.  
  - There is little to no opportunity for reward farming, fund release bots are the exception and will receive a maximum of .01% of unfilled orders. This is intentional as community run release bots are encouraged. 
  - Testers keep the buy tokens they or their contracts receive and are paid out separately. 
  - Everything found goes in a public issue list.
  - We seek committee and steward input on payout structure given experience with similar beta programs on other experimental CoW features. This list is amenable.  


Why DutchCoWs is Good for CoW:

  - **Volume that doesn't come from a frontend.** Each contract that integrates is a recurring source of orders.
  - **Hook-friendly order flow.** Hooks and funds are a natural user of CoW and don't have an easy way to reach it today.
  - **Developer onboarding.** A contract developer can reach CoW with three calls, which is a better first experience than building a custom handler.
  - **A Dutch auction primitive on CoW.** Teams coming from UniswapX have a familiar order type on a settlement layer with better protection.
  - **Real testing.** Dozens of independent integrations before anyone's treasury relies on it.

Fee and sustainability:

  The 0.1% partner fee is the only income the project has. It's set in app data and goes to the recipient address in the docs. I'm not asking the DAO to subsidize DutchCoWs long term. The grant covers getting it tested and trusted, and after that the fee covers maintenance.

  If the committee would rather the fee go somewhere else for the grant period or implemented a different way I'm open to discussing it.

Risks:

  - **Low participation.** Paying $10 to $25 may not be enough to attract many testers. If so, I'll report that and spend less. Unspent rewards are returned.
  - **Beta abuse.** One reward per person enforcement requires careful consideration and manual review. 
  - **Fills depend on solvers and the watch tower.** The contracts don't run either. If solvers don't find a pair worth filling, the auction expires and funds are refunded, which is the designed failure mode. I'll report fill rates.
  - **Unaudited code.** Testers should only use amounts they are fine losing. The beta rules say so up front.
  - **Token quirks** are handled where possible and documented where not.



Links:

- Source: [github.com/cobymnun1/DutchCow](https://github.com/cobymnun1/DutchCow)
- Base deployment:
  - Auctioneer: [`0xE105d00b8E13Cb7f33Ea22cA2c2320f40A0E2c63`](https://basescan.org/address/0xE105d00b8E13Cb7f33Ea22cA2c2320f40A0E2c63)
  - Handler: [`0xB417180c2FD563bE2fB7833E95908B4D243bB62b`](https://basescan.org/address/0xB417180c2FD563bE2fB7833E95908B4D243bB62b)
  - Escrow implementation: [`0x2607A4712ed8C0BE50A92f71f32f0cBE0C366de9`](https://basescan.org/address/0x2607A4712ed8C0BE50A92f71f32f0cBE0C366de9)

