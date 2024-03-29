---
description: The currentSeed getter can be used for access
---

# Accessing a Random Seed in a Smart Contract

With the new POSDAO implementation, smart contracts will have access to a random number generated by the protocol. 

The following is from the [POSDAO smart contracts repo](https://github.com/poanetwork/posdao-contracts#Smart-Contract-Summaries):

`currentSeed`. This public getter is used by the `ValidatorSetAuRa` contract at the latest block of each staking epoch to get the accumulated random seed for randomly choosing new validators among active pools. It can also be used by anyone who wants to use the network's random seed. Note, that its value is only updated when `revealSecret` function is called: that's expected to be occurred at least once per `collection round` which length in blocks can be retrieved with the `collectRoundLength` public getter. Since the revealing validator always knows the next random number before sending, your DApp should restrict any business logic actions \(that depends on random\) during `reveals phase`.

## Example code to retrieve a random seed

```javascript
pragma solidity 0.5.11;


interface IPOSDAORandom {
    function collectRoundLength() external view returns(uint256);
    function currentSeed() external view returns(uint256);
}

contract Example {
    IPOSDAORandom private _posdaoRandomContract; // address of RandomAuRa contract
    uint256 private _seed;
    uint256 private _seedLastBlock;
    uint256 private _updateInterval;

    constructor(IPOSDAORandom _randomContract) public {
        require(_randomContract != IPOSDAORandom(0));
        _posdaoRandomContract = _randomContract;
        _seed = _randomContract.currentSeed();
        _seedLastBlock = block.number;
        _updateInterval = _randomContract.collectRoundLength();
        require(_updateInterval != 0);
    }

    function useSeed() public {
        if (_wasSeedUpdated()) {
            // using updated _seed ...
        } else {
            // using _seed ...
        }
    }

    function _wasSeedUpdated() private returns(bool) {
        if (block.number - _seedLastBlock <= _updateInterval) {
            return false;
        }

        _updateInterval = _posdaoRandomContract.collectRoundLength();

        uint256 remoteSeed = _posdaoRandomContract.currentSeed();
        if (remoteSeed != _seed) {
            _seed = remoteSeed;
            _seedLastBlock = block.number;
            return true;
        }
        return false;
    }
}
```

