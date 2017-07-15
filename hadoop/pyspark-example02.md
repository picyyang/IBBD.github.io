# Example02: 使用PySpark进行数据探索

## Step01: 加载csv文件
正确的加载数据，是所有工作的基础。

```python
res = spark.read.csv("/data/jm/practice/jm_daoall2_tags3/000000_0_format/result/*")
```

加载数据的时候，可以对各个字段的类型进行定义，也可以指定是否有header等。

## Step02: 查看基础的数据结构
加载好数据之后，首先需要理解数据结构，主要是字段及其类型。

```python
res.columns
res.first()
res.take(3)
res.head()
# Row(_c0='811002608346736', _c1='1', _c2='1492017073', _c3='1492018055', _c4='江门天长路', _c5='113.0846516', _c6='22.61743464', _c7='蓬江', _c8='环市街办', _c9='否', _c10='宏站', _c11='室外', _c12='城区道路', _c13='0', _c14='N', _c15='江门天长路F-ZLH-1', _c16='16', _c17='977', _c18='0', _c19='0')
```

## Step03: 描述性统计
对数值型字段，可以做描述性统计

```python
res.select('_c17').describe().show()
+-------+------------------+
|summary|              _c17|
+-------+------------------+
|  count|              1978|
|   mean|110.38018200202224|
| stddev|1209.8913469284548|
|    min|                 0|
|    max|                99|
+-------+------------------+
```

## Step04: 字段交叉统计
经过前面之后，我们大概对数据有所了解了，不过还不够，我们还需要对一些关键的字段做统计

```python
res.groupby('_c0').count().show()
+---------------+-----+
|            _c0|count|
+---------------+-----+
|811002608346736|  514|
|811002443262645|  895|
|811078167875023|  568|
|811078025544915|    1|
+---------------+-----+

res.crosstab('_c0', '_c18').show()
+---------------+---+---+
|       _c0__c14|  0|  1|
+---------------+---+---+
|811078025544915|  1|  0|
|811078167875023|612|  0|
|811002608346736|544| 18|
|811002443262645|908| 19|
+---------------+---+---+

res.crosstab('_c0', '_c19').show()
+---------------+---+---+---+---+
|       _c0__c15|  0|  1|  2|  5|
+---------------+---+---+---+---+
|811078025544915|  0|  0|  0|  1|
|811078167875023|299|  0|  0|313|
|811002608346736|210| 93|  1|258|
|811002443262645|424| 13| 13|477|
+---------------+---+---+---+---+
```

注：怎么统计唯一值，貌似比较没有提供相应的函数，当然先去重再统计也是可以的，但是那样就麻烦了。

## Step05: 提取相关的字段
了解了数据之后，就可以提取相关的字段进行分析了：

```python
res.filter(res._c0=='811078167875023').select('_c1', '_c2', '_c3', '_c16', '_c17', '_c13', '_c14', '_c18', '_c19', '_c9', '_c10', '_c11', '_c4').show(400)
```


