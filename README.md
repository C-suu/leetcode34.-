# leetcode34.-

---

<img width="1146" height="776" alt="image" src="https://github.com/user-attachments/assets/72d8b9bb-f413-44fe-a09c-0ed610757151" />

---

<img width="1125" height="870" alt="image" src="https://github.com/user-attachments/assets/fa2bd738-c801-49c1-9613-62826f62a6a5" />

---



### （1）用白话文解释题目意思及核心概念

**题目白话文解释：**
有一个按从小到大排好序的数组，里面可能会有连续重复的数字。现在给你一个目标数字（`target`），请你找出这个数字在数组里第一次出现的位置（起始下标）和最后一次出现的位置（结束下标）。
如果数组里根本没有这个数字，就返回 `[-1, -1]`。
而且，题目硬性要求必须用 $O(\log n)$ 的时间复杂度，这等于明示了必须用**二分查找**。

**核心概念解释：**

  * **`lower_bound` (下界)**：这是一个非常经典的二分查找变体。它的目标不是“找到等于 target 的数”，而是\*\*“找到第一个大于或等于 target 的数的下标”\*\*。如果数组里所有的数都比 target 小，它就会越界，返回数组的长度 `len(nums)`。
  * **闭区间二分**：代码中使用的 `[left, right]` 是双闭区间。这意味着 `left` 和 `right` 指向的元素都在当前的搜索候选范围内。当 `left > right` 时，区间为空，循环结束。

-----

### （2）解题思路是怎么来的（基于代码反推）

这段代码的精妙之处在于：**只写一个 `lower_bound` 函数，却巧妙地解决了寻找起点和终点两个问题。**

1.  **找起点 (`start`)**：
    既然目标是找第一次出现的位置，这恰好完全契合 `lower_bound` 的定义：寻找第一个 `>= target` 的位置。

      * 找到后，必须验证一下：如果越界了，或者找到的那个数根本不是 `target`（比如找 8 却找到了 9），说明数组里没这个数，直接返回 `[-1, -1]`。

2.  **找终点 (`end`)**：
    常规思路是再写一个 `upper_bound` 函数来找最后一个 `<= target` 的位置，但这很容易把边界条件写错。
    作者用了一个极为聪明的数学转化：**找 `target` 的终点，等价于找 `target + 1` 的起点，然后往前退一格。**

      * 比如数组是 `[5, 7, 7, 8, 8, 10]`，找 `8` 的最后位置。
      * 我们去调用 `lower_bound(nums, 8 + 1)`，也就是找第一个 `>= 9` 的数。它会找到数字 `10`（下标为 5）。
      * 把这个下标减 1（`5 - 1 = 4`），下标 4 刚好就是最后一个 `8` 的位置！这样就完美复用了同一个二分函数。

-----

### （3）代码框内的详细注释

```python
class Solution:
    # 自定义核心二分函数：寻找第一个 >= target 的数字下标
    def lower_bound(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1    # 设定初始双闭区间搜索范围 [0, n-1]
        
        while left <= right:              # 只要区间内还有数字，就继续二分
            mid = (left + right) // 2     # 取中间位置
            
            if nums[mid] >= target:
                # 中间值已经 >= target，说明第一个 >= target 的数可能就是 mid，或者在 mid 的左边
                right = mid - 1           # 因此抛弃右半边，缩小范围到 [left, mid-1]
            else:
                # 中间值 < target，说明第一个 >= target 的数绝对在 mid 的右边
                left = mid + 1            # 因此抛弃左半边，缩小范围到 [mid+1, right]
                
        # 循环结束时，left 必然指向第一个 >= target 的位置
        return left

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        # 1. 寻找起点
        start = self.lower_bound(nums, target)
        
        # 2. 校验起点是否有效（越界，或者找到了一个比 target 大的数）
        if start == len(nums) or nums[start] != target:
            return [-1, -1] 
            
        # 3. 寻找终点：复用函数，找 target + 1 的起点，然后左移一位
        end = self.lower_bound(nums, target + 1) - 1
        return [start, end]
```

-----

### （4）每一行代码的详细中文解释

  * `def lower_bound(self, nums: List[int], target: int) -> int:`
      * 定义辅助函数，参数为数组和目标值。它负责返回第一个 $\ge \text{target}$ 的索引。
  * `left, right = 0, len(nums) - 1`
      * 初始化二分查找的左右边界。`left` 指向数组最开头，`right` 指向数组最末尾。
  * `while left <= right:`
      * 开启循环。因为是闭区间，当 `left == right` 时，区间里还有一个元素需要检查，所以必须带 `=` 号。
  * `mid = (left + right) // 2`
      * 计算左右边界的中间位置（向下取整）。
  * `if nums[mid] >= target:`
      * 核心判断逻辑：如果中间位置的数字大于或等于目标值。
  * `right = mid - 1`
      * 既然我们要找的是**第一个**大于等于目标值的位置，当前虽然满足条件，但它的左边可能还有满足条件的数。因此将右边界收缩到 `mid - 1`，继续向左逼近。
  * `else:`
      * 如果中间位置的数字严格小于目标值。
  * `left = mid + 1`
      * 说明满足条件的值必定在当前 `mid` 的右侧，因此将左边界收缩到 `mid + 1`，向右推进。
  * `return left`
      * 根据循环不变量，当循环结束（`left > right`）时，`left` 左边的数全部 $< \text{target}$，`right` 右边的数全部 $\ge \text{target}$。此时的 `left` 正好跨过了那些比目标值小的数，停在第一个 $\ge \text{target}$ 的位置上。
  * `def searchRange(self, nums: List[int], target: int) -> List[int]:`
      * 定义主函数。
  * `start = self.lower_bound(nums, target)`
      * 调用上面的二分逻辑，寻找目标值的起步下标。
  * `if start == len(nums) or nums[start] != target:`
      * 这是一个安全校验。如果 `start == len(nums)` 说明全数组的数都比 `target` 小；如果 `nums[start] != target` 说明找到了一个稍大一点的数（比如找 4 找到了 5）。这两种情况都说明 `target` 不存在。
  * `return [-1, -1]`
      * 查无此数，按题目要求返回 `[-1, -1]`。
  * `end = self.lower_bound(nums, target + 1) - 1`
      * 运用神级技巧寻找结束下标：找比目标值大 `1` 的那个数的起始位置，找到后，把它前面的那个位置（减去 1）赋给 `end`。
  * `return [start, end]`
      * 返回最终算出的起止范围列表。

-----

### （5）具体数值算例及步骤表格展示

**算例设定：**
输入：`nums = [5, 7, 7, 8, 8, 10]`, `target = 8`

**第一步：找起点 `lower_bound(nums, 8)`**

| 步骤 | 左边界 `L` | 右边界 `R` | 中间值 `mid` 及 `nums[mid]` | 判断与边界更新 | 搜索区间剩余情况 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **初始** | `0` | `5` | `mid=2`, 值 `7` | `7 < 8`，选右半边<br>➔ `L = mid + 1 = 3` | `[8, 8, 10]` |
| **次轮** | `3` | `5` | `mid=4`, 值 `8` | `8 >= 8`，向左逼近<br>➔ `R = mid - 1 = 3` | `[8]` |
| **尾轮** | `3` | `3` | `mid=3`, 值 `8` | `8 >= 8`，继续向左逼近<br>➔ `R = mid - 1 = 2` | 区间变为空 |
| **结束** | `3` | `2` | / | 循环结束 `L > R` | 返回 ` L =  `**`3`** |
*校验：* `start = 3`，`nums[3] == 8`，合法。

**第二步：找终点 `lower_bound(nums, 8 + 1) - 1`**
（即找第一个 $\ge 9$ 的位置）

| 步骤 | 左边界 `L` | 右边界 `R` | 中间值 `mid` 及 `nums[mid]` | 判断与边界更新 | 搜索区间剩余情况 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **初始** | `0` | `5` | `mid=2`, 值 `7` | `7 < 9`，选右半边<br>➔ `L = mid + 1 = 3` | `[8, 8, 10]` |
| **次轮** | `3` | `5` | `mid=4`, 值 `8` | `8 < 9`，选右半边<br>➔ `L = mid + 1 = 5` | `[10]` |
| **尾轮** | `5` | `5` | `mid=5`, 值 `10` | `10 >= 9`，向左逼近<br>➔ `R = mid - 1 = 4` | 区间变为空 |
| **结束** | `5` | `4` | / | 循环结束 `L > R` | 返回 ` L =  `**`5`** |
*计算终点：* ` end = 5 - 1 =  `**`4`**。
最终返回 `[3, 4]`。
