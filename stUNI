// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/Pausable.sol";

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "./UniStaker.sol";
import "./Uni.sol";

contract stUni is Ownable, ERC20, Pausable {
    UniStaker private _unistaker;
    IERC20 private _uni;
    IERC20 private _reward;
    address defaultDelegatee;

    mapping(address account => uint256 amount) public balances;
    mapping(address account => uint256 depositId) private ids;

    event Staked(address indexed depositor, address indexed delegatee, uint256 amount);
    event Withdrawn(address indexed depositor, uint256 amount);
    event BeneficiaryAltered(address indexed depositor, address newBeneficiary);
    event DelegateeAltered(address indexed depositor, address newDelegatee);

    constructor(address unistaker, address uni, address reward)
        Ownable (msg.sender)
        ERC20("Liquid Staked UNI", "stUNI") {
        _unistaker = UniStaker(unistaker);
        _uni = IERC20(uni);
        _reward = IERC20(reward);
        defaultDelegatee = address(0);
    }

    function decimals() public pure override returns (uint8) {
        return 18;
    }

    function setIsPaused(bool isPaused) external onlyOwner {
        if (isPaused) _pause();
        else _unpause();
    }

    function setDefaultDelegatee(address delegatee) external onlyOwner {
        defaultDelegatee = delegatee;
    }

    function stake(uint256 amount, address delegatee) external whenNotPaused {
        if(delegatee == address(0)){
            delegatee = defaultDelegatee;
        }
        SafeERC20.safeTransferFrom(_uni, msg.sender, address(this), amount);
        _uni.approve(address(_unistaker), amount);

        uint256 id = ids[msg.sender];
        if(id > 0) {
            _unistaker.stakeMore(UniStaker.DepositIdentifier.wrap(id - 1), amount);
        } else {
            UniStaker.DepositIdentifier depositId = _unistaker.stake(amount, delegatee, msg.sender);
            ids[msg.sender] = UniStaker.DepositIdentifier.unwrap(depositId) + 1;
        }

        balances[msg.sender] += amount;
        emit Staked(msg.sender, delegatee, amount);

        _mint(msg.sender, amount);
    }

    function withdraw(uint256 amount) external whenNotPaused {
        uint256 id = ids[msg.sender];
        require(id > 0, "stake for this account does not exist");
        uint256 currentBalance = balances[msg.sender];

        require(currentBalance >= amount, "amount to withdraw is more than balance");
        uint256 balanceStUni = balanceOf(msg.sender);
        require(balanceStUni >= amount, "Not enough stUni");

        _unistaker.withdraw(UniStaker.DepositIdentifier.wrap(id - 1), amount);

        balances[msg.sender] -= amount;
        SafeERC20.safeTransfer(_uni, msg.sender, amount);

        emit Withdrawn(msg.sender, amount);

        _burn(msg.sender, amount);
    }

    function alterBeneficiary(address newBeneficiary) external {
        uint256 id = ids[msg.sender];
        require(id > 0, "stake for this account does not exist");
        _unistaker.alterBeneficiary(UniStaker.DepositIdentifier.wrap(id - 1), newBeneficiary);
        emit BeneficiaryAltered(msg.sender, newBeneficiary);
    }

    function alterDelegatee(address newDelegatee) external {
        uint256 id = ids[msg.sender];
        require(id > 0, "stake for this account does not exist");
        _unistaker.alterDelegatee(UniStaker.DepositIdentifier.wrap(id - 1), newDelegatee);
        emit DelegateeAltered(msg.sender, newDelegatee);
    }
}
