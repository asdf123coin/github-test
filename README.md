# github-test
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.4/contracts/token/ERC20/ERC20.sol";

contract DogeInuAI is ERC20 {
    uint256 private _maxTokensPerWallet = 333333333 * 10**18; // 333 million tokens

    address private _owner;
    mapping(address => bool) private _bots;
    mapping(bytes32 => bool) private _processedTransactions; // Keep track of processed transactions


    uint256 private _totalBurnt;
    uint256 private _taxRate = 100; // 0.1% tax rate
    uint256 private _burnRate = 50; // 0.05% burn rate

    constructor() ERC20("Doge Inu AI", "AINU") {
        _mint(msg.sender, 2_000_000_000_000 * 10**18); // 2 billion tokens
        _owner = msg.sender;
    }

    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        require(!_bots[msg.sender], "Sender address is flagged as bot");
        require(!_bots[recipient], "Recipient address is flagged as bot");
        require(balanceOf(recipient) + amount <= _maxTokensPerWallet, "Recipient wallet exceeds max token limit");

        uint256 taxAmount = amount * _taxRate / 10_000;
        uint256 burnAmount = amount * _burnRate / 10_000;
        uint256 transferAmount = amount - taxAmount - burnAmount;

        _totalBurnt += burnAmount;
        _burn(msg.sender, burnAmount);
        _transfer(msg.sender, _owner, taxAmount);
        _transfer(msg.sender, recipient, transferAmount);

        return true;
    }

    function addBot(address botAddress) public {
        require(msg.sender == _owner, "Only contract owner can add bots");
        _bots[botAddress] = true;
    }

    function removeBot(address botAddress) public {
        require(msg.sender == _owner, "Only contract owner can remove bots");
        _bots[botAddress] = false;
    }

    function getMaxTokensPerWallet() public view returns (uint256) {
        return _maxTokensPerWallet;
    }

    function getTotalBurnt() public view returns (uint256) {
        return _totalBurnt;
            }
}
