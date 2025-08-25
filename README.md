## **Paxeer Swap API Documentation**

This API provides all the necessary data for a rich user interface, powered by the Blockscout database for fast response times.

**Base URL**: `http://swap-api.paxeer.app`

### **1. Get All Supported Tokens**

Fetches a list of all tokens available for swapping, including their current price and the total liquidity in the vault. This should be called when the app loads to populate the token selection menus.

  * **Endpoint**: `GET /api/tokens`
  * **Method**: `GET`
  * **Success Response (200 OK)**:
    An array of token objects.
    ```json
    [
      {
        "name": "Wrapped Ether",
        "symbol": "WETH",
        "decimals": 18,
        "address": "0xD0C1a714c46c364DBDd4E0F7b0B6bA5354460dA7",
        "icon_url": "https://coin-images.coingecko.com/coins/images/279/large/ethereum.png",
        "value": "423718938753546012776",
        "price": 4706.61
      },
      {
        "name": "USD Coin",
        "symbol": "USDC",
        "decimals": 6,
        "address": "0x29E1f94F6b209B57eCdc1fE87448a6d085a78a5a",
        "icon_url": "https://coin-images.coingecko.com/coins/images/6319/large/usdc.png",
        "value": "20003780714556",
        "price": 0.999818
      }
    ]
    ```

### **2. Get a Swap Quote**

Calculates the expected output for a swap. This should be called whenever the user types an amount in the input field to give them a real-time quote.

  * **Endpoint**: `POST /api/quote`
  * **Method**: `POST`
  * **Request Body**:
    ```json
    {
      "tokenIn": "0x29E1f94F6b209B57eCdc1fE87448a6d085a78a5a",
      "tokenOut": "0xD0C1a714c46c364DBDd4E0F7b0B6bA5354460dA7",
      "amountIn": "100000000",
      "tokenInDecimals": 6
    }
    ```
  * **Success Response (200 OK)**:
    ```json
    {
      "amountIn": "100000000",
      "amountOut": "212174332920268128",
      "fee": "300000"
    }
    ```
      * **amountOut**: The estimated amount of `tokenOut` the user will receive.
      * **fee**: The 0.3% fee, calculated in terms of `tokenIn`.

### **3. Get Recent Transactions**

Fetches a list of the 20 most recent swap transactions. This is used to display a transaction history feed in the UI.

  * **Endpoint**: `GET /api/transactions`
  * **Method**: `GET`
  * **Success Response (200 OK)**:
    An array of decoded swap transaction objects.
    ```json
    [
      {
        "transactionHash": "0x123abc...",
        "timestamp": "2025-08-25T06:05:00.000Z",
        "user": "0xUserAddress...",
        "tokenIn": "0xTokenInAddress...",
        "tokenOut": "0xTokenOutAddress...",
        "amountIn": "1000000000000000000",
        "amountOut": "212174332920268128"
      }
    ]
    ```

-----

## **Smart Contract ABI Documentation**

For the frontend to execute swaps, it needs to interact directly with the **Vault** smart contract.

  * **Vault Contract Address**: `0x218590F75Ba9ab2E42CD1D62D7A0E309E221647E`

### **Primary Functions**

#### **1. `approve` (on the ERC20 Token Contract)**

Before a user can swap, they must first approve the Vault contract to spend their tokens.

  * **Function**: `approve(address spender, uint256 amount)`
  * **ABI Fragment**:
    ```json
    {
      "constant": false,
      "inputs": [
        { "name": "spender", "type": "address" },
        { "name": "amount", "type": "uint256" }
      ],
      "name": "approve",
      "outputs": [{ "name": "", "type": "bool" }],
      "type": "function"
    }
    ```
  * **Parameters**:
      * `spender`: The Vault contract address (`0x2185...`).
      * `amount`: The exact amount of the input token the user wants to swap.

#### **2. `swapExactTokensForTokens` (on the Vault Contract)**

This is the main function to execute the swap after approval has been granted.

  * **Function**: `swapExactTokensForTokens(address _tokenIn, address _tokenOut, uint256 _amountIn, uint256 _minAmountOut)`
  * **ABI Fragment**:
    ```json
    {
      "inputs": [
        { "internalType": "address", "name": "_tokenIn", "type": "address" },
        { "internalType": "address", "name": "_tokenOut", "type": "address" },
        { "internalType": "uint256", "name": "_amountIn", "type": "uint256" },
        { "internalType": "uint256", "name": "_minAmountOut", "type": "uint256" }
      ],
      "name": "swapExactTokensForTokens",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    }
    ```
  * **Parameters**:
      * `_tokenIn`: The contract address of the token the user is spending.
      * `_tokenOut`: The contract address of the token the user is receiving.
      * `_amountIn`: The exact amount of `_tokenIn` to be swapped.
      * `_minAmountOut`: **(Security Critical)** The minimum amount of `_tokenOut` the user is willing to accept. This protects them from price slippage. It should be calculated from the API quote. For example, for a 1% slippage tolerance, this value would be `amountOut_from_quote * 0.99`.

-----

### **Frontend Swap Workflow**

Here is the step-by-step logic for the frontend to follow:

1.  **Load Data**: On page load, call `GET /api/tokens` to populate token lists and display balances.
2.  **Get Quote**: As the user types an input amount, call `POST /api/quote` with the current input details. Display the returned `amountOut` in the UI.
3.  **Calculate Slippage**: Based on the user's selected slippage tolerance (e.g., 0.5%, 1%), calculate the `_minAmountOut` from the quoted `amountOut`.
4.  **Check Allowance**: Check if the Vault contract is already approved to spend the required `_amountIn` of `_tokenIn`.
5.  **Request Approval**: If the allowance is insufficient, prompt the user to sign an `approve` transaction on the `_tokenIn` contract.
6.  **Execute Swap**: Once the approval transaction is confirmed, prompt the user to sign the `swapExactTokensForTokens` transaction on the Vault contract, using the parameters from the previous steps.
