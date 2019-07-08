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
| **Input cursor / table** ||
| lCloseTable   | .T. the cursor / table which holds the data source is closed |
| **Output cursor / table** ||
| lCursorOnly   | .T. The result is stored in a cursor, otherwise in a free table |
| cOutFile      | Name of the cursor / table which holds the result |
| lDisplayNulls | .T. / .F. => Set null ON / OFF |
| lBrowseAfter  | Specifies whether to open a Browse window on the cross tab output |
| **CrossTab: a. Rows** ||
| cRowField     | Field name / Field expression for rows (group) |
| nRowField     | Field position (row number in AFIELDS(,cSource)) for rows (group) |
| cPageField    | Field name / Field expression for rows supergroup (optional) |
| nPageField    | Field position (row number in AFIELDS(,cSource)) for rows supergroup |
| **b. Columns** ||
| cColField     | Field name / Field expression for columns (group) |
| nColField     | Field position (row number in AFIELDS(,cSource)) for columns (group) |
| **c. Each column field holds a single data (cell) column** ||
| cDataField    | Field name for cells |
| nDataField    | Field position (row number in AFIELDS(,cSource)) for cells |
| nFunctionType | Aggregate function used for cells: 1 = Sum, 2 = Count, 3 = Avg, 4 = Min, 5 = Max, 6 = Custom |
| cFunctionExp  | The expression used for cells when nFunctionType=6 (ignored if nFunctionType<>6) |
| **d. Some columns contains more than a single data (cell) column** ||
| nMultiDataField | Number of data (cell) columns (default=1) |
| acDataField | Array with field names for cells |
| anDataField | Array with field positions (row number in AFIELDS(,cSource)) for cells |
| anFunctionType | Array with aggregate functions used for cells: 1 = Sum, 2 = Count, 3 = Avg, 4 = Min, 5 = Max, 6 = Custom |
| acFunctionExp | Array with the expressions used for cells when anFunctionType()=6 |
| **e. Miscellaneous** ||
| nAvePrec | Decimal precision when nFunctionType = 3 (average) |
| cCondition | Expression for a where condition |
| cHaving | Expression for a having condition |
| nRowField2 | When nRowField2 = 0 and !empty(cRowField), FastXTab distribute cells by columns and rows (according to cRowField and cColField). Ignored when nRowField2 <> 0 or empty(This.cRowField) |
| lTotalRows | When .T. a supplementary row with totals is added |

### Notes

There are three type of outputs:

1. When nRowField2 = 0 and !empty(cRowField), FastXTab distributes cells by columns and rows (according to cRowField and cColField); no aggregate functions are performed. If nFunctionType / anFunctionType = 6, cells contains the expression from cFunctionExp / acFunctionExp. Otherwise, cells contains the field from cDataField.
      
2. When nRowField = 0 and EMPTY(cRowField), FastXTab distributes cells by columns (according to cColField);  no aggregate functions are performed. If nFunctionType / anFunctionType = 6, cells contains the expression from cFunctionExp / acFunctionExp. Otherwise, cells contains the field from cDataField.

3. Otherwise FastXTab applies aggregate functions and distributes results by columns and rows (according to cPageField, cRowField and cColField).

    * If nFunctionType / anFunctionType = 1, cells contains SUM(cDataField).
    * If nFunctionType / anFunctionType = 2, cells contains COUNT(cDataField).
    * If nFunctionType / anFunctionType = 3, cells contains AVERAGE(cDataField).
    * If nFunctionType / anFunctionType = 4, cells contains MAX(cDataField)
    * If nFunctionType / anFunctionType = 5, cells contains MIN(cDataField)
    * If nFunctionType / anFunctionType = 6, cells contains the expression from cFunctionExp / acFunctionExp (must be valid expression from the point of the aggregation)
