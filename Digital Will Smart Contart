// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DigitalWill {
    address payable public testator;
    bool public isExecuted = false;

    struct Beneficiary {
        address payable recipient;
        uint256 sharePercentage; // Represented as a percentage (0-100)
    }

    mapping(uint256 => Beneficiary) public beneficiaries;
    uint256 public beneficiaryCount = 0;

    mapping(string => string) public instructions; // Key-value pairs for instructions

    event BeneficiaryAdded(address recipient, uint256 sharePercentage);
    event InstructionAdded(string key, string value);
    event WillExecuted(address indexed executor, uint256 timestamp);

    modifier onlyTestator() {
        require(msg.sender == testator, "Only the testator can perform this action.");
        _;
    }

    modifier notExecuted() {
        require(!isExecuted, "The will has already been executed.");
        _;
    }

    constructor() payable {
        testator = payable(msg.sender);
    }

    function addBeneficiary(address payable _recipient, uint256 _sharePercentage) public onlyTestator notExecuted {
        require(_sharePercentage <= 100, "Share percentage must be between 0 and 100.");
        beneficiaries[beneficiaryCount] = Beneficiary(_recipient, _sharePercentage);
        emit BeneficiaryAdded(_recipient, _sharePercentage);
        beneficiaryCount++;
    }

    function addInstruction(string memory _key, string memory _value) public onlyTestator notExecuted {
        instructions[_key] = _value;
        emit InstructionAdded(_key, _value);
    }

    // This is a SIMPLIFIED and potentially INSECURE way to trigger execution.
    // In a real-world scenario, this would require a more robust mechanism
    // involving oracles or trusted third parties.
    function executeWill(address payable _executor) public notExecuted {
        require(msg.sender == _executor, "Only the designated executor can execute the will.");
        require(beneficiaryCount > 0, "No beneficiaries defined.");

        isExecuted = true;
        uint256 totalBalance = address(this).balance;
        uint256 totalPercentage = 0;

        for (uint256 i = 0; i < beneficiaryCount; i++) {
            totalPercentage += beneficiaries[i].sharePercentage;
        }
        require(totalPercentage == 100, "Total beneficiary share percentage must be 100.");

        for (uint256 i = 0; i < beneficiaryCount; i++) {
            uint256 amountToSend = (totalBalance * beneficiaries[i].sharePercentage) / 100;
            (bool success, ) = beneficiaries[i].recipient.call{value: amountToSend}("");
            require(success, "ETH transfer failed for a beneficiary.");
        }

        emit WillExecuted(_executor, block.timestamp);
    }

    // Fallback function to allow the contract to receive Ether
    receive() external payable {}
}
