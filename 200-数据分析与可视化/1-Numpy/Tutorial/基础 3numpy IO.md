[Importing data with `genfromtxt`](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#)

- [Defining the input](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#defining-the-input)
- Splitting the lines into columns
  - [The `delimiter` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-delimiter-argument)
  - [The `autostrip` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-autostrip-argument)
  - [The `comments` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-comments-argument)
- Skipping lines and choosing columns
  - [The `skip_header` and `skip_footer` arguments](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-skip-header-and-skip-footer-arguments)
  - [The `usecols` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-usecols-argument)
- [Choosing the data type](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#choosing-the-data-type)
- Setting the names
  - [The `names` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-names-argument)
  - [The `defaultfmt` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-defaultfmt-argument)
  - [Validating names](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#validating-names)
- Tweaking the conversion
  - [The `converters` argument](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#the-converters-argument)
  - [Using missing and filling values](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#using-missing-and-filling-values)
  - [`missing_values`](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#missing-values)
  - [`filling_values`](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#filling-values)
  - [`usemask`](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#usemask)
- [Shortcut functions](https://numpy.org/doc/1.19/user/basics.io.genfromtxt.html#shortcut-functions)

```python
>>> import numpy as np
>>> from io import StringIO
```



# 函数genfromtxt

numpy.genfromtxt**(***fname***,** *dtype=<class 'float'>***,** *comments='#'***,** *delimiter=None***,** *skip_header=0***,** *skip_footer=0***,** *converters=None***,** *missing_values=None***,** *filling_values=None***,** *usecols=None***,** *names=None***,** *excludelist=None***,** *deletechars=" !#$%&'()\*+***,** *-./:;<=>?@[\]^{|}~"***,** *replace_space='_'***,** *autostrip=False***,** *case_sensitive=True***,** *defaultfmt='f%i'***,** *unpack=None***,** *usemask=False***,** *loose=True***,** *invalid_raise=True***,** *max_rows=None***,** *encoding='bytes'***)**

步骤1：遍历字符串序列的每一行

步骤2：将每一个字符串转换为对应的数据类型

- 效率低，但是更灵活
- 同时考虑到缺失值(loadtext效率高，但不考虑缺失值)



数据文件：

- gzip：.gz
- bzip2：bz2



## 1. 将字符串按行分成列

- 跳过空行，或注释行

### 参数delimiter|分隔符

- 在CSV文件中用逗号或分号
- \t
- 默认None：默认空格(包含tabs) => 连续的空格视作一个空格

```python
>>> data = u"1, 2, 3\n4, 5, 6"
>>> np.genfromtxt(StringIO(data), delimiter=",")
array([[ 1.,  2.,  3.],
       [ 4.,  5.,  6.]])

### 特别：针对固定宽度的数据 => delimiter设置为整数，或一组整数
>>> data = u"  1  2  3\n  4  5 67\n890123  4"
>>> np.genfromtxt(StringIO(data), delimiter=3)   #每3个字符组成一个数
array([[   1.,    2.,    3.],
       [   4.,    5.,   67.],
       [ 890.,  123.,    4.]])
>>> data = u"123456789\n   4  7 9\n   4567 9"
>>> np.genfromtxt(StringIO(data), delimiter=(4, 3, 2))   #依次4个、3个、2个字符组成一个数
array([[ 1234.,   567.,    89.],
       [    4.,     7.,     9.],
       [    4.,   567.,     9.]])
```



### 参数autostrip|是否删除各个条目的前导或尾随空格(两边空格)

```python
>>> data = u"1, abc , 2\n 3, xxx, 4"
>>> # Without autostrip
>>> np.genfromtxt(StringIO(data), delimiter=",", dtype="|U5")
array([['1', ' abc ', ' 2'],   #2之前有空格
       ['3', ' xxx', ' 4']], dtype='<U5')
>>> # With autostrip
>>> np.genfromtxt(StringIO(data), delimiter=",", dtype="|U5", autostrip=True)
array([['1', 'abc', '2'],
       ['3', 'xxx', '4']], dtype='<U5')
```





### 参数comments|注释符号

默认comments='#'

- 版本1.7.0 comments="None"

````python
>>> data = u"""#
... # Skip me !
... # Skip me too !
... 1, 2
... 3, 4
... 5, 6 #This is the third line of the data
... 7, 8
... # And here comes the last line
... 9, 0
... """
>>> np.genfromtxt(StringIO(data), comments="#", delimiter=",")
array([[1., 2.],
       [3., 4.],
       [5., 6.],
       [7., 8.],
       [9., 0.]])
````

注意：如果参数names=True，那么第一行就单独作为对names的检查



## 2. 选择行、列

### skip_header：文件标头header

### skip_footer：文件的最后几行

```python
>>> data = u"\n".join(str(i) for i in range(10))
>>> np.genfromtxt(StringIO(data),)
array([ 0.,  1.,  2.,  3.,  4.,  5.,  6.,  7.,  8.,  9.])
>>> np.genfromtxt(StringIO(data),
...               skip_header=3, skip_footer=5)
array([ 3.,  4.])
```



### 参数 usecols|提取目标行

- 0表示第一行
- -1表示最后一行

```python
>>> data = u"1 2 3\n4 5 6"
>>> np.genfromtxt(StringIO(data), usecols=(0, -1))
array([[ 1.,  3.],
       [ 4.,  6.]])


### 提取目标列：names中的列名
>>> np.genfromtxt(StringIO(data),
...               names="a, b, c", usecols=("a", "c"))
array([(1.0, 3.0), (4.0, 6.0)],
      dtype=[('a', '<f8'), ('c', '<f8')])
>>> np.genfromtxt(StringIO(data),
...               names="a, b, c", usecols=("a, c"))
    array([(1.0, 3.0), (4.0, 6.0)],
          dtype=[('a', '<f8'), ('c', '<f8')])
```





## 选择数据类型

### 参数dtype

- 默认dtype=float
- 一组数据类型：dtype=(int, float, float)
- 逗号分隔：dtype="i4,f8,U3"
- 字典的keys(names或formats)
- 一组元组：(name, type) => dtype=[('A', int), ('B', float)]
- numpy.dtype对象
- 特殊值None

dtype=None是为了方便：但是明确指定数据类型更加高效





## 改变names

### 参数names

```python
### 方法1：明确指定names
>>> data = StringIO("1 2 3\n 4 5 6")
>>> np.genfromtxt(data, dtype=[(_, int) for _ in "abc"])
array([(1, 2, 3), (4, 5, 6)],
      dtype=[('a', '<i8'), ('b', '<i8'), ('c', '<i8')])

### 方法2：逗号分隔
>>> data = StringIO("1 2 3\n 4 5 6")
>>> np.genfromtxt(data, names="A, B, C")  #默认dtype=float
array([(1.0, 2.0, 3.0), (4.0, 5.0, 6.0)],
      dtype=[('A', '<f8'), ('B', '<f8'), ('C', '<f8')])

>>> data = StringIO("So it goes\n#a b c\n1 2 3\n 4 5 6")
>>> np.genfromtxt(data, skip_header=1, names=True)   #skip_header
array([(1.0, 2.0, 3.0), (4.0, 5.0, 6.0)],
      dtype=[('a', '<f8'), ('b', '<f8'), ('c', '<f8')])


### names覆盖
>>> data = StringIO("1 2 3\n 4 5 6")
>>> ndtype=[('a',int), ('b', float), ('c', int)]
>>> names = ["A", "B", "C"]
>>> np.genfromtxt(data, names=names, dtype=ndtype)
array([(1, 2.0, 3), (4, 5.0, 6)],
      dtype=[('A', '<i8'), ('B', '<f8'), ('C', '<i8')])
```



### 参数defaultfmt|更改names缺失的默认项

```python
>>> data = StringIO("1 2 3\n 4 5 6")   #注意此处StringIO数据的格式
>>> np.genfromtxt(data, dtype=(int, float, int))
array([(1, 2.0, 3), (4, 5.0, 6)],
      dtype=[('f0', '<i8'), ('f1', '<f8'), ('f2', '<i8')])


### 缺失的names会自动选择默认项
>>> data = StringIO("1 2 3\n 4 5 6")
>>> np.genfromtxt(data, dtype=(int, float, int), names="a")
array([(1, 2.0, 3), (4, 5.0, 6)],
      dtype=[('a', '<i8'), ('f0', '<f8'), ('f1', '<i8')])  #names默认f0 f1

# 更改names缺失的默认项
>>> data = StringIO("1 2 3\n 4 5 6")
>>> np.genfromtxt(data, dtype=(int, float, int), defaultfmt="var_%02i")
array([(1, 2.0, 3), (4, 5.0, 6)],
      dtype=[('var_00', '<i8'), ('var_01', '<f8'), ('var_02', '<i8')])
```



### 有效的names

结构化的dtype可以被看作[`recarray`](https://numpy.org/doc/1.19/reference/generated/numpy.recarray.html#numpy.recarray)，可以像属性一样访问字段

前提：name字段不含有任何空格，或者无效的字符

控制names的可选参数：

- deletechars：待删除的字符，默认是`~!@#$%^&*()-=+~\|]}[{';: /?.>,<`
- excludelist：不被允许的name列表，比如return, file；如果出现，会自动添加下划线
- case_sensitive：True/False, "upper"或"lower"





## 调整转换

### 参数converters|控制转换格式

- datetime：YYYY/MM/DD
- xx% ：百分数转化为浮点数

```python
>>> convertfunc = lambda x: float(x.strip(b"%"))/100.
>>> data = u"1, 2.3%, 45.\n6, 78.9%, 0"
>>> names = ("i", "p", "n")
>>> # General case .....
>>> np.genfromtxt(StringIO(data), delimiter=",", names=names)
array([(1., nan, 45.), (6., nan, 0.)],   #百分数无法被转换：用np.nan替代
      dtype=[('i', '<f8'), ('p', '<f8'), ('n', '<f8')])

>>> # Converted case ...
>>> np.genfromtxt(StringIO(data), delimiter=",", names=names,
...               converters={1: convertfunc})   #也可以用列名“p”代替index(1) converters={"p": convertfunc})
array([(1.0, 0.023, 45.0), (6.0, 0.78900000000000003, 0.0)],
      dtype=[('i', '<f8'), ('p', '<f8'), ('n', '<f8')])

>>> data = u"1, , 3\n 4, 5, 6"
>>> convert = lambda x: float(x.strip() or -999)   #如果x.strip()为False(空字符串)，那么默认返回-999作为标记
>>> np.genfromtxt(StringIO(data), delimiter=",",
...               converters={1: convert})
array([[   1., -999.,    3.],
       [   4.,    5.,    6.]])
```





## 填补缺失值

### 参数missing_values

- 默认空字符串为缺失值
- “N/A”，“???”也是缺失值、无效数据

1. 字符串，或者逗号分隔的字符串 => 作为缺失值的标记
2. 一组字符串 => 每一列的缺失值对应一个字符串标记
3. 字典 
   - 字典的values是字符串或一组字符串
   - 字典的keys是列的index(整数)，或者列的名字(字符串)；若为None表示所有列都适用



### 参数filling_values

默认：填补缺失值

| Expected type | Default     |
| :------------ | :---------- |
| `bool`        | `False`     |
| `int`         | `-1`        |
| `float`       | `np.nan`    |
| `complex`     | `np.nan+0j` |
| `string`      | `'???'`     |



1. 单个值
2. 一组值
3. 字典

```python
>>> data = u"N/A, 2, 3\n4, ,???"
>>> kwargs = dict(delimiter=",",
...               dtype=int,
...               names="a,b,c",
...               missing_values={0:"N/A", 'b':" ", 2:"???"},  #数据中的N/A用0标记
...               filling_values={0:0, 'b':0, 2:-999})   #针对标记0，最终用0来表示
>>> np.genfromtxt(StringIO(data), **kwargs)
array([(0, 2, 3), (4, 0, -999)],
      dtype=[('a', '<i8'), ('b', '<i8'), ('c', '<i8')])
```



### 参数 usemask

当数据缺失时返回True

[MaskedArray|ndarray的子类](https://numpy.org/doc/1.19/reference/maskedarray.baseclass.html#numpy.ma.MaskedArray)





# save/load

```python
np.save('my_array', ndarray)  #将ndarray保存到叫做my_array.npy的文件中
y = np.load('my_array.npy')
```



# #补充

模块|numpy.lib.io

- recfromtxt
  - usemask=False：返回standard [`numpy.recarray`](https://numpy.org/doc/1.19/reference/generated/numpy.recarray.html#numpy.recarray)
  - usemaske=True：返回MaskedRecords
- recfromcsv
  - 默认delimiter=","

