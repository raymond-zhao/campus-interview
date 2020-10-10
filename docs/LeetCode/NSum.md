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
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    if (nums == null || nums.length < 3) return res;
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 2; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                res.add(Arrays.asList(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l + 1]) ++l;
                while (l < r && nums[r] == nums[r - 1]) --r;
                ++l;
                --r;
            } else if (sum < 0) {
                ++l;
            } else --r;
        }
    }
    return res;
}
```

## 最接近的三数之和

- [最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)
- 方法：排序 + 双指针

```java
public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int len =  nums.length;
        int res = nums[0] + nums[1] + nums[2];
        for (int i = 0; i < len - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1])
                continue;
            int l = i + 1, r = len - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (target == sum) return target;
                if (Math.abs(sum - target) < Math.abs(res - target))
                    res = sum;
                if (sum < target) {
                    ++l;
                    while (l < r && nums[l] == nums[l - 1]) ++l;
                } else {
                    --r;
                    while (l < r && nums[r] == nums[r + 1]) --r;
                }
            }
        }
        return res;
    }
```

## 四数之和

- [四数之和](https://leetcode-cn.com/problems/4sum/)
- 解法：排序 + 双指针

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res = new ArrayList<>();
    if (nums == null || nums.length < 4) return res;
    Arrays.sort(nums); // [-2, -1, 0, 0, 1, 2]
    for (int i = 0; i < nums.length - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        for (int j = i + 1; j < nums.length - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;
            int l = j + 1, r = nums.length - 1;
            while (l < r) {
                int sum = nums[i] + nums[j] + nums[l] + nums[r];
                if (sum == target) {
                    res.add(Arrays.asList(nums[i], nums[j], nums[l], nums[r]));
                    while (l < r && nums[l] == nums[l + 1]) ++l;
                    while (l < r && nums[r] == nums[r - 1]) --r;
                    ++l;
                    --r;
                } else if (sum < target) {
                    ++l;
                } else {
                    --r;
                }
            }
        }
    }
    return res;
}
```

