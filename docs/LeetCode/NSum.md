## 两数之和

- [题目描述](https://leetcode-cn.com/problems/two-sum/)
- 方法：哈希

```java
public int[] twoSum(int[] nums, int target) {
    if (nums == null || nums.length == 0)
        return new int[0];
	Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement))
            return new int[] {i, map.get(complement)};
        map.put(nums[i], i);
    }
    return new int[0];
}
```

## 三数之和

- [题目描述](https://leetcode-cn.com/problems/3sum/)
- 方法：双指针。先固定一个，再挪动两个指针。

```java
public List<List<Integer>> threeSum(int[] nums, int target) {
    List<List<Integer> res = new ArrayList<>();
    Arrays.sort(nums);
    int len = nums.length;
    for (int i = 0; i < len - 2; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i - 1] == nums[i]) continue;
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == target) {
                res.add(Arrays.asList(nums[i], nums[l], nums[r]));
                // 去重
            }
        }
    }
}
```

## 最接近的三数之和

## 四数之和

