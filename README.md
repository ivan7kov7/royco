# Royco: The Incentivized Action Market Protocol
![Royco Banner](./roycobanner.png)

Royco allows anyone to create a market around any onchain action. Those who wish to pay users to execute an onchain action are called "Incentive Providers" (IPs) and offer token or points incentives in exchange for an "Action Provider" (APs) to take some action, be it enter a staking vault, or execute some "recipe" of one or more smart contract interactions, each having their own market types on Royco.

## Vault Markets
Actions which deposit in staking vaults are called "Vault Markets". Vault Markets consist of a Royco 4626i Vault and a Royco Vault Orderbook.
### ERC4626i.sol
Vault Markets center around ERC4626i vaults, the 'i' standing for "incentives". 4626i vaults are created by the ERC4626iFactory by simply pointing to an existing 4626 vault and deploying a new ERC4626i wrapper to allow IPs to distribute Unipool style staking incentives to APs who deposit in the vault through the wrapper. The underlying 4626 vault must be fully compliant to vanilla 4626 behavior, meaning working previewWithdraw functions, immediate deposits/withdrawals, etc.

ERC4626i vaults are owned by the Incentive Provider who deploys them. The IP who owns a 4626i vault is solely permissioned to distribute incentives, and may add additional incentive tokens or points campaigns at any time, (for up to 20 different assets). IPs can also "extend" incentive campaigns by calling extendRewardsInterval, however to prevent dishonest AP limit order sniping, IPs must keep the new rewards rate equivalent or higher than the current rewards rate when doing so.

### VaultOrderbook.sol
The Vault Orderbook is where APs offer to deposit into 4626i vaults. APs place "limit orders" on vaults specifying the token rates they would need to receive per deposit token in order to be allocated into a vault. Once a vault is streaming the rate desired by the LP, an IP can call allocateOrder to draw the in-range orders into the pool.

Orders on Royco Orderbooks are placed without transferring tokens to the orderbook, allowing many orders to be placed off of the same tokens. Additionally, orders can be placed against not only ERC20 tokens but also against other 4626 or 4626i vaults, withdrawing from the vaults on order fill to allow APs to farm one pool while simultaneously offering to enter other pools if incentive rates increase.

## Recipe Markets
More long-tail actions which may not be easily represented by 4626 vaults instead occur through recipe markets, which heavily rely on the [operation-chaining/scripting language Weiroll](https://github.com/weiroll/weiroll). Weiroll allows IPs to write complex chained "Recipes" of smart contract interactions that can express any chain of actions you could make through an EOA.

### WeirollWallet.sol
Instead of depositing in a 4626i vault, recipe markets deposit assets into a lightweight, disposable, smart contract wallet owned by the AP with the Weiroll VM built in. This allows the AP to execute weiroll scripts without exposing all their funds to the arbitrary (and potentially malicious) interactions given by the script. Weiroll Wallets also allow locking the wallet itself, enabling IPs to ensure an AP holds their position after executing the script for some set amount of time. 

After a timelock has expired, an AP gains full control over the wallet, and is now able to call a withdrawal recipe specified in the market, or can simply pass raw calldata for the wallet to execute.

### RecipeOrderbook.sol
Recipe Orderbook functions much like Vault Orderbook, but allows Incentive Providers to also place limit orders, stating that they are willing to pay APs some amount of incentive in exchange for calling the Weiroll recipe with some amount of deposit token. Both LP and AP limit orders may be filled by the opposite parties.

## Other Contracts

### ERC4626iFactory.sol
A factory contract for spinning up new ERC4626i vaults, and tracks the current protocol fee and frontend fee.

### Points.sol
An ERC20-like contract which has no state which could be tokenized, but rather simply awards APs points via events. Points award functions are highly restricted to prevent anyone from awarding points arbitrarily.

### PointsFactory.sol
A factory contract for spinning up new points campaigns, keeps a mapping of points programs for orderbooks to quickly decide if an address belongs to a token or a points campaign.

## Getting started
Royco is built using [Foundry](https://github.com/foundry-rs/foundry)

**Installing Dependencies** ``` forge install ```

**Running Tests:** ``` forge test ```
