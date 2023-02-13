程序中的所有数在计算机内存中都是以二进制的形式储存的。位运算就是直接对整数在内存中的二进制位进行操作。

[位运算基础](https://www.w3school.com.cn/js/js_bitwise.asp)

基础部分要看超链接，我写的不细致，不好看懂。

零填充左位移： （js写作 '<<' ）

a << b 

a向左移b位（在后面添b个0）

在js中的写法

```
6 << 2 // 6的二进制110，右边添加两个0即：11000，所以 6 << 2 ： 34
```
二进制的右侧添加 2个0相当于乘以 2的2次方。  a << b 相当于 a * 2的b次方。

相应的

有符号右位移： （js写作 '>>' ）

a >> b

a向右移b位（去掉末b位）

在js中的写法

```
43 >> 2 // 43的二进制101011，向右移2位即：1010，所以 6 << 2 ： 34
```

二进制的去除末尾2位相当于除以 2的2次方，并且向下取整（因为不管末尾2位是11、01、10、00 都被舍弃了，都相当于00）。  a >> b 相当于 Math.floor(a / 2的b次方)。

and运算：（js写作 '&' ）

|运算	|结果	|
|--	|--	|
|0&1	|0	|
|1&0	|0	|
|1&1	|1	|
|0&0	|0	|

都是 1 则结果为 1

应用：用它检查某一位上的数值是否是1

常见应用：判断单双数



or运算:    (js写作 '|' )

|运算	|结果	|
|--	|--	|
|0|1	|1	|
|1|0	|1	|
|1|1	|1	|
|0|0	|0	|

至少其中1个为1 则结果为1



vue中的应用：

patchFlag是vue3做的优化标记，再patch阶段vue使用这个标记获取有哪些是动态的、需要更新。从而可以有目标的去做相应的patch。

```
  /**
   * dev only flag -> name mapping
   */
  const PatchFlagNames = {
    [1 /* TEXT */]: `TEXT`, // 									1 << 0
    [2 /* CLASS */]: `CLASS`, // 								1 << 1: 10
    [4 /* STYLE */]: `STYLE`, // 								1 << 2: 100
    [8 /* PROPS */]: `PROPS`, // 								1 << 3: 1000
    [16 /* FULL_PROPS */]: `FULL_PROPS`, // 					1 << 4: 10000
    [32 /* HYDRATE_EVENTS */]: `HYDRATE_EVENTS`, //				1 << 5: 100000
    [64 /* STABLE_FRAGMENT */]: `STABLE_FRAGMENT`, // 			1 << 6: 1000000
    [128 /* KEYED_FRAGMENT */]: `KEYED_FRAGMENT`, // 			1 << 7: 10000000
    [256 /* UNKEYED_FRAGMENT */]: `UNKEYED_FRAGMENT`, // 		1 << 8: 100000000
    [512 /* NEED_PATCH */]: `NEED_PATCH`, // 					1 << 9: 1000000000
    [1024 /* DYNAMIC_SLOTS */]: `DYNAMIC_SLOTS`, // 			1 << 10: 10000000000
    [2048 /* DEV_ROOT_FRAGMENT */]: `DEV_ROOT_FRAGMENT`, // 	1 << 11: 100000000000
    [-1 /* HOISTED */]: `HOISTED`,
    [-2 /* BAIL */]: `BAIL`
  };

// 伪代码
let patchFlag = 0
if (STABLE_FRAGMENT) {
	patchFlag |= 64 // patchFlag的二进制表示： 1000000
}
if (TEXT) {
	patchFlag |= 1 // patchFlag的二进制表示： 1000001
}
// 二进制的第7位和第1位是1。 代表了 TEXT 和 STABLE_FRAGMENT的存在。
// 然后可以用 patchFlag & 64 获取 STABLE_FRAGMENT 是否存在

// vue中
			const flagNames = Object.keys(PatchFlagNames)
              .map(Number)
              .filter(n => n > 0 && patchFlag & n)
              .map(n => PatchFlagNames[n])
              .join(`, `);
// 获取相应的 PatchFlagNames
```

可以看到。PatchFlagNames 的key除了两个复数， 他都是2的几次方。写成2进制之后它们都在某一位为1，其它位为0。

例如：64：1000000 第7位为1。 





通过它我们可以发现的and运算的普遍应用： 可以将2进制的某一位代表某个东西是否存在。1代表存在0代表不存在。然后用 and运算获取这一位是否为1。

or运算的普遍应用：配合and运算的应用，如果要记录某个东西存在，本来需要将某个东西加入数组，并且已经存在数组中的元素不应该重复添加。本来可以用 new set()对象存储。或者判断了是否存在之后再push进数组。

我们用 2进制的某一位代表某个东西是否存在后，可以用 or运算将这一位变成1。省去了检查这一个元素是否已经存在在数组中的过程。而对比用 new set 储存。or运算的效率也更高。

————————————————————————

效率测试: 

```
      const count = 31

      let p = 0
      const a = []
      for (let i = 0; i < count; i++) {
        a[i] = 1 << i
      }
      let st = new Date().getTime()
      for (let i = 0; i < 10000000; i++) {
        const r = Math.round(Math.random() * (count - 1))
        p |= a[r]
      }
      const rs = a.filter(n => n > 0 && p & n)
      let et = new Date().getTime()
      console.log('ms:', et - st)
      console.log(p, rs)
      
      // let st = new Date().getTime()
      // let rs = new Set()
      // for (let i = 0; i < 10000000; i++) {
      //   const r = Math.round(Math.random() * (count - 1))
      //   rs.add(r)
      // }
      // let et = new Date().getTime()
      // console.log(et - st)
      // console.log(rs)

      // let st = new Date().getTime()
      // let rs = []
      // for (let i = 0; i < 10000000; i++) {
      //   const r = Math.round(Math.random() * (count - 1))
      //   if (rs.indexOf(r) === -1) {
      //     rs.push(r)
      //   }
      // }
      // let et = new Date().getTime()
      // console.log(et - st)
      // console.log(rs)
```

注意：用2进制判断是否存在的东西数量不能超过31。 大概是因为 js所有按位运算都以 32 位二进制数执行。



在MD5加密中的应用。
MD5加密的运算量可能很大，所以这应该也是MD5里面全是位运算的原因之1。
```
function md5blk(s) {
        var md5blks = [],
            i; /* Andy King said do it this way. */

        for (i = 0; i < 64; i += 4) {
            md5blks[i >> 2] = s.charCodeAt(i) + (s.charCodeAt(i + 1) << 8) + (s.charCodeAt(i + 2) << 16) + (s.charCodeAt(i + 3) << 24);
        }
        return md5blks;
    }
```

s是个长度64的数组，返回一个长度16的数组。它将s中的每4项合并成了1项。保存在md5blks中。 for循环一次 i +=4。 用二进制看是 += 100。 i >> 2之后就变成  += 1了。所以for循环一次。 md5blks数组末尾加一项



用二分法排序

```
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var sortArray = function(nums) {
    tails = []
    for (let i =0; i < nums.length; i++) {
      if (nums[i] > tails[tails.length - 1]) {
        tails.push(nums[i]);
      } else {
        let left = 0
        let right = tails.length - 1
        while (left < right) {
            let mid = (left + right) >> 1 
            if (tails[mid] < nums[i]) {
                left = mid + 1;
            } else {
                right = mid
            }
        }
        tails.splice(left, 0, nums[i])
      }
    }
    return tails
};
```

表示一个东西是否存在

除以  2的n次方

判断单双数

