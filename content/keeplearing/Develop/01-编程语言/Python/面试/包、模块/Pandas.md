> Pandas 是 Python **数据分析**的核心库，广泛用于 **数据处理、清洗、分析和可视化**。
### 数据结构
#### 1. Series
##### 定义
Series 是 Pandas 中的一个核心数据结构，类似于一个一维的数组，具有数据和索引。
Series 可以存储任何数据类型（整数、浮点数、字符串等），并通过标签（索引）来访问元素。
##### Series 特点：
- **一维数组**：Series 中的每个元素都有一个对应的索引值。
- **索引**：每个数据元素都可以通过标签（索引）来访问，默认情况下索引是从 0 开始的整数，但你也可以自定义索引。
- **数据类型**：`Series` 可以容纳不同数据类型的元素，包括整数、浮点数、字符串、Python 对象等。
- **大小不变性**：Series 的大小在创建后是不变的，但可以通过某些操作（如 append 或 delete）来改变。
- **操作**：Series 支持各种操作，如数学运算、统计分析、字符串处理等。
- **损失数据**：Series 可以包含缺失数据，Pandas 使用NaN（Not a Number）来表示缺失或无值。
- **自动对齐**：当对多个 Series 进行运算时，Pandas 会自动根据索引对齐数据，这使得数据处理更加高效。
##### Series 结构：
Series 由三个部分组成：**索引（Index）**、**名称（Name）** 以及**值（Values）**：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714114950553.png)
##### 创建 Series：
使用 pd.Series() 构造函数创建一个 Series 对象，传递一个数据数组（可以是列表、NumPy 数组等）和一个可选的索引数组：
```python
pandas.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
```
**参数说明**：
- `data`：Series 的数据部分，可以是列表、数组、字典、标量值等。如果不提供此参数，则创建一个空的 Series。
- `index`：Series 的索引部分，用于对数据进行标记。可以是列表、数组、索引对象等。如果不提供此参数，则创建一个默认的整数索引。
- `dtype`：指定 Series 的数据类型。可以是 NumPy 的数据类型，例如 `np.int64`、`np.float64` 等。如果不提供此参数，则根据数据自动推断数据类型。
- `name`：Series 的名称，用于标识 Series 对象。如果提供了此参数，则创建的 Series 对象将具有指定的名称。
- `copy`：是否复制数据。默认为 False，表示不复制数据。如果设置为 True，则复制输入的数据。
- `fastpath`：是否启用快速路径。默认为 False。启用快速路径可能会在某些情况下提高性能。
**创建一个简单的 Series 实例**：
```python
import pandas as pd

a = [1, 2, 3]

myvar = pd.Series(a)

print(myvar)
```
输出结果如下：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714115416452.png)
从上图可知，如果没有指定索引，索引值就从 0 开始，我们可以根据索引值读取数据：
```python
print(myvar[1]) # 输出为 2
```
可以指定索引值，如下实例：
```python
import pandas as pd

a = ["Google", "Runoob", "Wiki"]

myvar = pd.Series(a, index = ["x", "y", "z"])

print(myvar)
```
输出结果如下：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714115613759.png)
根据索引值读取数据:
```python
print(myvar["y"]) # 输出结果：Runoob
```

##### Series 方法：
| 方法名称                       | 功能描述                                |
| -------------------------- | ----------------------------------- |
| index                      | 获取 Series 的索引                       |
| values                     | 获取 Series 的数据部分（返回 NumPy 数组）        |
| head(n)                    | 返回 Series 的前 n 行（默认为 5）             |
| tail(n)                    | 返回 Series 的后 n 行（默认为 5）             |
| dtype                      | 返回 Series 中数据的类型                    |
| shape                      | 返回 Series 的形状（行数）                   |
| describe()                 | 返回 Series 的统计描述（如均值、标准差、最小值等）       |
| isnull()                   | 返回一个布尔 Series，表示每个元素是否为 NaN         |
| notnull()                  | 返回一个布尔 Series，表示每个元素是否不是 NaN        |
| unique()                   | 返回 Series 中的唯一值（去重）                 |
| value_counts()             | 返回 Series 中每个唯一值的出现次数               |
| map(func)                  | 将指定函数应用于 Series 中的每个元素              |
| apply(func)                | 将指定函数应用于 Series 中的每个元素，常用于自定义操作     |
| astype(dtype)              | 将 Series 转换为指定的类型                   |
| sort_values()              | 对 Series 中的元素进行排序（按值排序）             |
| sort_index()               | 对 Series 的索引进行排序                    |
| dropna()                   | 删除 Series 中的缺失值（NaN）                |
| fillna(value)              | 填充 Series 中的缺失值（NaN）                |
| replace(to_replace, value) | 替换 Series 中指定的值                     |
| cumsum()                   | 返回 Series 的累计求和                     |
| cumprod()                  | 返回 Series 的累计乘积                     |
| shift(periods)             | 将 Series 中的元素按指定的步数进行位移             |
| rank()                     | 返回 Series 中元素的排名                    |
| corr(other)                | 计算 Series 与另一个 Series 的相关性（皮尔逊相关系数） |
| cov(other)                 | 计算 Series 与另一个 Series 的协方差          |
| to_list()                  | 将 Series 转换为 Python 列表              |
| to_frame()                 | 将 Series 转换为 DataFrame              |
| iloc[]                     | 通过位置索引来选择数据                         |
| loc[]                      | 通过标签索引来选择数据                         |
##### 注意事项：
- `Series` 中的数据是有序的。
- 可以将 `Series` 视为带有索引的一维数组。
- 索引可以是唯一的，但不是必须的。
- 数据可以是标量、列表、NumPy 数组等。
#### 2. DataFrame
##### 定义
DataFrame 是 Pandas 中的另一个核心数据结构，类似于一个**二维的表格或数据库中的数据表**。
DataFrame 是一个**表格型**的数据结构，它含有一组**有序的列**，每列可以是不同的值类型（数值、字符串、布尔型值）。
DataFrame 既有行索引也有列索引，它可以被看做由 Series 组成的字典（共同用一个索引）。
DataFrame 提供了各种功能来进行数据访问、筛选、分割、合并、重塑、聚合以及转换等操作。
DataFrame 是一个非常灵活且强大的数据结构，广泛用于数据分析、清洗、转换、可视化等任务。
##### DataFrame 特点
**二维结构**：`DataFrame` 是一个二维表格，可以被看作是一个 Excel 电子表格或 SQL 表，具有行和列。可以将其视为多个 `Series` 对象组成的字典。
- **列的数据类型**：不同的列可以包含不同的数据类型，例如整数、浮点数、字符串或 Python 对象等。
- **索引**：`DataFrame` 可以拥有行索引和列索引，类似于 Excel 中的行号和列标。
- **大小可变**：可以添加和删除列，类似于 Python 中的字典。
- **自动对齐**：在进行算术运算或数据对齐操作时，`DataFrame` 会自动对齐索引。
- **处理缺失数据**：`DataFrame` 可以包含缺失数据，Pandas 使用 `NaN`（Not a Number）来表示。
- **数据操作**：支持数据切片、索引、子集分割等操作。
- **时间序列支持**：`DataFrame` 对时间序列数据有特别的支持，可以轻松地进行时间数据的切片、索引和操作。
- **丰富的数据访问功能**：通过 `.loc`、`.iloc` 和 `.query()` 方法，可以灵活地访问和筛选数据。
- **灵活的数据处理功能**：包括数据合并、重塑、透视、分组和聚合等。
- **数据可视化**：虽然 `DataFrame` 本身不是可视化工具，但它可以与 Matplotlib 或 Seaborn 等可视化库结合使用，进行数据可视化。
- **高效的数据输入输出**：可以方便地读取和写入数据，支持多种格式，如 CSV、Excel、SQL 数据库和 HDF5 格式。
- **描述性统计**：提供了一系列方法来计算描述性统计数据，如 `.describe()`、`.mean()`、`.sum()` 等。
- **灵活的数据对齐和继承**：可以轻松地与其他 `DataFrame` 或 `Series` 对象进行合并、连接或更新操作。
- **转换功能**：可以对数据集中的值进行转换，例如使用 `.apply()` 方法应用自定义函数。
- **滚动窗口和时间序列分析**：支持对数据集进行滚动窗口统计和时间序列分析。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714120952423.png)
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714121010885.png)



##### 创建 DataFrame：
```python
pandas.DataFrame(data=None, index=None, columns=None, dtype=None, copy=False)
```
**参数说明**：
- `data`：DataFrame 的数据部分，可以是字典、二维数组、Series、DataFrame 或其他可转换为 DataFrame 的对象。如果不提供此参数，则创建一个空的 DataFrame。
- `index`：DataFrame 的行索引，用于标识每行数据。可以是列表、数组、索引对象等。如果不提供此参数，则创建一个默认的整数索引。
- `columns`：DataFrame 的列索引，用于标识每列数据。可以是列表、数组、索引对象等。如果不提供此参数，则创建一个默认的整数索引。
- `dtype`：指定 DataFrame 的数据类型。可以是 NumPy 的数据类型，例如 `np.int64`、`np.float64` 等。如果不提供此参数，则根据数据自动推断数据类型。
- `copy`：是否复制数据。默认为 False，表示不复制数据。如果设置为 True，则复制输入的数据。
Pandas DataFrame 是一个二维的数组结构，类似二维数组。
实例 - 使用列表创建
```python
import pandas as pd

data = [['Google', 10], ['Runoob', 12], ['Wiki', 13]]

# 创建DataFrame
df = pd.DataFrame(data, columns=['Site', 'Age'])

# 使用astype方法设置每列的数据类型
df['Site'] = df['Site'].astype(str)
df['Age'] = df['Age'].astype(float)

print(df)
```
也可以使用字典来创建：
```python
import pandas as pd

data = {'Site':['Google', 'Runoob', 'Wiki'], 'Age':[10, 12, 13]}

df = pd.DataFrame(data)

print (df)
```
输出结果如下：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250714121722169.png)
##### 注意事项
- `DataFrame` 是一种灵活的数据结构，可以容纳不同数据类型的列。
- 列名和行索引可以是字符串、整数等。
- `DataFrame` 可以通过多种方式进行数据选择、过滤、修改和分析。
- 通过对 `DataFrame` 的操作，可以进行数据清洗、转换、分析和可视化等工作

### 面试题
#### Pandas 中的两种核心数据结构是什么？
`Series` 和 `DataFrame`
```python
# Series: 类似于一维数组 
s = pd.Series([10, 20, 30, 40])  
# DataFrame: 类似于二维表格 
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
```

#### 如何从 CSV、Excel、SQL 读取数据？
`pd.read_csv()`、`pd.read_excel()`、`pd.read_sql()`
```python
# 读取 CSV 
df = pd.read_csv("data.csv")  
# 读取 Excel 
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")  
# 连接 SQL 数据库（示例） 
import sqlite3 
conn = sqlite3.connect("database.db") 
df = pd.read_sql("SELECT * FROM table_name", conn)
```

#### 如何将 DataFrame 保存到 CSV 或 Excel？
`to_csv()`、`to_excel()`
```python
df.to_csv("output.csv", index=False)  # 保存为 CSV，不包含索引 
df.to_excel("output.xlsx", sheet_name="Sheet1", index=False)  # 保存为 Excel 
```

#### 如何获取 DataFrame 的基本信息？
`df.shape`、`df.info()`、`df.describe()`
```python
print(df.shape)  # (行数, 列数) 
print(df.info())  # 数据类型、缺失值 
print(df.describe())  # 统计信息 
```

#### 如何处理缺失值（NaN）？
`dropna()`、`fillna()`
```python
df = pd.DataFrame({'A': [1, np.nan, 3], 'B': [4, 5, np.nan]})  
# 删除缺失值 
df.dropna()  
# 删除含 NaN 的行 
df.dropna(axis=1)  # 删除含 NaN 的列  
# 填充缺失值 
df.fillna(0)  # 用 0 填充 
df.fillna(df.mean())  # 用列的均值填充 
```
#### 如何去重？
`drop_duplicates()`
```python
df = pd.DataFrame({'A': [1, 2, 2, 3], 'B': [4, 5, 5, 6]})  
df.drop_duplicates()  # 删除重复行 
df.drop_duplicates(subset=['A'], keep='first')  # 只考虑 'A' 列，保留第一条 
```
#### 如何修改列名和索引？
`rename()`、`set_index()`
```python
df = pd.DataFrame({'old_name': [1, 2, 3], 'B': [4, 5, 6]})  
# 修改列名 
df.rename(columns={'old_name': 'new_name'}, inplace=True)  
# 修改索引 
df.set_index('new_name', inplace=True) 
```
#### 如何处理类别数据？
`pd.get_dummies()`、`astype('category')`
```python
df = pd.DataFrame({'Category': ['A', 'B', 'A', 'C']})  
# 独热编码（One-Hot Encoding） 
df_encoded = pd.get_dummies(df, columns=['Category'])  
# 转换为类别类型 
df['Category'] = df['Category'].astype('category') 
```
#### 如何合并多个 DataFrame？
`concat()`、`merge()`
```python
df1 = pd.DataFrame({'ID': [1, 2], 'Name': ['Alice', 'Bob']}) 
df2 = pd.DataFrame({'ID': [1, 2], 'Score': [90, 85]})  # 按 ID 连接（类似 SQL JOIN） 
df_merged = pd.merge(df1, df2, on='ID', how='inner')  # INNER JOIN  
# 按行合并 
df_concat = pd.concat([df1, df2], axis=0)  # 纵向合并 
df_concat = pd.concat([df1, df2], axis=1)  # 横向合并 
```
#### 如何对 DataFrame 进行分组聚合？
`groupby()`、`agg()`
```python
df = pd.DataFrame({'Category': ['A', 'B', 'A', 'B'], 'Value': [10, 20, 30, 40]})  # 按 'Category' 分组并求和 
print(df.groupby('Category').sum())  # 计算多个聚合指标 
df.groupby('Category').agg({'Value': ['sum', 'mean']}) 
```
#### Pandas 中如何使用 apply() 进行列操作？
`apply()`
```python
df = pd.DataFrame({'A': [1, 2, 3]})  # 对 'A' 列的每个元素计算平方 
df['A_squared'] = df['A'].apply(lambda x: x ** 2) 
```
#### 如何使用 Pivot Table（数据透视表）？
`pivot_table()`
```python
df = pd.DataFrame({'Category': ['A', 'B', 'A', 'B'],  'Type': ['X', 'X', 'Y', 'Y'],  'Value': [10, 20, 30, 40]})  
# 计算按 Category 和 Type 分组的均值 
df_pivot = df.pivot_table(index='Category', columns='Type', values='Value', aggfunc='mean') 
```
#### 如何优化 Pandas 计算性能？
向量化、`numba`、`categorical` 类型
```python
import numpy as np  # 向量化计算（比 Python for 循环快） 
df['Value_squared'] = df['Value'] ** 2  # 使用类别数据减少内存 
df['Category'] = df['Category'].astype('category')
```

#### 如何合并两个 dataframe 并做聚合
##### 最常见场景：根据某列合并，然后对另一列聚合
示例：两个 DataFrame
```python
import pandas as pd

df1 = pd.DataFrame({
    "id": [1, 2, 3],
    "value1": [10, 20, 30]
})

df2 = pd.DataFrame({
    "id": [1, 1, 2, 3, 3],
    "value2": [5, 7, 8, 4, 6]
})
```

##### 先 merge，再 groupby 聚合
1. 合并（按id）
```python
df = df1.merge(df2, on="id", how="left")
```
得到：

|id|value1|value2|
|---|---|---|
|1|10|5|
|1|10|7|
|2|20|8|
|3|30|4|
|3|30|6|

2. 再聚合：例如 sum
```python
result = df.groupby("id").agg({"value1": "first", "value2": "sum"}).reset_index()
```
输出：

|id|value1|value2|
|---|---|---|
|1|10|12|
|2|20|8|
|3|30|10|




























