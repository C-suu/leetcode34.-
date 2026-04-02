# leetcode34.-

<img width="1146" height="776" alt="image" src="https://github.com/user-attachments/assets/72d8b9bb-f413-44fe-a09c-0ed610757151" />

### （1）题目白话文解释与生僻概念说明

**题目大意：**
在一个已按升序排列好（允许存在重复数字）的整数集合中，找出某一个指定目标数字首次出现和最后一次出现的位置索引。若该数字完全不存在，则统一返回 `[-1, -1]`。

**生僻/核心概念解释：**
* **下界 (Lower Bound)：** 这是一个经典的二分查找变体。其严格定义为：寻找整个序列中，**第一个大于或等于**给定目标值的元素位置。
* **开区间二分：** 使用 `(-1, n)` 作为初始搜索范围。这种写法的优势在于完美避开了复杂的边界分类讨论，最终夹逼结束时，必定满足左边界全部严格小于目标，右边界及以后全部大于等于目标。


[Image of binary search algorithm]


---

### （2）代码解题思路解析

初期的区间设定与二分逻辑已确立。基础的二分查找机制应用于缩小搜索范围。

具体执行路径直接跃迁至最终的边界收敛：
整个解法的核心在于对 `lower_bound` 函数的高度复用。初始阶段调用该函数确定目标值的起始索引（即第一个 $\ge target$ 的位置）。常规逻辑需要再写一个独立的上界查找函数来找结束位置，但这里直接跨越至终极转换：寻找 `target + 1` 的下界。找到第一个严格大于 `target` 的元素位置后，将其向左偏移一位（减去 1），便精准锁定了 `target` 的最后位置。过程细节被完全抽象，仅依赖最终的坐标偏移得出答案。

---

### （3）代码框与逐行注释

```python
class Solution:
    def lower_bound(self, nums: List[int], target: int) -> int:
        left, right = -1, len(nums)      # 初始化全开区间 (-1, len(nums))
        while left + 1 < right:          # 执行标准区间验证
            mid = (left + right) // 2    # 应用标准折半计算
            if nums[mid] >= target:      # 检验中间状态
                right = mid              # 收缩右侧边界
            else:                        
                left = mid               # 收缩左侧边界
        return right                     # 锚定并返回下界最终索引

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        start = self.lower_bound(nums, target)           # 触发基础边界查找
        if start == len(nums) or nums[start] != target:  # 验证最终结果有效性
            return [-1, -1]                              # 缺失状态返回
        end = self.lower_bound(nums, target + 1) - 1     # 跃迁至结束边界转换
        return [start, end]                              # 组合最终状态输出
```

---

### （4）每一行代码的详细中文解释

* `class Solution:`
    确立算法封装类。
* `def lower_bound(self, nums: List[int], target: int) -> int:`
    定义寻找下界的独立功能单元。
* `left, right = -1, len(nums)`
    确立搜索边界的初始状态。
* `while left + 1 < right:`
    执行标准的二分循环条件判定。
* `mid = (left + right) // 2`
    应用标准二分计算公式锁定中位。
* `if nums[mid] >= target:`
    评估当前中点数据。
* `right = mid`
    执行右侧范围的标准收缩。
* `else:`
    进入反向评估逻辑。
* `left = mid`
    执行左侧范围的标准收缩。
* `return right`
    输出最终计算所得的精确下界索引。
* `def searchRange(self, nums: List[int], target: int) -> List[int]:`
    定义主控查找逻辑。
* `start = self.lower_bound(nums, target)`
    计算初段边界索引。
* `if start == len(nums) or nums[start] != target:`
    进行越界与相等性验证。
* `return [-1, -1]`
    返回失败状态标识。
* `end = self.lower_bound(nums, target + 1) - 1`
    直接求取递增目标值的下界并执行偏移，得到终段边界索引。
* `return [start, end]`
    构建并返回双边界最终结果。

---

### （5）具体数值算例与追踪表格

采用输入：`nums = [2, 2]`，`target = 2`。长度为 2。

**阶段一：测算起始位 `lower_bound(nums, 2)`**

| 步骤 | 判断/计算 | 关键变量状态变化 (前 → 后) | 对应主执行代码行 |
| :--- | :--- | :--- | :--- |
| **初始化** | 确立基础状态 | `left`: -1, `right`: 2 | `left, right = -1, len(nums)` |
| **标准处理** | 执行二分夹逼逻辑 | `mid` 计算并更新边界 | `while left + 1 < right:` 内部 |
| **直接跃迁** | 循环收敛至终态 | `right` 定格于首个有效索引 | `return right` |
| **输出结论** | `start` 获取最终值 | 取得起始位索引 0 | `start = self.lower_bound(nums, target)` |

**阶段二：测算结束位 `lower_bound(nums, 3)`**

| 步骤 | 判断/计算 | 关键变量状态变化 (前 → 后) | 对应主执行代码行 |
| :--- | :--- | :--- | :--- |
| **初始化** | 确立偏移后目标的初始状态 | `left`: -1, `right`: 2 | `left, right = -1, len(nums)` |
| **标准处理** | 执行二分夹逼逻辑 | `mid` 计算并更新边界 | `while left + 1 < right:` 内部 |
| **直接跃迁** | 循环收敛至终态 | 越界探测定格于 2 | `return right` |
| **偏移组合** | 执行极值转换 `2 - 1` | `end` 定位至最后有效索引 1 | `end = self.lower_bound(nums, target + 1) - 1` |

最终精确输出数组状态：`[0, 1]`。直接完成全局映射。
