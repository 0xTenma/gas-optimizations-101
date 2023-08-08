# Gas Optimizations 101

Gas is calculated via following formula: Gas fee = units of gas used * (base fee + priority fee). Opcodes has universally agreed cost in terms of gas. This is more extensively given in the [Ethereum yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf).

1. Cache array length before loops.
2. ≠ checks are slightly gas optimized than == check.
3. make sure state variables immutable if their value never changes
4. for array elements, array[index] + is cheaper than +=
5. use unchecked when the arithmetic can’t overflow/underflow
6. payable functions are less gas consuming
7. Priotitize functions by lowering method ID: The compiler orders public and external members by their method ID.
    
    Calling function at runtime will be cheaper if the function is positioned earlier in the order because 22 gas are added to the cost for every position that came before it. The average called will save on gas if you prioritize most called functions.
    
    ```solidity
    // Unoptimized
    
    bytes32 public occasionallyCalled;
    // Method ID: 0x13216062  (position: 1, gas: 2261)
    
    function mostCalled() public {}
    // Method ID: 0xd0755f53  (position: 3, gas:  142)
    
    function leastCalled() public {}
    // Method ID: 0x24de5553  (position: 2, gas:  120)
    
    // Optimized
    bytes32 public occasionallyCalled;
    // Method ID: 0x13216062  (position: 2, gas: 2283)
    
    function mostCalled_41q() public {}
    // Method ID: 0x0000a818  (position: 1, gas:   98)
    
    function leastCalled() public {}
    // Method ID: 0x24de5553  (position: 3, gas:  142)
    ```
    
8. Variable Packing: Each storage cost gas, packing variables helps in gas optimizations.
    
    ```solidity
    contract Unoptimized {
    	uint128 Zero;
    	uint256 One;
    	uint128 Two;
    }
    
    contract Optimized {
    	uint128 Zero;
    	uint128 Two;
    	uint256 One;
    }
    ```
    
9. Deleting the unused variables helps in gas optimization.
10. Compute possible values offchain.
    
    ```solidity
    contract Unoptimized {
    	bytes32 hash = keccak256(abi.encodePacked('HashingExample');
    }
    
    contract Optimized {
    	bytes32 hash = '';
    }
    ```
    
11. Don’t shrink variables: If we use uint8, the evm has to first convert it to uint256 to work on it and conversion costs gas.
12. Using events saves gas by not accessing onchain.
13. Using libraries on different functions reduce gas cost. 
14. Fixed sized arrays are cheaper than dynamic arrays.
15. Limiting the string length in ‘require’ statements saves gas. (make it to 32bytes)
16. Mapping over arrays saves gas.
17. Avoid assigning values that has no use.
    
    ```solidity
    uint256 value; // cheap
    uint256 value = 0 // expensive
    ```
    
18. EIP1167 minimal proxy contract is a standardized, gas-efficient way to deploy a bunch of contract clones from a factory.
19. Performing operations on memory and calldata than the storage is always cheaper.
20. Use calldata instead of memory.
21. Using modifiers saves gas.
22. Short-Circuit: In the case of `||`and `&&` operators, the evaluation is short-circuited, meaning the second condition is not evaluated if the first condition already determines the result of the logical expression.
    
    To optimize for lower gas usage, arrange conditions so that the less expensive computation is placed first. This could potentially bypass the execution of the more expensive computation.
    
23. If you don’t need ERC721Enumerable, for ERC721 which saves gas: The problem with this extension is that it adds a lot of overhead to any transfer (be it the the contract transferring to the user when the user mints, or any transfer from one user to another).
24. Using ERC721A standard saves gas cost.
25. Start with Token ID 1 not zero, making the first mint much cheaper.
26. Use merkle tree for whitelisting users. [READ MORE ON REF NO: 5)
27. Opt for push over pull payment patterns: When distributing payments, it's generally more gas-efficient to use a `push` pattern, where the contract sends funds to recipients, rather than a `pull` pattern, where recipients need to withdraw their funds.
28. Batch multiple operations: If your contract performs multiple similar operations, try to batch them together in a single transaction, reducing the per-operation gas cost.
29. Favor `external` over `public` for functions: If a function is only meant to be called from outside the contract, use the `external` visibility specifier instead of `public`. `external` functions are more gas-efficient when receiving large amounts of data as arguments.
30. To minimize gas consumption, it’s advantageous to load frequently accessed state variables into memory. By doing so, you reduce the number of expensive SSTORE operations and achieve substantial gas savings
    
    ```solidity
    uint256 public total;
    
    function sumIfEvenAndLessThen99(uint[] calldata nums) external {
    	uint256 _total = total;
    
    }
    ```
    
31. Using Pre Increment Operator saves gas [CAREFUL when using it]
32. Loading array elements to memory saves gas.
33. Use view and pure functions to reduce gas.
34. User bit shift operators: Cheaper over arthmetic operators. [CAREFUL: Doesn’t revert on overflow/underflow]
    
    ```solidity
    -    uint256 alpha = someVar / 256;
    -    uint256 beta = someVar % 256;
    +    uint256 alpha = someVar >> 8;
    +    uint256 beta = someVar & 0xff;
    
    -    uint 256 alpha = delta / 2;
    -    uint 256 beta = delta / 4; 
    -    uint 256 gamma = delta * 8;
    +    uint 256 alpha = delta >> 1;
    +    uint 256 beta = delta >> 2; 
    +    uint 256 gamma = delta << 3;
    ```
    
35. Avoid redundant checks
    
    ```solidity
    // Unoptimized:
    require(balance>0, "Insufficient balance");
    if (balance>0){ // this check is redundant
        . . . //some action
    }
    
    // Optimized: 
    require(balance>0, "Insufficient balance");
    . . . //some action
    ```
    
36. Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
    
    ```solidity
    contract NestedIfTest {
    
        //Execution cost: 22334 gas
      function funcUnoptimized(uint256 input) public pure returns (string memory) { 
           if (input<10 && input>0 && input!=6){ 
               return "If condition passed";
           } 
    
       }
    
        //Execution cost: 22294 gas
        function funcOptimized(uint256 input) public pure returns (string memory) { 
        if (input<10) { 
            if (input>0){
                if (input!=6){
                    return "If condition passed";
                }
            }
        }
    }
    }
    ```
    
37. Using mutiple require statements is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
38. Calling Internal functions are cheaper.
39. Packing works on struct too. Pack structs to save gas
40. Don’t use unnecesasay require statements.
41. It is more gas efficient to revert with a custom error than a `require` with a string.
42. Don’t operate on expensive operations inside a loop, which can be costly
43. Caching variables that is used once is just waste of gas.
44. Importing entire library while only using one function isn’t necessary.
45. Setting the contructor to payable saves gas.
46. Duplicated `require()`[/](https://code4rena.com/reports/2023-03-asymmetry#g-02-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function)`revert()` Checks Should Be Refactored To A Modifier Or Function.
47. Using hardcoded address instread address(this) saves gas. [AS NEEDED]
48. Empty Blocks Should Be Removed Or Emit Something.
49. Using delete statements can save gas.
50. `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables.

[TIPS]

1. Use foundry —gas-report to check gas usage.
2. Layer 2 solutions reduces the amount of data that needs to be calculated and to be stored in on chain therefore reducing gas cost.
3. Use the optimizer on ‘On’ state to reduce gas.
4. Use solidity version 0.8.19 to gain some gas boost.

[REFERENCES]

1. [https://yamenmerhi.medium.com/gas-optimization-in-solidity-75945e12322f](https://yamenmerhi.medium.com/gas-optimization-in-solidity-75945e12322f)
2. [https://github.com/kadenzipfel/gas-optimizations](https://github.com/kadenzipfel/gas-optimizations)
3. [https://github.com/ZeroEkkusu/re-golf-course](https://github.com/ZeroEkkusu/re-golf-course)
4. [https://certik.medium.com/gas-optimization-in-ethereum-smart-contracts-10-best-practices-cbd57548bdf0](https://certik.medium.com/gas-optimization-in-ethereum-smart-contracts-10-best-practices-cbd57548bdf0)
5. [https://medium.com/@WallStFam/the-ultimate-guide-to-nft-gas-optimization-7e9289e2d88f](https://medium.com/@WallStFam/the-ultimate-guide-to-nft-gas-optimization-7e9289e2d88f)
6. [https://github.com/devanshbatham/Solidity-Gas-Optimization-Tips](https://github.com/devanshbatham/Solidity-Gas-Optimization-Tips)
