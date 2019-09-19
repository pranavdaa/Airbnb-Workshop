# Airbnb-Workshop

pragma solidity ^0.5.0;
// Most tutorials use version 0.4.something but we go big here :)

contract EthBnb {
    
    // ***************
    // DEFINITIONS
    // ***************
    
    // We need to define the addresses as payable, as we need to send funds
    address payable landlordAddress;
    address payable tenantAddress;

    // We define the flat as a struct
    // We store the price, the Ethereum address of the current occupant
    // and if the flat is available.
    struct Flat {
        uint256 priceInWei;
        address currentOccupant;
        bool flatIsAvailable;
    }

    // Let's assume this landlord has 8 flats to rent.
    // This is a pretty lazy assumption, would be nice to make it flexible
    // And let the landlord add and remove flats, but it will do for this article
    Flat[8] flatDB;

    // Some functions shouldn't be called by anyone else but the landlord
    // So we use a modifier
    modifier landlordOnly() {
        require(msg.sender == landlordAddress);
        _;
    }
    
    // This runs only once, when the contract is created
    constructor() public {
        
        // We store the address creating the contract as "landlord"
        landlordAddress = msg.sender;
        
        // Filling the array with some test data
        // (all flats available, even # flats cost 0.1 ether, odd #s cost 0.2 ether)
        // Not necessary, but gives us some test data.
        for (uint i=0; i<8; i++) {
            flatDB[i].flatIsAvailable = true;
            if (i % 2 == 0) {
                flatDB[i].priceInWei = 0.1 ether;
            } else {
                flatDB[i].priceInWei = 0.2 ether;
            }
        }
    }
    
    // ***************
    // GETTERS
    // ***************
    
    // This getter tells us if a flat is available or not
    function getFlatAvailability(uint _flat) view public returns(bool) {
        return flatDB[_flat].flatIsAvailable;
    }

    // This getter tells us the price of a flat
    function getPriceOfFlat(uint _flat) view public returns(uint256) {
        return flatDB[_flat].priceInWei;
    }

    // This getter tells us who's renting the flat at the moment
    function getCurrentOccupant(uint _flat) view public returns(address) {
        return flatDB[_flat].currentOccupant;
    }

    // ***************
    // SETTERS
    // ***************

    // This function sets the availability of a flat
    function setFlatAvailability(uint8 _flat, bool _newAvailability) landlordOnly public {
        flatDB[_flat].flatIsAvailable = _newAvailability;
        // if the flat is to be set as "available", let's remove the current occupant
        // There is no 'null' in Solidity, and an address can't be set to 0 (type mismatch)
        // So we need to use address(0)
        if (_newAvailability) {
            flatDB[_flat].currentOccupant = address(0);
        }
    }
    
    // This function sets the price of a flat
    // Can only be called by the landlord
    function setPriceOfFlat(uint8 _flat, uint256 _priceInWei) landlordOnly public {
        flatDB[_flat].priceInWei = _priceInWei; 
    }
    
    // This function handles the renting
    function rentAFlat(uint8 _flat) public payable returns(uint256) {

        // Look, a tenant!
        tenantAddress = msg.sender;

        // Let's check if the tenant is sending the right amount
        // The modulo operator will be true if the amount is zero,
        // so we have to use &&
        if (msg.value % flatDB[_flat].priceInWei == 0 && msg.value > 0 && flatDB[_flat].flatIsAvailable == true) {

            // We calculate how many days the tenant has just paid for
            uint256 numberOfNightsPaid = msg.value / flatDB[_flat].priceInWei;

            // Set the availability to false
            flatDB[_flat].flatIsAvailable = false;

            // Set the tenant address in flatDB
            flatDB[_flat].currentOccupant = tenantAddress;

            // Send the rent to the landlord
            landlordAddress.transfer(msg.value);

            // The function returns the number of days paid
            // You can use this info on the frontend
            return numberOfNightsPaid;
        
        // What is the tenant sent the wrong amount?
        // Well we send it back
        } else {
            
            // we send back the funds (minus gas)
            tenantAddress.transfer(msg.value);
            
            // we return 0 days;
            return 0;
        }
        
    }   

}
