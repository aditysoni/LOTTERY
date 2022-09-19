//SPDX-License-Identifier: GPL-3.0
 
pragma solidity >=0.5.0 <0.9.0;

contract Lottery{
    
    // declaring the state variables
    address payable[] public players; //dynamic array of type address payable
    address immutable public manager; // declaring the address immutable so that no onc can change the owner 
                                     //   and the gas for the owner variable is compartively less now 
    
     
    constructor()                 // declaring the constructor 
    { 
     manager = msg.sender;        // intializing the manager variable with the contract that deploys the contract
    }
    
    // receive function is declared to receive Ether from participents 
    receive () payable external{
        // if player sends exactly 0.5 ETH then the forward operation will be continued
        require(msg.value == 0.5 ether,"please send 0.5 ether only");
        // appending the player to the players array
        players.push(payable(msg.sender));
     
    }
    
    // getting the contract's balance in wei
    function getBalance() public view returns(uint){
        
        require(msg.sender == manager); // only the manager is allowed to call the function 
        return address(this).balance;
    }
   
   // function that returns a long random no . which is used for getting the winner
    function random() internal view returns(uint){
       return uint(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players.length)));
    }
    
    //function to return the probability of winning the lottery   
    function probability () view public returns (uint)
    {
        return uint ((1/players.length)*100);
    }
    
    
    // selection of the winner
    function pickWinner() public{
        // only the manager can pick a winner 
        require(msg.sender == manager);
        uint r = random();
        address payable winner;
        
        // computing a random index of the array
        uint index = r % players.length;
    
        winner = players[index]; // this is the winner
        
        // transferring the entire contract's balance to the winner
        winner.transfer(getBalance());
        
        // resetting the lottery for the next round
        players = new address payable[](0);
    }

}
