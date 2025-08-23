
# 数组操作

## 1. 前缀和

适用于**原始数组不会被修改的情况下，频繁查询某个区间的累加和**的场景

### 一维数组前缀和

```
a b c d e f g        0 a a+b a+b+c a+b+c+d a+b+c+d+e a+b+c+d+e+f a+b+c+d+e+f+g 
0 1 2 3 4 5 6 =>     0 1  2    3      4        5          6            7  
```


```cpp
class NumArray {
	vector<int> preSum;
public:
	NumArray(vector<int>& nums) {
		int m = nums.size();
		preSum.resize(m + 1);
		if (m == 0) return;
		
		for (int i = 1; i <= m; ++i) {	
			preSum[i] = preSum[i - 1] + nums[i - 1];
		}
	}

	int sumRange(int left, int right) {
		return preSum[right + 1] - preSum[left];
	}
};
```

### 二维数组前缀和

```
  0 1        0   1           2
0 a b =>   0 0   0           0
1 c d      1 0   a         a + b 
		   2 0 a + c     a + b + c + d
```

```cpp
class NumArray {
	vector<vector<int>> preSum;
public:
	NumArray(vector<vector<int>>& mat) {
		int m = mat.size(), n = mat.size();
		if (m == 0) return;

		preSum.resize(m + 1, vector<int>(n + 1, 0));

		for (int i = 1; i <= m; ++i) {
			preSum[i][j] = preSum[i - 1][j] + preSum[i][j - 1] + mat[i - 1][j - 1] 
				- preSum[i - 1][j - 1]; 
		}
	}

	int sumRange(int row1, int col1, int row2, int col2) {
		return preSum[row1 + 1][col1 + 1] - preSum[row1][col2 + 1] +    preSum[row2 + 1][col1] + preSum[row1][col1];
	}
};
```

## 2. 差分数组


适用于**需要频繁对原始数组的某个区间的元素进行增减**的场景

原数组 `nums`：

```
a b c d e
```

构造一个差分数组  `diff`：

```
a b-a c-b d-c e-d
```

从 `diff` 可以反推出 `nums`：

```
a a+(b-a) a+(b-a)+(c-b) a+(b-a)+(c-b)+(d-c) a+(b-a)+(c-b)+(d-c)+(e-d)
```

即：

```
nums = diff[0], diff[1] + diff[0], diff[2] + diff[1] + diff[0] ...
```

这是前缀和算法。

所以如果对 `nums` 执行从 `nums[i]` 到 `nums[j] (0 <=i <= j <= nums.size()) ` 所有元素加`x`的操作，只需要执行 `diff[i] += x; diff[j + 1] -= x` 即可；因为 `diff[i] += x` 等同于 `nums[i...]` 均加了 `x`，`diff[j + 1] -= x` 等同于 `nums[j + 1 ...] -= x` 这样一来就 `nums[i -> j]` 加了 `x`。

### 区间加法

```cpp
class Solution {
public:
    vector<int> getModifiedArray(int length, vector<vector<int>>& updates) {
        vector<int> nums(length, 0);
        
        Difference df(nums);
        for (const auto& update : updates) {
            int i = update[0];
            int j = update[1];
            int val = update[2];
            df.increment(i, j, val);
        }
        return df.result();
    }

	// manually
    class Difference {
        vector<int> diff;

    public:
        Difference(const vector<int>& nums) {
            assert(!nums.empty());
            diff.resize(nums.size());

            diff[0] = nums[0];
            for (size_t i = 1; i < nums.size(); ++i) {
                diff[i] = nums[i] - nums[i - 1];
            }
        }

        void increment(int i, int j, int val) {
            diff[i] += val;
            if (j + 1 < diff.size()) {
                diff[j + 1] -= val;
            }
        }

        vector<int> result() {
            vector<int> res(diff.size());

            res[0] = diff[0];
            for (size_t i = 1; i < diff.size(); ++i) {
                res[i] = res[i - 1] + diff[i];
            }
            return res;
        }
    };
};
```

## 3. 二维数组
