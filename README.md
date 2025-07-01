# pandas
pandas interpret with sas and sql

## GroupBy
--- pandas.core.groupby.DataFrameGroupBy.filter
DataFrameGroupBy.filter(func, dropna=True, *args, **kwargs)[source]
Filter elements from groups that don’t satisfy a criterion.

```js
>>> df = pd.DataFrame({'A' : ['foo', 'bar', 'foo', 'bar',
...                           'foo', 'bar'],
...                    'B' : [1, 2, 3, 4, 5, 6],
...                    'C' : [2.0, 5., 8., 1., 2., 9.]})
>>> grouped = df.groupby('A')
>>> grouped.filter(lambda x: x['B'].mean() > 3.)
     A  B    C
1  bar  2  5.0
3  bar  4  1.0
5  bar  6  9.0
```

In SAS and SQL:
```js
/* 1. 创建数据集，相当于 Pandas 的 DataFrame */
data df;
   input A $ B C;
   datalines;
foo 1 2.0
bar 2 5.0
foo 3 8.0
bar 4 1.0
foo 5 2.0
bar 6 9.0
;
run;

/* 2. 计算每个 A 分组的 B 列平均值 */
proc means data=df noprint;
   class A; /* 按 A 分组 */
   var B;   /* 计算 B 的平均值 */
   output out=means_out mean(B)=B_mean; /* 输出平均值到新数据集 */
run;

/* 3. 筛选 B 平均值大于 3 的组 */
data filtered_groups;
   set means_out;
   where B_mean > 3;
   keep A; /* 只保留 A 列用于后续合并 */
run;

/* 4. 将原始数据集与筛选出的组合并，保留符合条件的行 */
proc sql;
   create table result as
   select df.*
   from df
   inner join filtered_groups
   on df.A = filtered_groups.A;
quit;

/* 5. 输出结果 */
proc print data=result;
run;
```

代码说明：创建数据集：data df; 创建一个数据集 df，包含列 A（字符型）、B（数值型）、C（数值型）。
datalines; 用于输入与 Pandas 示例相同的数据。

计算分组平均值：proc means 按 A 分组，计算列 B 的平均值。
noprint 避免打印冗长输出。
output out=means_out mean(B)=B_mean; 将每组的 B 平均值输出到新数据集 means_out。

筛选平均值大于 3 的组：data filtered_groups; 从 means_out 中筛选出 B_mean > 3 的组。
保留 A 列，用于后续合并。

合并数据：proc sql 使用 SQL 合并原始数据集 df 和筛选出的组 filtered_groups，只保留 A 值匹配的行（即 B 平均值大于 3 的组的行）。

输出结果：proc print 打印最终结果，显示与 Pandas 输出相同的行：

### 注意事项：
SAS 的 proc means 和 Pandas 的 groupby 功能类似，但 SAS 需要显式输出中间结果并进行合并。
proc sql 用于实现类似 Pandas filter 的功能，通过合并保留符合条件的行。
结果与 Pandas 代码完全一致，只保留 A='bar' 的行。

### Only in SAS
要用 SAS 实现与之前 Pandas 代码相同的效果（按列 A 分组，筛选列 B 平均值大于 3 的组的所有行），且不使用 PROC SQL，我们可以使用 SAS 的 DATA 步骤和 MERGE 语句来实现合并。

```js
/* 1. 创建数据集，相当于 Pandas 的 DataFrame */
data df;
   input A $ B C;
   datalines;
foo 1 2.0
bar 2 5.0
foo 3 8.0
bar 4 1.0
foo 5 2.0
bar 6 9.0
;
run;

/* 2. 计算每个 A 分组的 B 列平均值 */
proc means data=df noprint;
   class A; /* 按 A 分组 */
   var B;   /* 计算 B 的平均值 */
   output out=means_out mean(B)=B_mean; /* 输出平均值到新数据集 */
run;

/* 3. 筛选 B 平均值大于 3 的组 */
data filtered_groups;
   set means_out;
   where B_mean > 3;
   keep A; /* 只保留 A 列用于后续合并 */
run;

/* 4. 对原始数据集和筛选出的组进行排序以准备合并 */
proc sort data=df;
   by A;
run;

proc sort data=filtered_groups;
   by A;
run;

/* 5. 使用 MERGE 合并，保留符合条件的行 */
data result;
   merge df(in=a) filtered_groups(in=b);
   by A;
   if a and b; /* 仅保留 A 在 filtered_groups 中的行 */
run;

/* 6. 输出结果 */
proc print data=result;
run;
```

### 代码说明：
创建数据集：与之前相同，data df; 创建包含列 A（字符型）、B 和 C（数值型）的数据集，数据与 Pandas 示例一致。

计算分组平均值：proc means 按 A 分组，计算列 B 的平均值，输出到 means_out 数据集，包含 A 和 B_mean（B 的平均值）。

筛选平均值大于 3 的组：data filtered_groups; 从 means_out 中筛选出 B_mean > 3 的组，仅保留 A 列。

排序数据集：proc sort 对 df 和 filtered_groups 按 A 排序，因为 SAS 的 MERGE 语句要求数据集按合并键（A）排序。

合并数据：merge df(in=a) filtered_groups(in=b); 合并 df 和 filtered_groups，通过 by A 按 A 匹配。
if a and b; 确保只保留 A 在 filtered_groups 中出现的行（即 B 平均值大于 3 的组的行）。

输出结果：proc print 打印结果，输出与 Pandas 代码相同：

#### 与之前代码的区别：
用 proc sort 和 merge 替换了 proc sql，实现相同的合并逻辑。
in=a 和 in=b 用于跟踪哪些行来自 df 和 filtered_groups，if a and b 确保只保留匹配的行。
结果与 Pandas 和之前的 SAS 代码完全一致。

