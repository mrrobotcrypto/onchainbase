# BASE-YAS
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Escrow {
    address public buyer;
    address public seller;
    address public escrowAgent;
    uint256 public amount;
    bool public productDelivered;

    enum State { Created, Funded, Shipped, Completed, Refunded }
    State public currentState;

    modifier onlyBuyer() {
        require(msg.sender == buyer, "Only the buyer can call this");
        _;
    }

    modifier onlySeller() {
        require(msg.sender == seller, "Only the seller can call this");
        _;
    }

    modifier onlyEscrowAgent() {
        require(msg.sender == escrowAgent, "Only the escrow agent can call this");
        _;
    }

    modifier inState(State _state) {
        require(currentState == _state, "Invalid state");
        _;
    }

    constructor(address _buyer, address _seller, address _escrowAgent) {
        buyer = _buyer;
        seller = _seller;
        escrowAgent = _escrowAgent;
        currentState = State.Created;
    }

    function fund() public payable onlyBuyer inState(State.Created) {
        amount = msg.value;
        currentState = State.Funded;
    }

    function confirmShipment() public onlySeller inState(State.Funded) {
        currentState = State.Shipped;
    }

    function confirmDelivery() public onlyBuyer inState(State.Shipped) {
        productDelivered = true;
        payable(seller).transfer(amount);
        currentState = State.Completed;
    }

    function refund() public onlyEscrowAgent inState(State.Funded) {
        payable(buyer).transfer(amount);
        currentState = State.Refunded;
    }
}

