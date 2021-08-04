# This is the forked project of PanCake Farming 🍴

## modification

  take the syrup token (cake's syrup token that represent the cake holding when users are staking) minting out of the enterStaking and leaveStaking function because the syrup token was deprecated due to the announcement that it has a vulnerability that users can manage to mint unlimited syrup sullply somehow.
  
  add the explaination and calculation formula in order to make it easier to fork and modify as our wish.
  
## code and function explaination

  #### add function

    function add(uint256 _allocPoint, IBEP20 _lpToken, bool _withUpdate) public onlyOwner {
            if (_withUpdate) {
                massUpdatePools();
            }
            uint256 lastRewardBlock = block.number > startBlock ? block.number : startBlock; //to change the lastRewardBlock of other LP to = startblock
            totalAllocPoint = totalAllocPoint.add(_allocPoint); 
            poolInfo.push(PoolInfo({
                lpToken: _lpToken,
                allocPoint: _allocPoint,
                lastRewardBlock: lastRewardBlock,
                accCakePerShare: 0
            }));
            updateStakingPool();
        }
    
    
    _allocpoint will assign the allocationpoint of that LP token

    _lpToken is the address of pancake pair

    massUpdatePools will call the updatepool() of every pools including cake pool

    lastRewardBlock = block number that the reward has been calculated.

    lastRewardBlock of _lpToken will be changed to = startBlock in order start getting cake reward at the startBlock too.

    totalAllocPoint += _allocPoint 

    poolInfo array will be updated with the 

    {           lpToken: _lpToken,
                allocPoint: _allocPoint,
                lastRewardBlock: lastRewardBlock,
                accCakePerShare: 0}

## Deployed Contracts / Hash

### BKC testnet

- CakeToken 0xC80412c23fF0B5dD9f61c054f72280C16570AcA7
- MasterChef - 0x083A7a096087b0A75473D1Fe2ba9f91250f09eA0
- SyrupBar - 0x77DB5E829d71Eab4C82E4Ad4D32FB97142F7C581

