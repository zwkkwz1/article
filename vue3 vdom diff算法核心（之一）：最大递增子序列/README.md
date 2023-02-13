# 最大递增子序列

[Vue3 DOM Diff核心算法解析](https://blog.csdn.net/weixin_37352936/article/details/108873370)

[二分查找](https://blog.csdn.net/weixin_37352936/article/details/107255177)

[将分治、动态规划、回溯、贪心一锅炖](https://blog.csdn.net/weixin_37352936/article/details/106740874)

我是看这个链接学习的。但我讲的可能更加详细。可以看看。

### vue中为什么要用最大递增子序列：

```
   <div id='app'>
      <div v-for="item in list" :key='item'>
        {{ item }}
      </div>
      <div @click='listChange'>listChange</div>
    </div>
    <script type="text/javascript">
      const app = Vue.createApp({
        data() {
          return {
            count: 1,
            show: false,
            list: [1, 2, 3, 4, 5, 6, 7]
          }
        },
        methods: {
          listChange() {
            this.list = [7, 1, 3, 4, 5, 6, 2]
          }
        }
      }).mount('#app')
  </script>
```

点击listChange按钮之后

lis从 [1, 2, 3, 4, 5, 6, 7]变成[7, 1, 3, 4, 5, 6, 2]。 每一个位置都变了。而如何移动最少的节点达到更新目的。答案：算出最大递增子序列。 [1, 3, 4, 5, 6]。 这几个节点不变。移动7， 2节点就够了。

#### vue使用的算法：贪心 + 二分查找
贪心: 

引用上面连接中的解释：

这里再结合本题理解一下贪心思想，同样是长度为 2 的序列，[1,2] 一定比 [1,4] 好，因为它更有潜力。换句话说，我们想要组成最长的递增子序列，
就要让这个子序列中上升的尽可能的慢，这样才能更长。

我们可以创建一个 tails 数组，用来保存最长递增子序列，如果当前遍历的 nums[i] 大于 tails 的最后一个元素(也就是 tails 中的最大值)时，我们将其追加到后面即可。否则的话，我们就查找 tails 中第一个大于 nums[i] 的数并替换它。因为是单调递增的序列，我们可以使用二分查找，将时间复杂度降低到 O(logn) 。
————————————————
版权声明：本文为CSDN博主「童欧巴」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_37352936/article/details/108873370



关于二分查找。连接中也能找到解释。另外解释下：

连接中的二分查找在 目标（nums[i]） 大于 中间那个数（tails[mid]），计算区间取右边时，left = mid + 1

因为  let mid = (left + right) >> 1 （>> 是二进制右移1，相当于除以2的1次方再向下取整。）

因为是向下取整。所以mid的计算结果会偏向left。

所以最极限的情况：如果 目标（nums[i]）是小于 tails[mid + 1], 也必然能计算到。

而如果  left = right - 1。 mid =  (left + right) >> 1 = left。 如果这时，中间数 （tails[mid]）< 目标 (nums[i]),  如果 left = mid。 那这种情况 left没变。right没变。 while将无线循环。 所以必须 left = mid + 1。

而为什么先要将 nums[i]与tails数组最后一项比较一次。  因为   let mid = (left + right) >> 1 是 (left + right)除以2的1次方再向下取整。所以用反正法：如果要mid = right。 必须 let mid = (right + right) >> 1。 即 left = right。 这while的条件不允许的。 

而为什么 while的条件不能是 while(left <= right)  因为当 left = right， 如果 tails[mid]即tails[right] < nums[i] ， left = mid + 1 = right + 1 跳出循环，这没事。如果 tails[mid]即tails[right] > nums[i] right = mid = right。  又是一个无限循环。

**但是：我们可以尝试: 如果改成 mid =  (left + right + 1) >> 1 中间数右偏，代码会不会有另一种写法。**

贪心+二分算法 的 举个栗子：

|1	|4	|9	|3	|5	|8	|
|--	|--	|--	|--	|--	|--	|
|1	|	|	|	|	|	|
|1	|4	|	|	|	|	|
|1	|4	|9	|	|	|	|
|1	|3	|9	|	|	|	|
|1	|3	|5	|	|	|	|
|1	|3	|5	|8	|	|	|

1 3 9 的时候 （这里可能会有疑惑，这不是错了吗。但是就算数组在这里结束，这种错误不会让最大递增子序列长度出错，也不会让最后的一个节点出错）



可以用上面连接中的算法去尝试。看结果对不对。这调数据我没试过的

这组数据的结果没错。而如果数组是： 1, 4, 9, 3 得出的结果是 1, 3, 9 不就错了吗？ 

### vue给出了答案：

**原理**：因为贪心算法得出的子序列（result）的长度是正确的，result的末尾那一项也是正确的。
而如果在贪心算法在result里面每一次插入一个值（假设是 num）时，假如在序列的索引 i 处插入值，那么这个值必定大于 result[i-1],而且这个值在原数组中的位置也在
result[i-1]这个值后面。所以 result[i-1]，num 这个排列肯定是对的，而result末尾的那一项又是对的，所以可以记住每一个数前面正确的数值，
重result的最后一项反响推到出正确的结果

**如果贪心算法有一定的理解，对这个反推就不难理解了。**

Vue3用一个数组（const p = arr.slice();）记住每一个数前面正确的数值
```
       if (arr[j] < arrI) {
        p[i] = j;
        result.push(i);
        continue;
     }
		  // 以及

       if (arrI < arr[result[u]]) {
       if (u > 0) {
         p[i] = result[u - 1];
       }
       result[u] = i;
     }

		//   然后由于上面括号里讲过，上面的步骤不会让获取的递增子序列的最后一个值出错，所以可以

    u = result.length;
    v = result[u - 1];
    while (u-- > 0) {
      result[u] = v;
      v = p[v];
    }

	// 通过数组p，从后往前一步一步找回了正确的结果。
```

vue的算法获取的返回值是arr最大递增子序列的索引。
```
  function getSequence(arr) {
    const p = arr.slice();
    const result = [0];
    let i, j, u, v, c;
    const len = arr.length;
    for (i = 0; i < len; i++) {
      const arrI = arr[i];
      if (arrI !== 0) {
        j = result[result.length - 1];
        if (arr[j] < arrI) {
          p[i] = j;
          result.push(i);
          continue;
        }
        u = 0;
        v = result.length - 1;
        while (u < v) {
          c = ((u + v) / 2) | 0;
          if (arr[result[c]] < arrI) {
            u = c + 1;
          }
          else {
            v = c;
          }
        }
        if (arrI < arr[result[u]]) {
          if (u > 0) {
            p[i] = result[u - 1];
          }
          result[u] = i;
        }
      }
    }
    u = result.length;
    v = result[u - 1];
    while (u-- > 0) {
      result[u] = v;
      v = p[v];
    }
    return result;
  }
```



vue的算法可能还有不好看懂，因为他的结果是一个索引

我用vue的方法改变了下链接中的贪心+二分算法。

```
      function sortTwo(nums) {
        let len = nums.length;
        const p = {} // 拷贝一个数组 p
        if (len <= 1) {
            return len;
        }
        let tails = [nums[0]];
        for (let i = 0; i < len; i++) {
            // 当前遍历元素 nums[i] 大于 前一个递增子序列的 尾元素时，追加到后面即可
            if (nums[i] > tails[tails.length - 1]) {
                p[nums[i]] = tails[tails.length - 1]
                tails.push(nums[i]);
            } else {
                // 否则，查找递增子序列中第一个大于当前值的元素，用当前遍历元素 nums[i] 替换它
                // 递增序列，可以使用二分查找
                let left = 0;
                let right = tails.length - 1;
                while (left < right) {
                    let mid = (left + right) >> 1;
                    if (tails[mid] < nums[i]) {
                        left = mid + 1;
                    } else {
                        right = mid;
                    }
                }
                if (left - 1 >= 0) {
                  p[nums[i]] = tails[left - 1]
                }
                tails[left] = nums[i];
            }
        }
        let right = tails.length // tails: [1, 3, 9]  p: {3: 1, 4: 1, 9: 1}
        let v = tails[right - 1]
        while (right-- > 0) {
          tails[right] = v
          v = p[v]
        }
        return tails.length;  // [1, 4 , 9]
      }

      sortTwo([1, 4, 9, 3])
```

也许看这个更加直观


## 下面是新的二分查找法

上面提到的二分查找法理解起来需要花点时间。

并且，如果给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。（这是一道力扣题）

这两种位置的查找通常需要不一样的二分查找模板

但是，最近发现一个通用的二分查找模板，是在力扣看到的转载B站的 五点七边 UP主的一个算法

连接（力扣文章地址），五点七边（B站UP主主页）

	left  指针和 right 指针开始取在界外，left =首个元素下标-1，right=末尾元素下标+1；
	循环条件始终为 left +1 ≠ right
	mid取值为 (left + right) >> 1
	l 指针和 r 指针变化的时候直接变为 mid 指针，无需 +1 或者 -1 


上代码：这个代码是随心源的。
```
const binarySearchLeft = (nums, target) => {
    let left = -1, right = nums.length;
    while (left+1 != right) {
        const mid = Math.floor((left + right) / 2);
        if (nums[mid] >= target) { // 因为是查询目标值的开始位置。 所以相等时将查询区域向左移，right = mid
            right = mid;
        } else {
            left = mid;
        }
    }
    return right;
}

const binarySearchRight = (nums, target) => {
    let left = -1, right = nums.length;
    while (left+1 != right) {
        const mid = Math.floor((left + right) / 2);
        if (nums[mid] > target) {
            right = mid
        } else { // 因为是查询目标值的结束位置。 所以相等时将查询区域向右移，left = mid
            left = mid;
        }
    }
    return left;
}

var searchRange = function(nums, target) {
    let ans = [-1, -1];
    const leftIdx = binarySearchLeft(nums, target);
    const rightIdx = binarySearchRight(nums, target);
    if (leftIdx <= rightIdx && rightIdx < nums.length && nums[leftIdx] === target && nums[rightIdx] === target) {
        ans = [leftIdx, rightIdx];
    } 
    return ans;
};

```

binarySearchLeft 用于获取目标值在数组中的开始位置

binarySearchRight 用于获取目标值在数组中的结束位置



看 函数 binarySearchLeft，每次循环。left或者right将=mid，也就是right和left的距离在以接近每次除以2的速度缩短。必然能达到他们相差1的情况。也许有人会想，会不会left直接等于right而跳过rigth === left + 1。 

const mid = Math.floor((left + right) / 2);所以要让循环之后left === right的条件是：right = mid 且 right === left + 1。但是这时循环条件已经不符合。

left和right的距离以每次缩小接近二分之一的速度变化。必然能达到right === left + 1这个结束循环的条件，而达到这个条件的上一次循环， 必然是 right === left + 2 或者 right === left + 3。

所以在不断缩小查询区域的过程中，left和right的差距必然被缩小到只有2或者3，而目标值（target）必然在这个区域内（除非数组内没有target）。所以我们只要讨论 right === left + 2 和 right === left + 3两种情况。

先证明数组left位置的值必定不是target:

假设left != -1 即 left变化过。所以 执行过left = mid。 此时  target < nums[mid],

所以数组在left位置上的值必然不是target。(结论1)

而如果left === -1。必然也符合上面一行的结果。

将target分为四种情况。

1. target > nums[nums.length - 1] （情景1）

right = mid 将一直不会执行，所以right === nums.length

2.target < nums[0]（情景2）

left = mid 一直不会执行，所以 left === -1

3. target >= nums[0] 且 target <= nums[nums.length - 1]。

a.假设数组内存在target。

right === left + 3

 \- ... - - nums[left] nums[left + 1] nums[left + 2] nums[right]	 - - ... -

由结论1可知，target必然在left右侧，

不论right在不在数组上。这时target只有可能在数组的 left + 1 位置 left + 2 或者right位置。

下一次循环。mid = left + 1 如果 nums[left + 1] === target。 right = mid将执行。 循环结束。right得到目标值在数组中的开始位置。

如果target 大于 nums[left + 1] 。 left  = mid执行。 即left指针右移一格。

这时变成了 right === left + 2 的情况。

right === left + 2
 
 \- ... - - nums[left] nums[left + 1] nums[right]	 - - ... -
这时目标只可能在right位置和left + 1位置

下一次循环。 mid = left + 1。如果 nums[left + 1] === target。 right = mid将执行。 循环结束。right得到目标值在数组中的开始位置。

如果nums[left + 1] !== target. 即taget在right位置。left = mid执行。 循环结束。 right的值还是正确的。

假设数组内不存在target。循环结束时不论right的值是什么  nums[right] !== target（情景3）

情景1、 2、 3 在最后兜底检查：

```
if (leftIdx <= rightIdx && rightIdx < nums.length && nums[leftIdx] === target && nums[rightIdx] === target) {
    ans = [leftIdx, rightIdx];
} 
```

中不会通过


binarySearchRight的方法应该也可以同样证明。