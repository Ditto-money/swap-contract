# Incentivized swap deposit contract

Smart contract for the incentivized swap of selected Ethereum tokens to DITTO. Lets users deposit selected Ethereum tokens and receive DITTO on Binance smart chain in exchange.

## TOC

  * [Deployment and admin functions](#deployment-and-admin-functions)
    + [Starting and ending a swap](#starting-and-ending-a-swap)
  * [End-user functions](#end-user-functions)
  * [Swap execution](#swap-execution)
    + [How DITTO claims are stored](#how-ditto-claims-are-stored)
- [Ropsten testnet deployment](#ropsten-testnet-deployment)

## Deployment and admin functions

The smart contract is deployed without constructor arguments.

Before a swap can be started, the admin must add at least one supported input tokens using the function `addInputToken(address, rate)` where `address` is the tokens address and `rate` is the USD price of the token with 3 decimals. E.g.:

```
addInputToken(0xE5301fF14399AE0ae7Fc98E3D6e2Df3557B83438,950)
```

The DITTO/USD rate must be set as well ahead of a swap (it is set to 1000/$1 initially).

```
updateDittoRate(1100)

```

### Starting and ending a swap

Start the swap using `startSwap(userCap, totalCap, bonusMultiplier)`.

- userCap: Maximum DITTO output per user (9 decimals)
- totalCap: Total DITTO available in this batch (9 decimals)
- bonusMultiplier: = Set to 100 by default = 100% exchange value. Add a bonus by setting this to values higher than 100: E.g. if set to 105% users will receive 5% extra DITTO in the swap.

Users will be able to swap until the totalCap is reached. To end the swap the admin needs to call `endSwap()`.

## End-user functions

Getting information about the supported input tokens and rates:

```
struct InputToken {
  string ticker;
  uint256 decimals;
  uint256 usdRate; // Token price in USD (4 decimals, e.g. 981 = $0.981)
}
    
mapping(address => InputToken) public inputs;
 
function numberOfInputs() external view returns (uint256); //  Number of supported input tokens
address[] public inputAddresses; // list of supported input token addresses

function getDittoOutputAmount(uint256 amount, address inputAddress) public view returns (uint256); // Amount of DITTO returned in exchange for <amount> of token at <inputAddress>
uint256 public dittoUSDRate;

```

Getting information about the currently active swap:

```
struct Swap {
  uint256 userCap; // max DITTO that can be swapped by user
  uint256 totalCap; // total amount of DITTO available in batch
  uint256 bonusMultiplier;
  uint256 totalClaimed; // amout of DITTO already claimed in this batch
}

function activeSwapInfo() internal view returns swapInfo;

function remainingTokensInActiveSwap() external view returns (uint256); // Amount of DITTO reminaing in this batch
function remainingTokensForUser(address _addr) external view returns (uint256); // Amount of DITTO that can still be swapped by address _addr
```

## Swap execution

"Swapping" happens as follows:

- The users deposits some amount of the supported input tokens
- The smart contract calculates the respective amount of DITTO returned, emits an event and stores the user's DITTO claim in the `claims` mapping.
- On the BSC side, the event gets logged and the swaped DITTO gets transferred to the user's BSC wallet.

To swap, the user has to create an allowance for the swap contract and call the `swap` function.

```
function swap(address inputTokenAddress, uint256 amount) external; 
```

### How DITTO claims are stored

On successful swap the smart contract emits an event:

```
event SwapDeposit(address depositor, address input, uint256 inputAmount, uint256 outputAmount);
```

User claims are also stored in the mapping `claims`:

```
mapping(uint256 => mapping (address => uint256)) public claims;
```

For instance, to get the amount of DITTO the user 0xaaaaa(..) should receive from the first swap (swapIndex == 0):

```
claims[0][0xaaaa(...)]
```

# Ropsten testnet deployment

- [MockERC20 1 - "AMPL" 9 decimals: 0xE5301fF14399AE0ae7Fc98E3D6e2Df3557B83438](https://ropsten.etherscan.io/address/0xE5301fF14399AE0ae7Fc98E3D6e2Df3557B83438)
- [MockERC20 2 - "BASED" 18 decimals: 0xED8cB605B4172ED9c1223e3242D1310698d26afE](https://ropsten.etherscan.io/address/0xED8cB605B4172ED9c1223e3242D1310698d26afE)
- [DittoTokenSwap: 0xFDaCD496EfFB198C81Fb5E74F156e889f4ecCF91](https://ropsten.etherscan.io/address/0xFDaCD496EfFB198C81Fb5E74F156e889f4ecCF91) 

