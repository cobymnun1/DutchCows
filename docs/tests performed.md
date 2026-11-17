# Tests performed

Dutch CoWs has been tested at four levels: handler and escrow behavior, hostile token behavior, a local Base fork with a watch tower, and the Base mainnet deployment. This file is the running log. The grant application summarizes it. The commands and the older Foundry runs are in [`../test/DUTCH_AUCTION_TESTS.md`](../test/DUTCH_AUCTION_TESTS.md).

It has not been independently audited.

## How to add a result

Keep one place for each kind of result. Do not paste new runs into the older sections.

**Base mainnet.** Append one row to [`base-mainnet-tests.csv`](base-mainnet-tests.csv). Take `maxAmount`, `validFor`, and `slippageBps` from the static input inside the posted order's signature. Take the posted buy amount and the executed buy amount from the Base orderbook order. Then add one bullet in the mainnet section below, and add one row to the table in `application.md`. If the result is odd, say what should be repeated. Leave the row in the log either way.

**Foundry.** When `~/.foundry/bin/forge test --match-path 'test/*.t.sol'` is run again, replace the "Latest automated suite" section in `test/DUTCH_AUCTION_TESTS.md` and the one-line result in `application.md`. Keep the older run in the test report.

**Fork or watch tower.** Add a dated subsection under "Base fork and local watch-tower" below. Say whether the orderbook was the mock or the real CoW API.

## Base mainnet deployment tests

All four runs used `RealDutchCowTest` at `0xe537e7206526486A2b4fBe7679Ee779734462437`. That contract holds the sell token, places the auction, and keeps whatever buy token or refund it receives. It does not forward tokens to a wallet. Each run sold 1 USDC. The Base orderbook posted the orders from the escrows. Transaction hashes, order hashes, and both amounts are in [`base-mainnet-tests.csv`](base-mainnet-tests.csv).

- **1. USDC → WETH, not filled.** Start block 55369917. The auction asked for 0.000473612052545155 WETH at the start, ran for 600 blocks, and decayed 2%. That start was above the market. Nothing filled. `releaseFunds` returned 0.999 USDC to the test contract and 0.001 USDC to the caller.
- **2. USDC → WETH, filled.** Start block 55375291. Start price 0.000469003477061249 WETH, 1000 blocks, 5% decay. Settlement paid 0.000468524232915469 WETH to the test contract. The posted buy amount on the filled order was 0.000468511023410336 WETH.
- **3. USDC → Ratspeak, filled.** Doppler v3 token `0xf1e9Baa65d418A9025e1851DD2D37f1AD208bba3`. Start block 55409835. Start price 105316.649104320337197049 Ratspeak, 1000 blocks, 5% decay. Settlement paid 103405.544704553055204752 Ratspeak to the test contract. The posted buy amount was 103320.898603793466807488. Reported result: the token charges about 3%, and the payout matched the market price minus that fee.
- **4. USDC → Fren Pet, filled, needs a repeat.** Uniswap v2 token `0xFF0C532FDB8Cd566Ae169C1CB157ff2Bdc83E105`. Start block 55412727. Start price 0.3998004 Fren Pet, 1000 blocks, 5% decay. The filled order's posted buy amount was 0.39928065948 Fren Pet. Settlement paid 0.415176738867189007 Fren Pet to the test contract, which is more than that quote. A reported 4% fee did not show up as a smaller payout. Repeat this pair before treating the fee behavior as known.

Tests 5 through 16 used `RealDutchCowTest` at `0x0dE8769698fAf248E696058fF6900bAD09a790B1`. Tests 17 through 23 used `0x897FB2145CBd860aF27d08d39cFB1d8f5c1B6B4e`. Each sold a different kind of token for USDC, with a 1000-block auction and 5% decay. Settlement paid the USDC to that test contract. The posted buy amount is the filled orderbook order. The received amount is the USDC the settlement transferred.

- **5. BNKR → USDC, filled.** Bankr coin `0x22aF33FE49fD1Fa80c7149773dDe5890D3c76F3b`, an agent token. Start block 56632091. Sold 4100 BNKR. Start price 0.993849 USDC. The posted buy amount was 0.991644 USDC. Settlement paid 0.996458 USDC.
- **6. AERO → USDC, filled.** Aerodrome `0x940181a94A35A4569E4529A3CDfB74e38FD98631`, a DEX governance token. Start block 56651484. Sold 1.89 AERO. Start price 0.997799 USDC. The posted buy amount was 0.988391 USDC. Settlement paid 0.989593 USDC.
- **7. ZORA → USDC, filled.** Zora `0x1111111111166b7FE7bd91427724B487980aFc69`, a protocol token. Start block 56653481. Sold 217 ZORA. Start price 1.046752 USDC. The posted buy amount was 1.042852 USDC. Settlement paid 1.042918 USDC.
- **8. BRETT → USDC, filled.** Brett `0x532f27101965dd16442E59d40670FaF5eBB142E4`, a memecoin. Start block 56654205. Sold 288 BRETT. Start price 1.043078 USDC. The posted buy amount was 1.038346 USDC. Settlement paid 1.038665 USDC.
- **9. DEGEN → USDC, filled.** Degen `0x4ed4E862860beD51a9570b96d89aF5E1B0Efefed`, a community token. Start block 56655538. Sold 1438 DEGEN. Start price 1.109664 USDC. The posted buy amount was 1.105374 USDC. Settlement paid 1.106724 USDC.
- **10. CLANKER → USDC, filled.** tokenbot `0x1bc0c42215582d5A085795f4baDbaC3ff36d1Bcb`, a token-launch token. Basescan labels it tokenbot and the contract symbol is CLANKER. Start block 56656702. Sold 0.08279 CLANKER. Start price 1.095955 USDC. The posted buy amount was 1.091959 USDC. Settlement paid 1.092481 USDC.
- **11. VIRTUAL → USDC, filled, paid above the quote.** Virtuals `0x0b3e328455c4059EEb9e3f84b5543F74E24e7E1b`, an agent-platform token. Start block 56657125. Sold 1.978 VIRTUAL. Start price 0.525158 USDC. The posted buy amount was 0.524352 USDC. Settlement paid 1.033611 USDC, about twice that quote. A solver may pay more than the current price, so this is a higher payout.
- **12. USDT → USDC, filled.** Native bridged Tether `0xfde4C96c8593536E31F229EA8f37b2ADa2699bb2`, a 6-decimal stablecoin. The contract name is Tether USD and the symbol is USDT. Start block 56658099. Sold 0.996 USDT. Start price 0.999616 USDC. The posted buy amount was 0.992903 USDC. Settlement paid 0.992935 USDC.
- **13. USDbC → USDC, filled.** Bridged USDC `0xd9aAEc86B65D86f6A7B5B1b0c42FFA531710b6CA`, a second dollar asset. The name is USD Base Coin. Start block 56662849. Sold 1.04 USDbC. Start price 1.039986 USDC. The posted buy amount was 1.037542 USDC. Settlement paid 1.03762 USDC.
- **14. DAI → USDC, filled.** Dai `0x50c5725949A6F0c72E6C4a641F24049A917DB0Cb`, an 18-decimal stablecoin. Start block 56663629. Sold 1.04 DAI. Start price 1.040499 USDC. The posted buy amount was 1.037015 USDC. Settlement paid 1.037331 USDC.
- **15. cbBTC → USDC, filled.** Coinbase Wrapped BTC `0xcbB7C0000aB88B473b1f5aFd9ef808440eed33Bf`, 8 decimals. Start block 56679297. Sold 0.000018 cbBTC. Start price 1.282019 USDC. The posted buy amount was 1.280803 USDC. Settlement paid 1.285432 USDC.
- **16. cbETH → USDC, filled.** Coinbase Wrapped Staked ETH `0x2Ae3F1Ec7F1F5012CFEab0185bfc7aa3cf0DEc22`, a liquid staking token. Start block 56679811. Sold 0.00045 cbETH. Start price 1.037466 USDC. The posted buy amount was 1.036191 USDC. Settlement paid 1.039141 USDC. wstETH was not in this batch.
- **17. EURC → USDC, filled.** EURC `0x60a3E35Cc302bFA44Cb288Bc5a4F316Fdb1adb42`, a 6-decimal euro stablecoin, on `0x897FB2145CBd860aF27d08d39cFB1d8f5c1B6B4e`. Start block 56720112. Sold 0.941 EURC. Start price 1.044998 USDC. The posted buy amount was 1.038238 USDC. Settlement paid 1.038387 USDC. An earlier auction of the same 0.941 EURC, started at 1.149398 USDC, expired with no fill.
- **18. AIXBT → USDC, filled.** aixbt by Virtuals `0x4F9Fd6Be4a90f2620860d680c0d4d5Fb53d1A825`, the checklist agent-token address. Start block 56720689. Sold 73 AIXBT. Start price 1.037195 USDC. The posted buy amount was 1.032452 USDC. Settlement paid 1.033935 USDC.
- **19. aBasUSDC → USDC, filled.** Aave Base USDC `0x4e65fE4DbA92790696d040ac24Aa414708F5c0AB`. That is the checklist address that was uncertain, and the contract name is Aave Base USDC. Start block 56721969. Sold 1.035 aBasUSDC, and settlement took that full amount. Start price 1.03 USDC. The posted buy amount was 1.028317 USDC. Settlement paid 1.032515 USDC, above the quote and above the start. This fill did not come up short of the amount placed.
- **20. BIO → USDC, filled.** BIO `0x226A2FA2556C48245E57cd1cbA4C6c9e67077DD2`. This address is not on the remembered checklist. The name and symbol are both BIO. Start block 56722538. Sold 54 BIO. Start price 1.041183 USDC. The posted buy amount was 1.034735 USDC. Settlement paid 1.035537 USDC.
- **21. DICK → USDC, filled.** D1ckGPT `0xE183b1A4DD59Ca732211678EcA1836EE35bCE582`. This address is not on the remembered checklist. Start block 56722955. Sold 487 DICK. Start price 1.038073 USDC. The posted buy amount was 1.026496 USDC. Settlement paid 1.028026 USDC.
- **22. PEPTAI → USDC, filled.** Peptai `0x54F16Bd3996169914c84dBb2A16635100cF48A0a`. This address is not on the remembered checklist. Start block 56723727. Sold 5.87 PEPTAI. Start price 1.03 USDC. The posted buy amount was 1.028215 USDC. Settlement paid 1.029713 USDC.
- **23. AUBRAI → USDC, filled.** Aubrai by Bio `0x9d56c29e820Dd13b0580B185d0e0Dc301d27581d`. This address is not on the remembered checklist. Start block 56724497. Sold 1.59 AUBRAI. Start price 1.03232 USDC. The posted buy amount was 1.032014 USDC. Settlement paid 1.03259 USDC.

## Contract and lifecycle tests

The latest recorded run of the repository tests was:

```bash
~/.foundry/bin/forge test --match-path 'test/*.t.sol'
```

Result: **30 passed, 0 failed, 1 skipped**. The skipped test is `test_fork_quotes_through_canonical_composable_cow`, because that command was not pointed at a Base fork.

- `DutchCoWs.t.sol`: 5 tests. The price holds for the first five blocks and then decays. `validTo` rounds up near the end of the window. EIP-1271 accepts a buy amount at or above the current price and rejects a lower price, a tampered field, a bad hash, and a signature from another escrow. A fill stops polling.
- `DutchCoWAuctioneer.t.sol`: 14 tests. Placement pulls the allowance. Receiver, start block, `validFor` of 25, and slippage bounds are enforced. Zero amounts, the allowance window, an oversized approval, overflow, and a zero step are rejected. A taxed token that arrives short reverts, and a cushion within the allowance window is accepted. Release timing, the dust tip, and a second release are covered.
- `IntegratorApps.t.sol`: 3 tests. Treasury, DCA, fee, rebalance, smart-wallet, sweep, and keeper callers can place. Invalid callers and reentrancy are blocked. A second release does not pay twice.
- `AdversarialDutchCow.t.sol`: 8 tests, listed in the next section.

An earlier run of only `DutchCoWs.t.sol` and `DutchCoWAuctioneer.t.sol` passed 43 tests. Those files have since been reorganized into the counts above. The earlier command and result stay in the test report.

## Adversarial ERC20 tests

`AdversarialDutchCow.t.sol` uses mock tokens for 18, 8, 6, and 0 decimals. It covers a USDT-style approval, a false return, and a missing return. It covers a tax on the sender, the recipient, a burn, and a tax that applies only on the way to the vault. It covers an allowance cushion, a `uint96` approval limit, blacklist and pause on release, a rebase down that looks filled, a rebase up that keeps quoting, and a balance that lies. These are mock-token tests. The Fren Pet run is the live case that still needs a repeat.

## Base fork and local watch-tower

`DutchCoWsFork.t.sol` deploys the handler and auctioneer onto a Base fork, uses the canonical ComposableCoW and GPv2 settlement addresses, and checks that a placed order quotes the starting buy amount to the right receiver. It is skipped unless the command is given a fork. It does not submit to the live CoW API.

A separate Anvil fork run, recorded in the test report, deployed its own handler, auctioneer, and escrow and pointed a watch tower at a mock orderbook. The tower saw `ConditionalOrderCreated` and posted discrete sell orders whose buy amount fell `966e18 → 964e18 → 962e18 → 960e18`, with `signingScheme` `eip1271`. An early `releaseFunds` reverted. After the window, the client received 0.999e18 and the caller received 0.001e18. No live solver fill was attempted. Orders from a fork are not valid on the production orderbook.
