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
   