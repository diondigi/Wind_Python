# 多因子选股模型

multifactorial_model文件夹中是目前最完整的。

**版本1和2**是初级的实现。对数据库进行了多次操作，更多的是考虑功能，性能问题暂时作为次要的。

**版本3**是改进版本，对原始数据可以实现自动获取、存储、计算衍生指标并存储。除此之外，还可以对数据进行自动清洗，包括缺失数据补全（由于wind对单次数据请求量的的限制，需要多种补全方式），异常值的修正。最后计算某一月末的所有股票的各个指标的t值、correlation相关性系数，并存储到csv文件中。

**版本4**中会更多的使用pandas和sql语句实现功能，以提高性能。

#### 下面是修正思路：

**原先：**所有数据存储到mysql数据中，每次计算新指标时，从数据库中取出、运算、写入数据表中，每一个指标的运算都要重复取出、运算、写入的过程
优点：内存消耗低
**缺点：**多次操作数据库，耗时长

##### 修正1：

将数据表的所有数据按照股票代码，分次取出该股票代码对应的从ipodate开始到2016-12-31的所有指标的数据，连续多年的数据如果一次无法取出，那就分年份取出该股票的数据。 最终用pandas的to_sql可以将所有数据存入数据库。

##### 修正2：

加减乘除可以在直接构造sql语句，在mysql里面进行操作。而且对空值，可以用sql语句指定处理方式。SQL ISNULL()、NVL()、IFNULL() 和 COALESCE() 函数

##### 运算过程：

从数据表中按照股票代码用pandas的方法从数据表中取出多年的所有数据，用pandas的方法对数据进行运算。所有的衍生指标的计算，缺失值，异常值，标准化的运算和处理直接在内存中进行，最终将得到的dataframe结果用to_sql存入另一个新的数据表中。

**优点：**计算速度会加快
**缺点：**有潜在的内存耗尽的风险

#### 根据最近的项目，产生的新想法：

##### Future Work：

- 利用Spark进行数据清洗、预处理、因子计算、回归检验、策略回测
- 针对数据存储部分，如果要拓展到海外市场，比如美股、港股等，可以考虑用hbase和hive，对海量数据的操作会比Mysql性能要好。
- 针对选股策略，可以考虑用尝试Spark MLlib中的机器学习算法。
- 针对回测部分，可以考虑结合Spark Streaming，对接交易所的数据，尝试实时回测。

### 2018-10-14 补充：
#### 移动平均值等计算
     可以采用: (1)pandas的rolling_max, rolling_min, rolling_corr, rolling_std函数


