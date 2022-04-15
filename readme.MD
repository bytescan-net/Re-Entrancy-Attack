#### Author: Dr. Mo Ashouri
#### ByteScan.net
#### April 2022
### What is Re-Entrancy Attack?

On 30 April 2016, the DAO was launched on Ethereum Block 1428757. 
It was a thrilling concept: a decentralized autonomous organization governed by a smart contract on the Ethereum blockchain and managed based on
the investors' votes. The DAO token sale was a resounding success, boosting fourteen percent of the Ether in circulation.
At the time, the funds increased were equivalent to $34 million [32]. It was the most successful crowdfunding event in history.
However, the actual operation was less successful unless you consider being hacked and drained of all funds a success. 
In June 2016, a hacker withdrew 3.6 million Ether from the DAO's smart contracts, the world's first large smart contract attack. Finally, parts of the Ethereum community settled to hard-fork the Ethereum blockchain to retrieve the lost funds. However, not everyone was satisfied with the decision, and the original chain lived on as Ethereum Classic [33] [34]. The exploited vulnerability was a re-entrancy bug (called "recursive send bug" at the time) in the splitDAO() function. The relevant parts of the source code are shown below:


```
function splitDAO(
 uint _proposalID,
 address _newCurator
) noEther onlyTokenholders returns (bool _success) {
 /*** Some checks & create new DAO (elaborate computations removed) ***/
 // Move ether and assign new Tokens
 uint fundsToBeMoved =
 (balances[msg.sender] * p.splitData[0].splitBalance) /
 p.splitData[0].totalSupply;
 if
(p.splitData[0].newDAO.createTokenProxy.value(fundsToBeMoved)(msg.sender)
== false)
 throw;
 /*** More elaborate computations removed ***/

 // Burn DAO Tokens
 Transfer(msg.sender, 0, balances[msg.sender]);
 withdrawRewardFor(msg.sender); // be nice, and get his rewards
 ```
 
 
 
In brief, the splitDAO() function makes a "child DAO" contract and transfers tokens to the new DAO. The number of transferred tokens is computed according to the user's balance. Close the end of the function; a reward is sent to the caller via withdrawRewardFor(), which calls the payOut() function.
 
 
 
 ```
 function payOut(address _recipient, uint _amount) returns (bool) {
      if (msg.sender != owner || msg.value > 0 || (payOwnerOnly && _recipient
     != owner))
      throw;
      if (_recipient.call.value(_amount)()) {
      PayOut(_recipient, _amount);
      return true;
      } else {
      return false;
      }
}
 ```

Note that there is a low-level instruction in the code, i.e., call statement in the payOut() function.
