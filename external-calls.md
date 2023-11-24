## External calls

This is one of the most common exploits. ways in which external calls can lead to vulnerabilities

1. Reentrancy -> This occurs when a contract function can be interrupted and called again before the initial execution finishes. It happens when a contract calls an external contract function. Imagine a scenario where a contract function performs some actions and calls an external contract. If that external contract, during its execution, invokes back a function in the original contract that's not yet completed, it results in reentrancy.  we talk more about this and some code examples in our reentrancy.md file. to prevent this kind of attack use **CEI** this refers to the checks effects and interractions format in all your functions. What this means is that we first add our checks this include the if and require statements, then the effects this is basically what is expected of the function, and lastly we can interact with other external functions. Another way to prevent this  attack is to use a **nonReentrant** modifier that ensures a function cannot be reentered unless its execution is completed. It is usually safe to put nonReentrant in every function that changes state.
   
**NOTE** -> multiple contracts in the system can cause nonReentrant modifier in a function to still fail and be reentered. use of nonReentrant only applies to the storage of a single smart contract, it protects against reentering from one function in a smart contract into any other arbirary function in that same smart contract. Lets break this down so we can understand. Say we have multiple smart contracts A and B. We update reentrant state in contract our contracts, yes we cannot re-enter into contract A in the same transaction, but guess what we can do. Because the same system has contract B, we can enter contract A and in the same transaction enter B, if contract B reads some state in A to perform some action it reads the state and the action in B is performed. This means we have reentered in the same system.

**NOTE** Now lets look at examples of different systems. In this case system A is the curve pool we call a function of curve and changing balance of a curve pool. We cannot reenter into the curve pool itself but there are other systems that read from the view functions in the curve pool, lets give the name B to our smart contract. IF the price of curve is already altered to our benefit, B reads the manipulated price and just like that we have reentrancy on the curve pool. This kind of reentrancy is called **Read-Only-Reentrancy**

2. DOS -> Denial of Service Attacks.
Lets start by asking this questions,
 -When we make an external call can it just fail? 
 -can we just shut down an important feature in a smart contract?
 -is there a callback in a liquidation where a user can just choose to revert in their callback contract and make the liquidation to fail
 -Is there a liquidation that sends native token to an address? and is this address a smart contract that does not accept ether(the native token) to make the transaction fail.

3. Return values -> All the return values that are to be returned should be checked. This ensures we get  a return value that we are expecting and not some unexpected bytes.
4. Gas -> if you call to an external contract and dont specify the gas to use you will forward all the gas, make sure that if not trusted dont forward all the gas