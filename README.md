# ETCswap V3 Staker

This is the canonical staking contract designed for [ETCswap V3](https://github.com/etcswap/v3-core), forked from Uniswap V3 Staker.

## ETCswap Deployments

| Network          | Staker Address                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------- |
| ETC Mainnet      | [0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9](https://etc.blockscout.com/address/0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9) |
| Mordor Testnet   | [0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9](https://etc-mordor.blockscout.com/address/0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9) |

## Reward Token

The V3 Staker supports any ERC20 token as a reward. ETCswap uses a dedicated reward token with admin-controlled mint/burn:

| Network          | Reward Token Address                                                                     |
| ---------------- | ---------------------------------------------------------------------------------------- |
| ETC Mainnet      | *Deploy using script below*                                                              |
| Mordor Testnet   | *Deploy using script below*                                                              |

### Deploying the Reward Token

A mintable/burnable reward token contract is available at `../scripts/DeployRewardToken.s.sol`.

**Deploy to Mordor testnet:**
```bash
forge script ../scripts/DeployRewardToken.s.sol:DeployRewardToken \
  --rpc-url https://rpc.mordor.etccooperative.org \
  --broadcast \
  --private-key $PRIVATE_KEY
```

**Deploy to ETC mainnet:**
```bash
forge script ../scripts/DeployRewardToken.s.sol:DeployRewardToken \
  --rpc-url https://etc.rivet.link \
  --broadcast \
  --private-key $PRIVATE_KEY
```

**Custom token parameters:**
```bash
TOKEN_NAME="My Reward Token" \
TOKEN_SYMBOL="MRT" \
TOKEN_DECIMALS=18 \
INITIAL_SUPPLY=1000000000000000000000000 \
forge script ../scripts/DeployRewardToken.s.sol:DeployRewardToken \
  --rpc-url https://etc.rivet.link \
  --broadcast \
  --private-key $PRIVATE_KEY
```

### Reward Token Functions

The reward token has the following admin-only functions:

```solidity
// Mint new tokens (admin only)
function mint(address to, uint256 amount) external onlyOwner;

// Burn tokens from an address (admin only)
function burn(address from, uint256 amount) external onlyOwner;

// Token holders can burn their own tokens
function burnSelf(uint256 amount) external;

// Transfer ownership
function transferOwnership(address newOwner) external onlyOwner;
```

**Mint tokens using cast:**
```bash
cast send <REWARD_TOKEN> 'mint(address,uint256)' <recipient> <amount> \
  --rpc-url https://etc.rivet.link \
  --private-key $PRIVATE_KEY
```

## Creating Staking Incentives

Once you have the reward token deployed, you can create staking incentives for V3 pools.

### 1. Approve the Staker to spend reward tokens

```bash
cast send <REWARD_TOKEN> 'approve(address,uint256)' 0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9 <amount> \
  --rpc-url https://etc.rivet.link \
  --private-key $PRIVATE_KEY
```

### 2. Create an incentive

```solidity
// IncentiveKey struct
struct IncentiveKey {
    IERC20Minimal rewardToken;
    IUniswapV3Pool pool;
    uint256 startTime;
    uint256 endTime;
    address refundee;
}
```

```bash
# Create incentive using cast
cast send 0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9 \
  'createIncentive((address,address,uint256,uint256,address),uint256)' \
  '(<REWARD_TOKEN>,<POOL_ADDRESS>,<START_TIME>,<END_TIME>,<REFUNDEE>)' \
  <REWARD_AMOUNT> \
  --rpc-url https://etc.rivet.link \
  --private-key $PRIVATE_KEY
```

### 3. Users stake their NFT positions

Users with V3 liquidity positions (NFTs) can stake to earn rewards:

1. Transfer NFT to staker: `nftPositionManager.safeTransferFrom(owner, staker, tokenId)`
2. Stake in incentive: `staker.stakeToken(incentiveKey, tokenId)`

### 4. Users claim rewards

```bash
cast send 0x12775aAf6bD5Aca04F0cCD5969b391314868A7e9 \
  'claimReward(address,address,uint256)' \
  <REWARD_TOKEN> <RECIPIENT> <AMOUNT> \
  --rpc-url https://etc.rivet.link \
  --private-key $PRIVATE_KEY
```

## Related Contracts

| Contract                          | ETC Mainnet                                      | Mordor Testnet                                   |
| --------------------------------- | ------------------------------------------------ | ------------------------------------------------ |
| V3 Factory                        | `0x2624E907BcC04f93C8f29d7C7149a8700Ceb8cDC`     | `0x2624E907BcC04f93C8f29d7C7149a8700Ceb8cDC`     |
| NFT Position Manager              | `0x3CEDe6562D6626A04d7502CC35720901999AB699`     | `0x3CEDe6562D6626A04d7502CC35720901999AB699`     |
| WETC                              | `0x1953cab0E5bFa6D4a9BaD6E05fD46C1CC6527a5a`     | `0x1953cab0E5bFa6D4a9BaD6E05fD46C1CC6527a5a`     |

## Original Uniswap Deployments (Reference)

For reference, original Uniswap V3 Staker deployments:

| Network          | Explorer                                                                                 |
| ---------------- | ---------------------------------------------------------------------------------------- |
| Mainnet          | <https://etherscan.io/address/0xe34139463bA50bD61336E0c446Bd8C0867c6fE65>                  |
| Arbitrum One     | <https://arbiscan.io/address/0xe34139463bA50bD61336E0c446Bd8C0867c6fE65>                   |
| Optimism         | <https://optimistic.etherscan.io/address/0xe34139463bA50bD61336E0c446Bd8C0867c6fE65>       |
| Base             | <https://basescan.org/address/0x42be4d6527829fefa1493e1fb9f3676d2425c3c1>                  |

## Links

- [Contract Design](docs/Design.md)

## Development and Testing

```sh
yarn
yarn test
```

## Gas Snapshots

```sh
# if gas snapshots need to be updated
$ UPDATE_SNAPSHOT=1 yarn test
```

## Contract Sizing

```sh
yarn size-contracts
```
