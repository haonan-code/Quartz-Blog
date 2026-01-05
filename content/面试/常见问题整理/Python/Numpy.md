> NumPy 是 Python 中用于**数值计算**的核心库，在数据分析、机器学习、深度学习等领域广泛应用。
#### 如何创建 NumPy 数组
`np.array()`、`np.zeros()`、`np.ones()`、`np.arange()`、`np.linspace()`
```python
import numpy as np  
# 用列表创建数组 
arr1 = np.array([1, 2, 3])  
# 创建全 0 数组 
arr2 = np.zeros((2, 3))  
# 2x3 矩阵  
# 创建全 1 数组 
arr3 = np.ones((3, 3))  
# 创建范围数组 
arr4 = np.arange(0, 10, 2)  # [0,2,4,6,8]  
# 创建等间隔数组 
arr5 = np.linspace(0, 1, 5)  # [0, 0.25, 0.5, 0.75, 1]  
print(arr1, arr2, arr3, arr4, arr5) 
```
#### 如何获取 NumPy 数组的形状、维度、大小、数据类型？
`shape`、`ndim`、`size`、`dtype`
```python
arr = np.array([[1, 2, 3], [4, 5, 6]])  
print(arr.shape)  # (2, 3)  -> 2 行 3 列 
print(arr.ndim)   # 2  -> 2 维数组 
print(arr.size)   # 6  -> 总共有 6 个元素 
print(arr.dtype)  # int64（根据系统可能不同） 
```
#### NumPy 数组的索引和切片
基本索引、切片、步长、布尔索引、花式索引
```python
arr = np.array([10, 20, 30, 40, 50])  
# 基本索引 
print(arr[2])  # 30  
# 切片 [start:end:step] 
print(arr[1:4])  # [20, 30, 40] 
print(arr[::2])  # [10, 30, 50]  
# 2D 数组索引 
arr2d = np.array([[1, 2, 3], [4, 5, 6]]) 
print(arr2d[1, 2])  # 6（第 1 行，第 2 列）  
# 布尔索引（筛选大于 25 的元素） 
print(arr[arr > 25])  # [30, 40, 50]  
# 花式索引（指定多个索引） 
print(arr[[0, 2, 4]])  # [10, 30, 50] 
```
#### NumPy 中的广播机制是什么？
```python
arr1 = np.array([[1, 2, 3], [4, 5, 6]]) 
arr2 = np.array([1, 2, 3])  # 1x3 矩阵  
# arr2 会自动扩展为 2x3 矩阵 
result = arr1 + arr2   
print(result) 
```
**规则**：
- 维度不同的数组可以进行运算，小数组会自动扩展。
- 例如：**(2,3) + (3,) → (2,3)**
#### 计算 NumPy 数组的均值、最大值、最小值
`np.mean()`、`np.max()`、`np.min()`
```python
arr = np.array([[1, 2, 3], [4, 5, 6]])  
print(np.mean(arr))  # 3.5 
print(np.max(arr, axis=0))  # 按列最大值 [4, 5, 6] 
print(np.min(arr, axis=1))  # 按行最小值 [1, 4] 
```
#### NumPy 提供的数学运算
`np.sum()`、`np.prod()`、`np.exp()`、`np.log()`、`np.sqrt()`
```python
arr = np.array([1, 2, 3])  
print(np.sum(arr))  # 6 
print(np.prod(arr))  # 6（1*2*3） 
print(np.exp(arr))   # [2.718, 7.389, 20.085]（指数） 
print(np.log(arr))   # [0, 0.693, 1.099]（自然对数） 
print(np.sqrt(arr))  # [1, 1.41, 1.73]（平方根） 
```
#### 如何使用 NumPy 生成随机数？
`np.random` 模块
```python
np.random.seed(42)  # 设置随机种子，保证结果可复现  
print(np.random.rand(3, 3))  # 生成 3x3 介于 [0,1) 之间的随机数 
print(np.random.randint(0, 10, (2, 3)))  # 生成 2x3 介于 [0,10) 之间的整数 
print(np.random.randn(3))  # 生成 3 个服从标准正态分布的数 
```
#### NumPy 如何处理缺失值？
`np.isnan()`、`np.nan`、`np.nanmean()`
```python
arr = np.array([1, np.nan, 3, np.nan, 5])  # 检测 NaN 
print(np.isnan(arr))  # [False  True False  True False]  
# 忽略 NaN 计算均值 
print(np.nanmean(arr))  # 3.0 
```
#### NumPy 如何合并和拆分数组？
`np.concatenate()`、`np.vstack()`、`np.hstack()`、`np.split()`
```python
arr1 = np.array([[1, 2], [3, 4]]) 
arr2 = np.array([[5, 6]])  
# 垂直合并（行方向） 
print(np.vstack((arr1, arr2)))  
# 水平合并（列方向） 
print(np.hstack((arr1, arr2.T)))  
# 拆分数组 
arr = np.array([1, 2, 3, 4, 5, 6]) 
print(np.split(arr, 3))  # [array([1, 2]), array([3, 4]), array([5, 6])] 
```
#### NumPy 如何提高计算性能？
`vectorization`（[向量化计算](https://zhida.zhihu.com/search?content_id=254202028&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F%E5%8C%96%E8%AE%A1%E7%AE%97&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTI2Mzk5MzUsInEiOiLlkJHph4_ljJborqHnrpciLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTQyMDIwMjgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.-axDSuL47jVUQlEhEuJu4lkv6TaDXv3xE6EcGvY33u8&zhida_source=entity)）、`numba`、`cython`
```python
import time  
# 普通 Python 计算 
arr = np.random.rand(1000000) 
start = time.time() 
result = [x**2 for x in arr] 
print("Python list time:", time.time() - start)  
# NumPy 向量化计算 
start = time.time() 
result = arr ** 2 
print("NumPy time:", time.time() - start) 
```
**NumPy 计算速度更快**，因为它基于 **C 语言实现**，避免了 Python 解释器的开销。
























