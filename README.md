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
  
  #### - updatestakingpool function

    function updateStakingPool() internal {
    	uint256 length = poolInfo.length;
            uint256 points = 0;
            for (uint256 pid = 1; pid < length; ++pid) {
                points = points.add(poolInfo[pid].allocPoint);
            }
            if (points != 0) {
                points = points.div(3);
                totalAllocPoint = totalAllocPoint.sub(poolInfo[0].allocPoint).add(points);
                poolInfo[0].allocPoint = points;
            }
      }
    

   call this function at the end of adding LP token pool to update the alloc for cake and totalalloc point

   
    	uint256 length = poolInfo.length;
   

   length ≥2 when it's called because we just pushed the LP to the [pool.info](http://pool.info) array

  
    for (uint256 pid = 1; pid < length; ++pid) {
                points = points.add(poolInfo[pid].allocPoint);
            }

   points = all pools' allocpoints combined except for **cake** **pool** (alloc point was set by the pool adder) in add func's parameter

    
    if (points != 0) {
                points = points.div(3);
                totalAllocPoint = totalAllocPoint.sub(poolInfo[0].allocPoint).add(points);
                poolInfo[0].allocPoint = points;
            }
    ```

   points is divided by 3 and then set to the pool[0] allocPoint in order to set the **cake pool's allocPoint to** **always be 25% relatively to other pools' alloc**

   for example before add new LP, totalalloc = cake.alloc = 100

   after add new LP with 100 alloc, totalalloc = 100+100=200, points = 100 , new points(divided by 3) = 33, new totalalloc = totalalloc - cake.alloc + newpoints = 200-100+33=133, cake.allocpoint = new points (divided by 3) = 33 

   So cake's alloc/totalalloc = 33/133 = 25%
   
  #### - pendingcake

    
    function pendingCake(uint256 _pid, address _user) external view returns (uint256) {
            PoolInfo storage pool = poolInfo[_pid];
            UserInfo storage user = userInfo[_pid][_user];
            uint256 accCakePerShare = pool.accCakePerShare;
            uint256 lpSupply = pool.lpToken.balanceOf(address(this));
            if (block.number > pool.lastRewardBlock && lpSupply != 0) //current block must be > lastrewardblock to get the positive multiplier {
                uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number); //10x 20x , so on on the website. 100=1x
                uint256 cakeReward = multiplier.mul(cakePerBlock).mul(pool.allocPoint).div(totalAllocPoint);
    						//cakereward is the total cake that that pool will get from the total cake minted for all pools
                accCakePerShare = accCakePerShare.add(cakeReward.mul(1e12).div(lpSupply));
    						//accCakePerShare is the multiplier of how many cake user will get calculated proportionately to the LPsupply for example, user has 1 LP, they will get 1* accCakePerShare cake tokens
            }
            return user.amount.mul(accCakePerShare).div(1e12).sub(user.rewardDebt)
    								//if 1st stake 100, 2nd stake 100> system will transfer cake due to the 1st 100 stake. then user will have 200 of user.amount so his pending will be user.amount-rewarddebt = 200-100 =100
    								//rewarddebt = amount that has been transfered to user before to reset their reward when they enter the new staking
    }
   - 
   
    PoolInfo storage pool = poolInfo[_pid];
    UserInfo storage user = userInfo[_pid][_user];
    
   to get the pool info and user info specific to that _pid

    
    uint256 accCakePerShare = pool.accCakePerShare;

   to get the pool.accCakePerShare (not the lastest one according to the block.number that has changed.


    uint256 lpSupply = pool.lpToken.balanceOf(address(this));

   get the LP tokens balances in Masterchef contract


    if (block.number > pool.lastRewardBlock && lpSupply != 0) //current block must be > lastrewardblock to get the positive multiplier 

   to check if block.number > pool.lastRewardBlock because we need getMultiplier to be more than 1 or knows that the block already runs from the lastRewardBlock

    
    uint256 cakeReward = multiplier.mul(cakePerBlock).mul(pool.allocPoint).div(totalAllocPoint);

   get cake reward for that pool proportionately


    accCakePerShare = accCakePerShare.add(cakeReward.mul(1e12).div(lpSupply));

   update the accCakePerShare according to the change of cakeReward/lpSupply (newly added accCakePerShare)
   
## Deployed Contracts / Hash

### BKC testnet

- CakeToken 0xC80412c23fF0B5dD9f61c054f72280C16570AcA7
- MasterChef - 0x083A7a096087b0A75473D1Fe2ba9f91250f09eA0
- SyrupBar - 0x77DB5E829d71Eab4C82E4Ad4D32FB97142F7C581

