---
title: Kylin, Mondrian, Saiku系统的整合
date: 2016-04-19
tags: [总结, 有赞, kylin]
categories: 大数据
language: ZH
---

# Kylin, Mondrian, Saiku系统的整合

本文主要介绍有赞数据团队为了满足在不同维度查看、分析重点指标的需求而搭建的OLAP分析工具。这个工具对Kylin、Mondrian以及Saiku做了一个整合，主要工作包括一些定制化的修改以及环境的配置。
目前这个系统还处于一个需要优化、完善的过程，这篇博文也会相应地更新。

## 背景
在[有赞](https://www.youzan.com/)发展的初期，数据团队主要的工作之一就是根据运营人员的报表需求，编写sql，从hive中获得数据并写入mysql中存储。最后，前端人员写相应的代码展现mysql中存储的报表数据。
随着公司业务的快速发展，如此长周期的报表开发流程已经很难跟上运营人员的分析需求了。为了避免深陷报表开发、维护的泥潭，数据组决定调研大数据场景下的OLAP分析工具。参考了[明略数据的解决方案](http://www.slideshare.net/lukehan/5-apache-kylin-apache-kylin-meetup-shanghai)之后，我们选择整合[Kylin](http://kylin.apache.org/)，[Mondrian](http://community.pentaho.com/projects/mondrian/)，[Saiku](http://www.meteorite.bi/products/saiku)来实现这样一个OLAP系统。

<!-- more -->

##  三巨头
###  Kylin
kylin是apache软件基金会的顶级项目，一个开源的分布式多维分析工具。下面是摘自Kylin官网的介绍：
> Apache Kylin™ is an open source Distributed Analytics Engine designed to provide SQL interface and multi-dimensional analysis (OLAP) on Hadoop supporting extremely large datasets, original contributed from eBay Inc.

个人的理解是：Kylin通过预计算所有合理的维度组合下各个指标的值并把计算结果存储到HBASE中的方式，大大提高分布式多维分析的查询效率。Kylin接收sql查询语句作为输入，以查询结果作为输出。通过预计算的方式，将在hive中可能需要几分钟的查询响应时间下降到毫秒级。更细致的关于Kylin的介绍，可以参考我的另一片博客[Kylin初体验](./kylin.md)。

### Mondrian
> Mondrian is an Open Source Business Analytics engine that enables organizations of any size to give business users access to their data for interactive analysis. You can build powerful Business Intelligence solutions with Mondrian as your Online Analytical Processing (OLAP) engine, enabling multidimensional queries against your business data, using the powerful MDX query language.

Mondrian是一个OLAP分析的引擎，主要工作是根据事先配置好的schema，将输入的多维分析语句[MDX](https://msdn.microsoft.com/zh-cn/library/ms145506(v=sql.120).aspx)(Multidimensional Expressions )翻译成目标数据库／数据引擎的执行语言（比如SQL）。

### Saiku
> Saiku allows business users to explore complex data sources, using a familiar drag and drop interface and easy to understand business terminology, all within a browser. Select the data you are interested in, look at it from different perspectives, drill into the detail. Once you have your answer, save your results, share them, export them to Excel or PDF, all straight from the browser.

Saiku提供了一个多维分析的用户操作界面，可以通过简单拖拉拽的方式迅速生成报表。Saiku的主要工作是根据事先配置好的schema，将用户的操作转化成MDX语句提供给Mondrian引擎执行。

##  技术架构

Kylin + Mondrian + Saiku是一个简单的三层架构。git上开源的Saiku的项目已经整合了mondrian的jar包。所以构建这样一个三层架构主要的工作是将Mondrian的schema和Kylin的schema对应起来，同时需要针对Kylin的语法对Mondrian做一些Kylin dialect的定制开发。
Git上已经有一个[整合Kylin，Mondrian以及Saiku的项目](https://github.com/mustangore/kylin-mondrian-interaction)。照着这个项目的指引，可以很轻松的搭建这么一个三层的系统。在此，致谢开源项目作者[mustangore](https://github.com/mustangore)。

## 一些细节
介绍完整体的结构，下面讲一些构建过程中遇到的坑。有些可能是我们的理解还不够深入，有些可能随着开源软件版本的升级已经不再是一个坑了。希望能给大家带来一些帮助，如果是由于我们理解的偏差导致踩到的坑，也希望大家留言给出指正：）
本套系统构建基于kylin1.5， Mondrian4.4以及Saiku3.7.4。底层是Hive0.14以及Hbase0.98。

### 关于schema
前面提到，要让系统运转，Kylin的schema必须和mondrian的schema能够对接上。Kylin是根据自身cube配置的schema来进行预计算的，schema决定Kylin能够接收的sql查询的范围。Mondrian又根据自身的shema翻译MDX到sql， Mondrian的schema决定它生成的sql的范围。如果两者有不一致的情况，就可能导致Mondrian生成的sql无法被Kylin执行。
kylin的schema配置比较简单，管理页面上有一套图形界面指引你一步步地构建一个星型模型，配置di mension、measure。不过要把cube设计得高效，Kylin还是有不少高级地设置的，比如选择 attribute group， derived dimension等。官网上有详细的[介绍](http://kylin.apache.org/blog/2016/02/18/new-aggregation-group/)。
Mondrian的schema没有比较好的图形配置工具，需要手写Mondrian schema的XML文档，文档格式参考[官方文档](http://mondrian.pentaho.com/documentation/schema.php)，通过Saiku上传。
需要注意的坑：

 - **不要用view作为lookup table** 在设计Kylin cube时，用hive view作为fact table是一个比较好的实践方式，可以屏蔽一些底层数据结构变化对Kylin cube的影响。但是不要用view作look up table，在build cube计算维度表容量时会出问题。
 - **Kylin无法在预计算指标时制定条件** 比如有两个字段：order_pay, is_payed。我们可以配置sum(order_pay)作为订单金额, sum(is_payed)作为付款订单数。但是没法配置sum(order_pay) where is_payed = 1来表示付款订单金额。我们需要在fact view中添加字段payed_order_pay表示付款的订单金额。
 - **尽量在Kylin中用int类型** 比如is_payed字段，就0/1两个值，通常我们在hive里可以设置为tiny int类型的字段。但是在Kylin中，针对tiny int 和 int类型的字段配置出来的measure类型是不一样的，tiny int 类型的字段得到的measure在和Saiku结合时可能会出现问题。
 - **把hive表放在default库中** Kylin添加hive table的时候是可以指定hive table所在库的，但是建议将fact table、lookup table都放在default库中。因为在Mondrian的schema中，physical table是默认去default库查找的，目前还没有发现很好的在Mondrian schema中指定数据库的方式。

### 关于count distinct
Kylin配置cube的时候可以指定某个measure的聚合方式为count distinct，有精准计算的方式也有基于hyperloglog算法的近似计算方式。同样，在Mondrian的schema里也可以配置count distinct的指标聚合方式。
看上去一切都OK，然而问题来了：
Kylin的count distinct语法只针用count distinct聚合的指标字段，在计算维度表大小的时候，kylin无法计算类似 ```select count(distinct date) from lu_date```这样的sql语句。在[mustangore的项目](https://github.com/mustangore/kylin-mondrian-interaction)中，对Mondrian打了Kylin-dialect的补丁。其中添加了一个JdbcDialect的实现：
<pre><code>
public class KylinDialect extends JdbcDialectImpl {

    public static final JdbcDialectFactory FACTORY =
            new JdbcDialectFactory(KylinDialect.class, DatabaseProduct.KYLIN) {
                protected boolean acceptsConnection(Connection connection) {
                    return super.acceptsConnection(connection);
                }
            };

    /**
     * Creates a KylinDialect.
     *
     * @param connection Connection
     * @throws SQLException on error
     */
    public KylinDialect(Connection connection) throws SQLException {
        super(connection);
    }

    @Override
    public boolean allowsCountDistinct() {
        return false;
    }

    @Override
    public boolean allowsJoinOn() {
        return true;
    }
}
</code></pre>

注意到：allowsCountDistinct()函数被设置成了return false；
mustangore 通过这种方式避免了Mondrian计算维度大小的时候count disctinct，然而这种一杆子打死的方式也使得Mondrian计算count distinct的指标的时候出现问题：```select count(distinct XXX) from tableA```这样的语句会被翻译成```select count YYY from (select distinct XXX as YYY from tableA)```，而Kylin又不能很好的执行后者。为了解决这个两难的问题，我们深入到Mondrian的源码中去，找到了计算维度表大小的代码：
<pre><code>
 private static String generateColumnCardinalitySql(
        Dialect dialect,
        String schema,
        String table,
        String column)
    {
        final StringBuilder buf = new StringBuilder();
        String exprString = dialect.quoteIdentifier(column);
        if (dialect.allowsCountDistinct()) {
            // e.g. "select count(distinct product_id) from product"
            buf.append("select count(distinct ")
                .append(exprString)
                .append(") from ");
            dialect.quoteIdentifier(buf, schema, table);
            return buf.toString();
        }
        else if (dialect.allowsFromQuery()) {
            // Some databases (e.g. Access) don't like 'count(distinct)',
            // so use, e.g., "select count(*) from (select distinct
            // product_id from product)"
            buf.append("select count(*) from (select distinct ")
                .append(exprString)
                .append(" from ");
            dialect.quoteIdentifier(buf, schema, table);
            buf.append(")");
            ...
</code></pre>
注意到只有dialect.allowsCountDistinct()为true时才会用count distinct来计算维度表大小。
我们只要将Kylin dialect的allowsCountDistinct()设置为true，同时在generateColumnCardinalitySql添加一个判断条件：
<pre><code>
  if (dialect.allowsCountDistinct()
            && !dialect.getDatabaseProduct().name().equalsIgnoreCase("KYLIN")) {
            ...
</code></pre>
就可以实现和kylin的count distcint measure的正常对接了。

### 关于Kylin sql
有了处理count distinct的问题的经验，我们发现，只要了解Kylin sql的特点，针对Kylin sql定制Mondrian 的Kylin—diect就能将Mondrian和kylin较好的对接。经过在Kylin1.5的交互界面中的测试，我们列出如下的区别：

 - 不能limit beg, end 只能limit length
 - 不支持 union, union all
 - 不支持 where exists 子句

## 结束语
以上是有赞数据团队实现多维分析工具的探索过程。总的来说，Kylin ＋ Saiku ＋ Mondrian的一套流程是能走通的，中途遇到一些零碎的问题没有完全列出来。通常是因为Kylin只支持cube范围内的查询，如果Mondrian翻译出的sql超出这个范围就会引起系统的错误。通常有三种解决方案：
 - 重新构建Kylin cube，让它能覆盖更广范围的查询
 - 修改Mondrian schema，让它的cube描述和Kylin cube吻合
 - 定制化开发Mondrian的Kylin dialect，让Mondrian生成符合Kylin特点的sql

目前我们还在对这套三层框架做一些定制化的功能开发以及性能的调优工作。希望这篇文章能给大家带来些帮助，也希望有独特见解或者发现我们的理解不对的朋友们留言交流。
