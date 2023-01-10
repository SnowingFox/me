---
title: Algorithm - Snowingfox
display: Algorithm
subtitle: To record my roadmap of algorithm.
---


## 链表

### [remove-nth-node-from-end-of-list](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
Reference: [快慢指针](https://programmercarl.com/0019.%E5%88%A0%E9%99%A4%E9%93%BE%E8%A1%A8%E7%9A%84%E5%80%92%E6%95%B0%E7%AC%ACN%E4%B8%AA%E8%8A%82%E7%82%B9.html#_19-%E5%88%A0%E9%99%A4%E9%93%BE%E8%A1%A8%E7%9A%84%E5%80%92%E6%95%B0%E7%AC%ACn%E4%B8%AA%E8%8A%82%E7%82%B9)

## 双指针
### [四数之和](https://leetcode.cn/problems/4sum/)
```ts
function fourSum(nums: number[], target: number): number[][] {
  nums = nums.sort((a, b) => a - b)
  const ret = []

  for (let i = 0; i < nums.length; i++) {
    if (i > 0 && nums[i] === nums[i - 1])
      continue

    for (let j = i + 1; j < nums.length; j++) {
      let left = j + 1
      let right = nums.length - 1
      if (j > i + 1 && nums[j] === nums[j - 1])
        continue

      while (left < right) {
        const flag = nums[i] + nums[j] + nums[left] + nums[right]
        if (flag > target) {
          right--
        }
        else if (flag < target) {
          left++
        }
        else {
          ret.push([nums[i], nums[j], nums[left], nums[right]])
          left++
          right--
          while (nums[left] === nums[left - 1]) left++
          while (nums[right] === nums[right + 1]) right--
        }
      }
    }
  }

  return ret
}
```
