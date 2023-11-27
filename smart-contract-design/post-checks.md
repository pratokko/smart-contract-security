## Post Checks

It implies always enforcing invariants after an action is done executing. Lets say you expect your contract to always have X balance at the end of a function call you assert with a validation call that the balance is always greater than X 

```solidity
bal > x

```

otherwise a user took some money that they shouldn't have and we are going to revert. This acts as an emergency last ressort effort say a malicious user is able to exploit the protocol the checks will run and it will just revert.