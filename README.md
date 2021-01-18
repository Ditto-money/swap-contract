# Incentivized swap deposit contract

Smart contract for the incentivized swap of selected Ethereum tokens to DITTO. Lets users deposit selected Ethereum tokens and receive DITTO on Binance smart chain in exchange.

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

### Getting information about DITTO claims

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
