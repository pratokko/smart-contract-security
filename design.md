## Principles of Designing Secure Smart Contracts

**what does a well designed smart contract look like?**

Below are some approaches that you want to take when designing smart contracts

        1. Less Code -> The less code that you have the less bugs that come up. If the smart contract you are writing can have 100 lines of code then there is no need to have the same contract have 200 lines or 150. Always remember the law that states the size(lines of code) of a software project increses the probability of bugs -> To achieve less code always strive to be extremely picky about your storage variables. Avoid extra craft in your storage variables which could lead to superfulous code in your functions to update the superfluous variables.
   
   **An example of an unnecessary storage variable that could lead to superfluous code**
   
 ```solidity
        pragma solidity 0.8.19;

        contract RedundantMapping {
         enum OrderState {Pending, Fulfilled, Cancelled}

         struct Order {
            uint256 orderId;
            uint256[] items;
            OrderState state;
         }

         mapping(uint256 => uint256) orderToPositionId;
         mapping(uint256 => Order) orders;

         function createOrder (uint256 _orderId, uint256[] memory _items) external {
            orders[_orderId] = Order(_orderId,  _items, OrderState.pending);
            OrderToPositionId[_orderId] = _orderId
         }
}
        
 ```

        In this case the mapping of positionId is not required and could be added as part of the struct, this means we won't need it in the createOrder function
```solidity

        struct Order {
            uint256 orderId;
            uint256 positionId;
            uint256[] items;
            OrderState state;
        }
        
```

Note that from removing one storage variable like we did we will remove unnecessary lines of code. Assume you had multiple storage variables as above your program will achive the less code principle. 
   


   2. **loops** -> using for-loops could end up in some DOS Atacks and should therefore be used with caution. Take an example of a perpetuals context, if you wanted to calculate the total open interest of a market, you cannot loop through every position because there is a limit of the protocol to only have enough positions that could fit within the block gas limit in that loop. loops have an indeterminant amount of gas and should be avoided or used with caution if there arent other ways around it.
   3.  **Be very explicit about what we want users to do in an application** -> Users should not be allowed to pass in inputs that make actions that we dont expect. Take an example of user who wants to create a position with 0 size and 0 collateral in a perpetuals exchange. The position does not make sense to exist and should be disallowed and cut of at the beginning of function execution. 
   **NOTE** A lot of security findings are found on the basis of what unexpected things are allowed and what can the unexpected things do.
   4. **Handle all cases** -> This is quite obvious but usually very hard to do since developers tend to think from their point of view. You are more likely to miss the distinct set of cases  that you did not think of initially, the issues arise from weird situations like **stablecoin depeg**. Say a protocol assumes that the value of USDC will always be $1.00 then that should never be the case. Another good example is an **insolvent liquidation**, this is a very common issue in perpetual protocols. In a perpetuals exchange we have people who earn profits and we also have those that make loses and need to be liquidated. Lets take an example of a user that has $1000 in collateral and  $500 in losses  assuming the liquidation fee is 0 they have $500. But what happens in a case that one has $1000 in collateral but $1100 in losses? As a developer you always think in such a way that there is a keeper that will liquidate such positions before we get to that point. Well that is not usually the case, depending on the token that is being traded the prices might crush really quickly, keepers can go down, a DOS like front running can be done to delay a keepers transaction, block stuffing and many other reasons that will cause the liquidation not to occur. From a smart contract perspective the keepers functionality is not included in the smart contract and the smart contract itself should handle the cases where keepers might fail, this can limit the losses
   ```solidity
   $1000 - $1100 this will equal to an underflow panic revert
   
   ```

   this position above cannot be liquidated. This becomes even more dangerous if a protocol allows you to put $0 in collateral and cover it by unrealized profits a  slight change in price even $1 in loss and we have an underflow panic revert. Handle all this edge cases. **NOTE** the above case can be handled if a smart contract handles the case as below 

   ```solidity
   $1000 - $1100 = -$100
   
   ```
   by doing so, yes the protocol is hit by a $100 loss but  this is way better than a panic underflow revert. a small thing to note is in such cases the liquidation fee ought to be lower so that the liquidation is fair to the liquidators.
   


    5. **Parallel Data Structures** -> This is related to the storage variable layout and less code and it is more of being very smart about your storage variable choices. It occurs when you have two pieces of state that are tracking the same thing. Lets take a look at an example below.

```solidity

//Position is a struct in this case

mapping(address => Position) positionsMapping; // tracks positions but now you can add an address as key and get position as a value

Position[] private positions; // tracks all the open positions on a perpetuals protocol

```

The above code shows two ways to acctually track the same data except they are being stored in different slots in the evm. Lets take an example of a position A that is on the list but not in the mapping. or rather what if position A is updated but the updated version is on the list but not on the mapping? An even more tricky case is when say positions[A] in an array is removed, that means the last index positions.length -1 has to take the space of the position that is removed in our case A, and  pop() the last, basically the last position now becomes the index of A, lets assume our struct has a uint256 index and we fail to update the new index of our last position which is now at the index of A on the array! Makes sense ? below is some illustration

```solidity

[A, B, C, D] // this is our original array

<!-- now lets remove the B which is in the second place -->
// we remove it and our last element D takes the space making our new array to look like this after we pop the last element in the array

[A, D, C] // if D is at the position of B but with its initial index it causes issues



```
**NOTE** use can use an enummerable mapping to replace this   parallel data structures
   


