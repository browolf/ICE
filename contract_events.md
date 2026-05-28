## 1. ICE v1 existed as the original token

The old ICE token is the v1 contract:
```
0xc335Df7C25b72eEC661d5Aa32a7c2B7b2a1D1874
```
That contract is a custom ERC-20 called ICEToken. It has normal token transfers, owner controls, anti-bot/launch controls, and an airdropToWallets() function that can create balances by increasing _balances and _totalSupply. It does not contain ION bridge logic.

## 2. A bridge-compatible ICE v2 / bridge token was introduced

The bridge-router contract identifies this address as the old bridge/v2 address:
```
0x1B31606fcb91BaE1DFFD646061f6dD6FB35D0Bb5
```
In the deployer, it is labelled:

bridgeAddress = 0x1B31606fcb91BaE1DFFD646061f6dD6FB35D0Bb5; // ICE.io v.2 (Bridge)

That same address is used both as the bridge contract and as the ICE v2 token interface.

## 3. A router was created to connect old ICE v1 to the bridge token

The IONBridgeRouter was designed so users could start with ICE v1 even though the bridge worked with the v2 bridge token.

Outbound flow:

ICE v1 → router → swap to ICE v2 / bridge token → burn via bridge → ION-side credit

Return flow in that uploaded router:

ION-side event → bridge mints ICE v2 / bridge token → router swaps back → user receives ICE v1

So at that stage, the bridge system still resolved back into ICE v1 for the user.

## 4. The bridge uses oracle votes, not direct magic cross-chain movement

The bridge contract pattern is:

contract Bridge is SignatureChecker, BridgeInterface, WrappedION

The bridge has an oracle set, checks signatures, prevents duplicate votes with finishedVotings, and mints after voteForMinting() passes.

So the mechanism is:

event on one chain
→ middleware/oracles observe it
→ signatures are submitted on BSC
→ bridge contract verifies signatures
→ wrapped token is minted on BSC

The minting call path is:

voteForMinting()
→ generalVote()
→ executeMinting()
→ mint(data)

The actual mint(data) function comes from WrappedION.

## 5. Burning had to be switched on by oracle vote

The voteForSwitchBurn() function is an oracle-approved on/off switch for bridge burning.

The specific transaction you linked called:

voteForSwitchBurn(true, nonce, signatures)

So it switched burning on for that bridge contract. That does not mint or burn by itself; it enables the burn function so outbound bridging can occur.

## 6. Later, a new ION bridge/token contract appears

The current ION token address you gave is:
```
0xe1ab61f7b093435204df32f5b3a405de55445ea8
```
This appears to be a newer Bridge/WrappedION-style contract: the token contract itself is also the bridge contract. That means ION tokens on BSC are minted by calling voteForMinting() on this contract, after oracle signatures approve the event.

So the later flow becomes:

ION-chain event
→ oracle signatures
→ voteForMinting() on 0xe1ab...
→ BSC ION is minted to the receiver
7. The old ICE bridge was effectively superseded

The clean summary is:

Old phase:
ICE v1 token
→ ICE v2 bridge token
→ bridge to ION-like chain

Later phase:
ION bridge/token contract
→ mints BSC ION directly

So yes: the bridge that originally handled/minted the ICE v2 bridge asset appears to have been superseded by a newer bridge contract that mints ION instead.

Not by changing the old contract address, but by deploying a new bridge/token contract.

## Main addresses
```
ICE v1 token:
0xc335Df7C25b72eEC661d5Aa32a7c2B7b2a1D1874

Old ICE v2 / bridge contract:
0x1B31606fcb91BaE1DFFD646061f6dD6FB35D0Bb5

Current ION bridge/token contract:
0xe1ab61f7b093435204df32f5b3a405de55445ea8

Bridge/router deployer/operator:
0xcaA077BeDd289F6deFF9832b15b05A42a2C25C20

Safe multisig owner used in router deployer:
0xDFDe8108E14c70B6796bdd220454A80E849C7689
```
Plain English: ICE started as a normal token, then got a bridge-compatible v2 wrapper, and later the system moved to an ION-branded bridge/token contract where ION itself is minted on BSC through oracle-approved bridge votes.
