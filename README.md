# FastXTab

翻译：xinjie   2019.07.08

该项目是对 FastXTab 类的升级，最初由 Alexander Golovlev 和 [Vilhelm-Ion Praisach](https://github.com/vgulielmus) 创建. 原始版本下载地址： http://www.universalthread.com/ViewPageNewDownload.aspx?ID=9944.

FastXTab 是 VFP 随附的 VFPXTab 的替代品。 它用于至少有三个字段的表或游标。 默认情况下，第一个字段中的数据成为结果中的行，第二个字段中的数据成为结果中的列，第三个字段中的数据被聚合（默认情况下求和）以形成所需的数据处理结果。 VFPXTab 速度很慢且功能有限。 FastXTab 则更快更强大。

FastXTab.prg 包含所有的源代码。针对不同的版本，存在两个源代码和示例目录：

* FastXTab 1.6 用于 VFP 9
* FastXTabs6 1.6 用于 VFP 6.

## 新属性
有关属性的完整列表，请参阅下面的“属性”部分。

- nAvePrec: 使用 AVE 聚合函数(算术平均值)时的精度（本属性所对应的默认值：3），数据类型为双精度型
- cPageField: 指定用于数据分页的字段名或表达式
- cRowField: 指定表示行的字段名或表达式
- nRowField2: 当 nRowField2 = 0 时指定 cRowField 用于数据分类（如果需要，还可以指定 cPageField 属性）
- cColField: 指定表示列的字段名或表达式
- cDataField: 指定表示数据的字段名或表达式
- nFunctionType: 聚合函数 1 Sum 2 Count 3 Avg 4 Min 5 Max 6 自定义 (数值字段的默认值为1，非数值字段的默认值为5)
- cFunctionExp: 当 nFunctionType=6 时的计算表达式
- cCondition: WHERE 子句
- cHaving: HAVING 子句
- nMultiDataField: 如果 nMultiDataField > 1，则可以为每列定义更多 DataField / FunctionType / FunctionExp
- anDataField[1],anFunctionType[1],acFunctionExp[1],acDataField[1]: 当 nMultiDataField > 1 时，nDataField，nFunctionType，cFunctionExp，cDataField的等效属性

### 新的特性
- 当 EMPTY(cRowField) 和 nRowField = 0 时，根据 cDataField，nFunctionType，cFunctionExp, cColField 属性设置，交叉表仅按列分配值。 （忽略 nFunctionType <> 6 的值）
- 聚合函数对非数字字段的权限（1表示求和，3表示平均值，默认为max）

产生的游标数据类型：
1. EMPTY(cRowField) 和 nRowField = 0 (按列分配)：
    - 与 nFunctionType <> 6 时的字段类型相同
    - 当 nFunctionType = 6 时从结果中获取

2. !EMPTY(cRowField) 和 nRowField <> 0：
    - nFunctionType = 2（COUNT） 时为整型
    - nFunctionType = 3（AVERAGE） 时为双精度型，小数位由 nAvePrec 属性指定
    - 取自 nFunctionType = 6 或 nFunctionType = 1 时的结果（以避免数据溢出）
    - 与源表字段类型相同

### 其他的更新
- 使用 "m." 前缀 
- 添加了 Local 语句
- 内部游标名称使用 SYS(2015)

### 一些例子：
在项目示例交叉表的测试表单中，Foxite1 方法显示了 Foxite 上最近几个场景的解决方案。其他示例作为注释发布在 cmdfastxtab 命令按钮的 click 方法中。

1. For http://www.foxite.com/archives/sql-help-0000401315.htm:

    ```foxpro
    oXTab.cRowField='cstcode'
    oXtab.cColField = 'subj'
    oXtab.nMultiDataField=3
    oXtab.acDataField[1] = 'subj'
    oXtab.anFunctionType[1] = 2
    oXtab.anFunctionType[2] = 6
    oXtab.acFunctionExp[2]="SUM(IIF(attend='P',1,0))"
    oXtab.anFunctionType[3] = 6
    oXtab.acFunctionExp[3]="SUM(IIF(attend='P',1,0))/COUNT(attend)*100"
    ```

2. For http://www.foxite.com/archives/row-to-column-0000401353.htm:

    ```foxpro
	oXtab.nRowField = 0 
	oXtab.cRowField = ''
	oXtab.cColField='ids'
	oXtab.cDataField ='qty'
	```

3. For http://www.foxite.com/archives/split-numbers-2-0000400387.htm:
	
    ```foxpro
    oXtab.nRowField = 0 
	oXtab.cRowField = ''
	oXtab.cColField='floor(no/10)+1'
	oXtab.cDataField ='no'
	```

4. For http://www.foxite.com/archives/split-numbers-2-0000400495.htm:

    ```foxpro
    oXtab.nRowField = 0 
    oXtab.cRowField = ''
    oXtab.cColField='floor(no/100000)+1'
    oXtab.cDataField ='no'
	```

## 属性
| 属性    | 描述 |
|-------------|-------------|
| **源表** ||
| lCloseTable   | .T. 生成交叉表后关闭源表 |
| **交叉表** ||
| lCursorOnly   | .T. 表示结果为游标，.F. 表示结果为自由表 |
| cOutFile      | 交叉表的表名 |
| lDisplayNulls | .T. / .F. 对应 Set null ON / OFF |
| lBrowseAfter  | 指定是否在生成交叉表后使用 Brow 命令浏览 |
| **交叉表 a. 行** ||
| cRowField     | 行（分组）的字段名称/字段表达式 |
| nRowField     | 行（分组）的字段位置（AFIELDS(,cSource) 中的行号） |
| cPageField    | 用于数据分页的字段名/字段表达式（可选） |
| nPageField    | 数据分页的字段位置（AFIELDS(,cSource) 中的行号） |
| **b. 列** ||
| cColField     | 列（分组）的字段名称/字段表达式 |
| nColField     | 列（分组）的字段位置（AFIELDS(,cSource) 中的行号） |
| **c. "单元格"** ||
| cDataField    | 表示单元格数据的字段名 |
| nDataField    | 单元格数据字段的位置（AFIELDS(,cSource) 中的行号） |
| nFunctionType | 用于单元格的聚合函数： 1 = Sum, 2 = Count, 3 = Avg, 4 = Min, 5 = Max, 6 = Custom |
| cFunctionExp  | 当 nFunctionType=6 时所使用的自定义表达式(如果 <> 6 则忽略) |
| **d. 某些列包含多个数据（单元格）列** ||
| nMultiDataField | 数据（单元格）列数（默认值= 1） |
| acDataField | 包含单元格字段名称的数组 |
| anDataField | 单元格的数据字段位置的数组（AFIELDS(,cSource) 中的行号） |
| anFunctionType | 用于单元格的聚合函数的数组： 1 = Sum, 2 = Count, 3 = Avg, 4 = Min, 5 = Max, 6 = Custom |
| acFunctionExp | 当 anFunctionType()=6 时所使用的自定义表达式数组 |
| **e. 杂项** ||
| nAvePrec | 当 nFunctionType = 3 (average) 时的小数位 |
| cCondition | Where 子句表达式 |
| cHaving | Having 子句表达式 |
| nRowField2 | 当 nRowField2 = 0 且 !empty（cRowField）时，FastXTab 按列和行分配单元格（根据cRowField和cColField）。 当 nRowField2 <> 0 或empty(This.cRowField) 时被忽略 |
| lTotalRows | 如果为 .T.，则在交叉表种添加合计行 |

### 说明

有三种类型的输出：

1. 当 nRowField2 = 0 和 !empty(cRowField) 时，FastXTab 按列和行分配单元格（根据 cRowField 和 cColField）; 没有聚合函数被执行。 如果nFunctionType / anFunctionType = 6，则单元格包含来自 cFunctionExp / acFunctionExp 的表达式结果。 否则，单元格包含 cDataField 中的字段。
      
2. 当 nRowField = 0 和 EMPTY(cRowField) 时，FastXTab 按列分配单元格（根据 cColField）; 没有聚合函数被执行。 如果 nFunctionType / anFunctionType = 6，则单元格包含来自 cFunctionExp / acFunctionExp 的表达式结果。 否则，单元格包含 cDataField 中的字段。

3. 其他，FastXTab 应用聚合函数并按列和行分配结果（根据 cPageField，cRowField 和 cColField）。

    * 如果 nFunctionType / anFunctionType = 1, 单元格包含 SUM(cDataField).
    * 如果 nFunctionType / anFunctionType = 2, 单元格包含 COUNT(cDataField).
    * 如果 nFunctionType / anFunctionType = 3, 单元格包含 AVERAGE(cDataField).
    * 如果 nFunctionType / anFunctionType = 4, 单元格包含 MAX(cDataField)
    * 如果 nFunctionType / anFunctionType = 5, 单元格包含 MIN(cDataField)
    * 如果 nFunctionType / anFunctionType = 6, 单元格包含来自 cFunctionExp / acFunctionExp 的表达式结果（必须是聚合点的有效表达式）
