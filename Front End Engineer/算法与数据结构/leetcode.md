## 1. 两数之和

**给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。** 

> 本人的思路是根据目标值和数组各项的差值，然后循环判断数组哪项和差值相等，再返回数组下标，这样就导致了可能目标值和数组某项差值等于该项自身值等问题。

> 大神思路
>
> - 初始化一个 `map = new Map()`
> - 从第一个元素开始遍历 `nums`
> - 获取目标值与 `nums[i]` 的差值，即 `k = target - nums[i]` ，判断差值在 `map` 中是否存在
>   1. 不存在（ `map.has(k)` 为 `false` ） ，则将 `nums[i]` 加入到 `map` 中（key为`nums[i]`, value为 `i` ，方便查找map中是否存在某值，并可以通过 `get` 方法直接拿到下标）
>   2. 存在（ `map.has(k)` ），返回 `[map.get(k), i]` ，求解结束
> - 遍历结束，则 `nums` 中没有符合条件的两个数，返回 `[]`
>
> ```javascript
> var twoSum = function(nums, target) {
>     let map = new Map()
>     for(let i = 0; i< nums.length; i++) {
>         let k = target-nums[i]
>         if(map.has(k)) {
>             return [map.get(k), i]
>         }
>         map.set(nums[i], i)
>     }
>     return [];
> };  // 时间复杂度O(n)
> ```

> 其他解法
>
> 1. 暴力法
>
>    - 使用两层循环，外层循环计算 当前元素与target之间的差值，内层循环寻找该差值，若找到该差值，则返回两个元素的下标。
>
>    - 时间复杂度： O(n^2)
>
>      ```javascript
>      /**
>      * @param {number[]} nums
>      * @param {number} target
>      * @return {number[]}
>      */
>      var twoSum = function(nums, target) {
>        for(var i = 0; i<nums.length; i++) {
>          var dif = target - nums[i];
>          // j = i+1 的目的是减少重复计算和避免两个元素下标相同
>          for(var j = i+1;j<nums.length;j++) {
>            if(nums[j] == dif)
>              return [i,j]
>          }
>        }
>      }
>      ```

> 2. 利用数组减少查询时间
>
>    - 使用一层循环，每遍历到一个元素就计算该元素与 targettarget 之间的差值 difdif，然后以 difdif 为下标到数组temp中寻找，如果 temp[dif] 有值(即不是 undefinedundefined)，则返回两个元素在数组 numsnums 的下标，如果没有找到，则将当前元素存入数组 temptemp 中(下标: nums[i], Value: inums[i],Value:i) 。
>
>    - 时间复杂度：O(n)
>
>      ```javascript
>      /**
>      * @param {number[]} nums
>      * @param {number} target
>      * @return {number[]}
>      */
>      var twoSum = function(nums, target) {
>        var temp = []
>        for(var i = 0;i<nums.length;i++) {
>          var dir = target - nums[i];
>          if(temp[dif] != undefined){
>            return [temp[dir], i];
>          }
>          temp[nums[i]] = i;
>        }
>      }
>      ```
>
>      

## 17.电话号码的字母组合

**给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。** 

> 例如输入：‘23’
>
> 输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].



> 思路：
>
> - 先把电话号码和字母映射建立起来
> - 将输入的数字字符串进行分隔成数组
> - 然后把第二步的数组映射成字母数组，然后循环第一项和第二项，并用一个数组记录保存下来
> - 以防字母数组长度大于2，就利用splice把结果数组前两项用结果数组替代，并和后面的继续递归操作

```javascript
/**
 * @param {string} digits
 * @return {string[]}
 */
var letterCombinations = function(digits) {
    let wordArray = ['',1,'abc','def','ghi','jkl','mno','pqrs','tuv','wxyz']//显而易见的可以看出数组序列为2-9时分别对应什么字母字符串
    let digitsArray = digits.split('')//把输入的数字字符串转换为一个一个数组，如'23'-> ['2','3']
    let codeArray = []//字母字符串数组
    digitsArray.forEach(item =>{
        //判断输入的字符串有没有1
        if(wordArray[item]) {
            codeArray.push(wordArray[item])
        }
    })
    // 还需要加一个判断, 如果只输入一个数字, 如 "2" 的时候, 只进行一次循环.
  if(codeArray.length === 1) {
    var temp = []
    for(let i = 0; i < code[0].length; i ++) {
      temp.push(codeArray[0][i])
    }
    return temp
  }
    let comb = arr => {
        let resultArray = []
        for(let i = 0,lh=codeArray[0].length;i<lh;i++) {
            for(let j = 0, lh=codeArray[1].length;j<lh;j++) {
                // let result = codeArray[0][i]+codeArray[1][j]
                // resultArray.push(result)
                //等换于
                resultArray.push(`${arr[0][i]}${arr[1][j]}`)
            }
        }
        arr.splice(0,2,resultArray)//把映射完成的数组从第一项开始替换前两项为resultArray
        if(arr.length > 1) {
            comb(arr)
        } else {
            console.log(resultArray)
            return resultArray
        }
        return arr[0]
    }
    return comb(codeArray)
};
letterCombinations("789")
```

## 914.卡牌组合

> 给定一副牌，每张牌上都写着一个整数。
>
> 此时，你需要选定一个数字 X，使我们可以将整副牌按下述规则分成 1 组或更多组：
>
> - 每组都有 X 张牌。
> - 组内所有的牌上都写着相同的整数。
>
> 仅当你可选的X>=2时返回true

> 思路：
>
> 1. 给定一个都是整数项的数组，先排序然后看看各数字出现的次数
> 2. 再判断次数的最大公约数是多少
> 3. 如果最大公约数是2的倍数，那么就返回true
>
> 最大公约数求法利用到了`辗转相除法` ，即欧几里得算法
>
> 例如求1997和615两个正整数的最大公约数，则：1997/615=3（余152）；615/152=4（余7）；152/7=21（余5）7/5=1（余2）；5/2=2（余1）；2/1=2（余0）
>
> 用代码实现即：
>
> ```javascript
> function gcd(a,b) {
>   if(a % b === 0) return b;
>   return gcd(b,a%b);
> }
> ```
>
> 代码详情：
>
> ```javascript
> /**
>  * @param {number[]} deck
>  * @return {boolean}
>  */
> var hasGroupsSizeX = function(deck) {
>     //最大公约数计算公式
>     function gcd(a,b) {
>         if(a%b === 0) return b;
>         return gcd(b,a%b)
>     }
>     //相同的牌出现的次数Map
>     let timeMap = new Map()
>     deck.forEach(item => {
>         timeMap.set(item,timeMap.has(item) ? timeMap.get(item) + 1 : 1);
>     });
>     let timeArr = [...timeMap.values()];
> 
>     //默认数组首位对公约数计算无干扰
>     let g = timeArr[0]
> 
>     //遍历出现次数，计算最大公约数
>     timeArr.forEach(time => {
>         g = gcd(g, time);
>     })
>     return g >=2;
> };
> ```

> 相同牌出现的次数可以利用reduce方法来实现
>
> ```javascript
> let timeMap = deck.reduce(function(allDeck, num){
>         if(num in allDeck) {
>             allDeck[num]++;
>         }
>         else {
>             allDeck[num] = 1;
>         }
>         return allDeck;
>  },{})
>  let timeArr = []
>  for(let i in timeMap) {
>    timeArr.push(timeMap[i])
>  }
> ```
>
> 

## 605.种花问题

> 假设你有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花卉不能种植在相邻的地块上，它们会争夺水源，两者都会死去。
>
> 给定一个花坛（表示为一个数组包含0和1，其中0表示没种植花，1表示种植了花），和一个数 n 。能否在不打破种植规则的情况下种入 n 朵花？能则返回True，不能则返回False。
>
> 实例：flowerbed = [1,0,0,0,1], n=1 输出：true
>
> **这个问题注重边界情况** 
>
> ```javascript
> //前后补个0把边界问题解决了
> var canPlaceFlowers = function(flowerbed, n) {
>     let result = 0, count = 0;
>     flowerbed.push(0);
>     flowerbed.unshift(0);
>     for (let i = 1, len = flowerbed.length; i < len; i++) {
>         if (flowerbed[i-1] === 0 && flowerbed[i] === 0 && flowerbed[i+1] === 0) {
>             flowerbed[i] = 1;
>             result++;
>         }
>     }
>     return result >= n ? true : false;
> };
> ```

> 也可以使用以1为分隔为突破口
>
> ```javascript
> var canPlaceFlowers = function(flowerbed, n) {
>   let temporaryArr = flowerbed.join('').split('1')
>   let resnum = 0
>   if (temporaryArr.length === 1) {
>     resnum = (temporaryArr[0].length % 2 === 0 ? temporaryArr[0].length / 2 : (temporaryArr[0].length + 1) / 2)
>     return resnum >= n
>   }
>   for (let i = 0; i < temporaryArr.length; i++) {
>     if (temporaryArr[i].length) {
>       if (i === 0 || i === temporaryArr.length - 1) {
>         resnum = resnum + (temporaryArr[i].length % 2 === 0 ? temporaryArr[i].length / 2 : (temporaryArr[i].length - 1) / 2)
>       } else {
>         resnum = resnum + (temporaryArr[i].length % 2 === 0 ? (temporaryArr[i].length - 2) / 2 : (temporaryArr[i].length - 1) / 2)
>       }
>     }
>   }
>   return resnum >= n
> };
> ```