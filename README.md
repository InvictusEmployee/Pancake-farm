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
    
    
  - _allocpoint will assign the allocationpoint of that LP token

  - _lpToken is the address of pancake pair

  - massUpdatePools will call the updatepool() of every pools including cake pool

  - lastRewardBlock = block number that the reward has been calculated.

  - lastRewardBlock of _lpToken will be changed to = startBlock in order start getting cake reward at the startBlock too.

  - totalAllocPoint += _allocPoint 

  - poolInfo array will be updated with the 

  {           lpToken: _lpToken,
              allocPoint: _allocPoint,
              lastRewardBlock: lastRewardBlock,
              accCakePerShare: 0}

  #### updatePool function
  
    function updatePool(uint256 _pid) public {
            PoolInfo storage pool = poolInfo[_pid];
            if (block.number <= pool.lastRewardBlock) {
                return; //when the current block isn't higher than lastrewardblock that was set in the constructor, do nothing
            }
            uint256 lpSupply = pool.lpToken.balanceOf(address(this));
            if (lpSupply == 0) {
                pool.lastRewardBlock = block.number; 
                return; //when not adding lp here it will set pool.lastRewardBlock = block.number; 
            }
            uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number);
            uint256 cakeReward = multiplier.mul(cakePerBlock).mul(pool.allocPoint).div(totalAllocPoint);
            cake.mint(devaddr, cakeReward.div(10));// if comment out, Dev's cake balance after alice's withdraw will be 0.
            cake.mint(address(syrup), cakeReward); // if comment out, Alice's cake balance after alice's withdraw will be 0.
            pool.accCakePerShare = pool.accCakePerShare.add(cakeReward.mul(1e12).div(lpSupply)); 
            //when update pool accCakePerShare will += cakeReward.mul(1e12).div(lpSupply) on and on.
            pool.lastRewardBlock = block.number; // update the block number.
        }
  

   uint256 _pid is number of pool which is 0(cake pool), 1, 2, 3, ... for other LP token pools

    
    if (block.number <= pool.lastRewardBlock) {
                return 
   

   when the current block isn't more than lastrewardblock that was set in the constructor, do nothing

    
    uint256 lpSupply = pool.lpToken.balanceOf(address(this));
    
   get the supply of LPtoken in masterchef address

    
    if (lpSupply == 0) {
                pool.lastRewardBlock = block.number; 
                return; //when not adding lp here it will set pool.lastRewardBlock = block.number; 
            }
    
   when add the LP pool first time by admin, lpSupply will be 0 which will only change the pool.lastRewardBlock (the latest block that reward was calculated) to be the current block number

    
    uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number);
    
   getMultiplier(pool.lastRewardBlock, block.number) will get the amount of bonus*(block between block.number and pool.lastRewardBlock) ; bonus = 1 (admin set it)

    
     uint256 cakeReward = multiplier.mul(cakePerBlock).mul(pool.allocPoint).div(totalAllocPoint);

   cakeReward is the amount of multiplier(block distance*bonus) times cakeperblock times the proportion of that pool allocation point comparing to the total  allocation point of all pools

    
    cake.mint(devaddr, cakeReward.div(10))
    cake.mint(address(syrup), cakeReward);
    
   mint cake using cake contract to developer address with the cakeReward.div(10) CAKE

   mint cake to syrup address in order to distribute to the users later.

    
    pool.accCakePerShare = pool.accCakePerShare.add(cakeReward.mul(1e12).div(lpSupply)); 
    //when update pool accCakePerShare will += cakeReward.mul(1e12).div(lpSupply) on and on.
    pool.lastRewardBlock = block.number; // update the block number.
        }

   accCakePerShare = amount of cake user will get per one share LP token. Multiply by 10^12 in order to avoid the decimal in Solidity

   lastly, update the blocknumber
  
## Deployed Contracts / Hash

### BKC testnet

- CakeToken 0xC80412c23fF0B5dD9f61c054f72280C16570AcA7
- MasterChef - 0x083A7a096087b0A75473D1Fe2ba9f91250f09eA0
- SyrupBar - 0x77DB5E829d71Eab4C82E4Ad4D32FB97142F7C581

