// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
contract ShibaScroll is IERC20, Ownable {
    using SafeMath for uint256;

    string private _name = "Shiba Scroll";
    string private _symbol = "SHIBS";
    uint8 private _decimals = 18;
    uint256 private _totalSupply = 1000000000 * 10**uint256(_decimals);
    
    uint256 private _marketingTax = 3; // 3% marketing tax
    address private _marketingWallet = 0x42D2A00FdC659f51Bdf5446DF75B39943e7Ea140;
    bool private _taxEnabled = true;
    
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;

    constructor() {
        _marketingWallet = msg.sender; // Set the marketing wallet to the deployer's address
        _balances[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    // Standard ERC-20 functions

    function name() public view returns (string memory) {
        return _name;
    }

    function symbol() public view returns (string memory) {
        return _symbol;
    }

    function decimals() public view returns (uint8) {
        return _decimals;
    }

    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }

    function balanceOf(address account) public view override returns (uint256) {
        return _balances[account];
    }

    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(msg.sender, recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        _transfer(sender, recipient, amount);
        _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount));
        return true;
    }

    // Custom functions

    function enableTax() public onlyOwner {
        _taxEnabled = true;
    }

    function disableTax() public onlyOwner {
        _taxEnabled = false;
    }

    function excludeFromFee(address account) public onlyOwner {
        // Implement the logic for excluding an address from fees
    }

    function includeInFee(address account) public onlyOwner {
        // Implement the logic for including an address in fees
    }

    function marketingTax() public view returns (uint256) {
        return _marketingTax;
    }

    function setMarketingWallet(address newWallet) public onlyOwner {
        _marketingWallet = newWallet;
    }

    // Internal functions

    function _transfer(address sender, address recipient, uint256 amount) internal {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");

        uint256 marketingFee = _calculateMarketingFee(amount);
        uint256 transferAmount = amount.sub(marketingFee);

        _balances[sender] = _balances[sender].sub(amount);
        _balances[recipient] = _balances[recipient].add(transferAmount);

        if (marketingFee > 0) {
            _balances[_marketingWallet] = _balances[_marketingWallet].add(marketingFee);
            emit Transfer(sender, _marketingWallet, marketingFee);
        }

        emit Transfer(sender, recipient, transferAmount);
    }

    function _calculateMarketingFee(uint256 amount) internal view returns (uint256) {
        if (_taxEnabled) {
            return amount.mul(_marketingTax).div(100);
        } else {
            return 0;
        }
    }

    function _approve(address owner, address spender, uint256 amount) internal {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");

        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
}
