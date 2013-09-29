---
layout: post
title: "sunspot 使用 jboss_solr 来代替 sunspot_solr 完成网站搜索"
date: 2012-09-19 20:16
comments: true
categories: ruby
---
###sunspot
>使用 sunspot + sunspot_solr 非常简单方便的就能完成对网站的搜索，用起来非常简单，插件打包好了一切。让你看不到里面的实现。

>sunspot_solr 非常好用是必须要肯定的，但是就是建立索引非常慢，其实这个插件就是模拟了一个 jboss solr，那为什么不直接用这个 java 的 solr 呢？公司其他部门已经搭建完成了一个 solr 搜索引擎了，所以我只要远程调用搜索就 ok。

###1.部署一个 jboss solr 环境，[网址](http://www.jboss.org/)。
>这里就不详细介绍了。

###2.增加 sunspot gem 包
{% highlight ruby %}
gem 'sunspot_rails'
gem 'sunspot'
{% endhighlight %}

Bundle it!
    bundle install

Generate a default configuration file:
    rails generate sunspot_rails:install

###3.增加 searchable block 到你希望索引的 model 里面。
{% highlight ruby %}
class Post < ActiveRecord::Base
  searchable do
    text :title
    test:body
    integer :author_id
    integer :category_ids, :multiple => true
    time :published_at
  end
end
{% endhighlight %}

###4.配置 solr 的 data-config.xml 文件
{% highlight xml%}
<dataConfig> 
  <dataSource driver="org.postgresql.Driver"  url="jdbc:postgresql://192.168.5.13/cbb_development:5432"  user="cbb" password="bbc" transactionIsolation="TRANSACTION_READ_COMMITTED" holdability="CLOSE_CURSORS_AT_COMMIT"/> 

  <document>
    <entity name="posts" pk='id'
            transformer="RegexTransformer"
            query="select 'Product'||' '||id as class_id, 'Product' as class_name,  name,language,platform,description,author,developer,add_cbb_time,maintainer from posts">
      <field column='class_id' name='id' />
      <field column='class_name' name='type' />
      <field column='name' name='name_ss' />
      <field column='language' name='language_ss' />
      <field column='platform' name='platform_ss' />
      <field column='author' name='author_ss' />
      <field column='developer' name='developer_ss' />
      <field column='maintainer' name='maintainer' />
    </entity>

  </document>
</dataConfig> 
{% endhighlight %}
>简单介绍下这里面的配置:

>dataSource 就是配置数据库，solr 可以查询到的数据源。

>document 一个 core 里面只能配置一个 document。

>entity 实体，对应的就是数据库的表或者视图，但是这里面有几个规则，首先就是数据库的 id 要重新加入 Model 名字在前面，和 id 一起变成 entity 的唯一标识。还要设置一个 type 用于区别不同的 Model，也就是区分 entity 用的。

>field 就是把数据库中的列对应的变成 solr 里面的 field 的概念，按照我的理解，field 其实也是列的意思。

###5.配置 schema.xml 文件
{% highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>

<schema name="sunspot" version="1.0">
  <types>
    <!-- field type definitions. The "name" attribute is
       just a label to be used by field definitions.  The "class"
       attribute and any other attributes determine the real
       behavior of the fieldType.
         Class names starting with "solr" refer to java classes in the
       org.apache.solr.analysis package.
    -->
    <!-- *** This fieldType is used by Sunspot! *** -->
    <fieldType name="string" class="solr.StrField" omitNorms="true"/>
    <fieldType name="tdouble" class="solr.TrieDoubleField" omitNorms="true"/>
    <fieldType name="rand" class="solr.RandomSortField" omitNorms="true"/>
    <fieldType name="text" class="solr.TextField" omitNorms="false">
      <analyzer type='index'>
          <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="false"/>
        <tokenizer class="org.wltea.analyzer.solr.IKTokenizerFactory" isMaxWordLength="false"/>
    <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type='query'>
          <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="false"/>
        <tokenizer class="org.wltea.analyzer.solr.IKTokenizerFactory" isMaxWordLength="true"/>
    <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>
    <fieldType name="boolean" class="solr.BoolField" omitNorms="true"/>
    <fieldType name="date" class="solr.DateField" omitNorms="true"/>
    <fieldType name="sdouble" class="solr.SortableDoubleField" omitNorms="true"/>
    <fieldType name="sint" class="solr.SortableIntField" omitNorms="true"/>
    <fieldType name="slong" class="solr.SortableLongField" omitNorms="true"/>
    <fieldType name="tint" class="solr.TrieIntField" omitNorms="true"/>
    <fieldType name="tfloat" class="solr.TrieFloatField" omitNorms="true"/>
    <fieldType name="tdate" class="solr.TrieDateField" omitNorms="true"/>
  </types>

  <fields>
    <!-- Valid attributes for fields:
     name: mandatory - the name for the field
     type: mandatory - the name of a previously defined type from the 
       <types> section
     indexed: true if this field should be indexed (searchable or sortable)
     stored: true if this field should be retrievable
     compressed: [false] if this field should be stored using gzip compression
       (this will only apply if the field type is compressable; among
       the standard field types, only TextField and StrField are)
     multiValued: true if this field may contain multiple values per document
     omitNorms: (expert) set to true to omit the norms associated with
       this field (this disables length normalization and index-time
       boosting for the field, and saves some memory).  Only full-text
       fields or fields that need an index-time boost need norms.
     termVectors: [false] set to true to store the term vector for a
       given field.
       When using MoreLikeThis, fields used for similarity should be
       stored for best performance.
     termPositions: Store position information with the term vector.  
       This will increase storage costs.
     termOffsets: Store offset information with the term vector. This 
       will increase storage costs.
     default: a value that should be used if no value is specified
       when adding a document.
   -->
    <field name="text" stored="true" type="string" multiValued="true" indexed="true"/>

    <field name="id" stored="true" type="string" multiValued="false" indexed="true"/>
 
    <field name="type" stored="false" type="string" multiValued="true" indexed="true"/>

    <field name="class_name" stored="false" type="string" multiValued="false" indexed="true"/>

    <field name="lat" stored="true" type="tdouble" multiValued="false" indexed="true"/>

    <field name="lng" stored="true" type="tdouble" multiValued="false" indexed="true"/>

    <!-- *** This dynamicField is used by Sunspot! *** -->
    <dynamicField name="random_*" stored="false" type="rand" multiValued="false" indexed="true"/>
    <dynamicField name="_local*" stored="false" type="tdouble" multiValued="false" indexed="true"/>
    <dynamicField name="*_text" stored="false" type="text" multiValued="true" indexed="true"/>
    <dynamicField name="*_texts" stored="true" type="text" multiValued="true" indexed="true"/>
    <dynamicField name="*_b" stored="false" type="boolean" multiValued="false" indexed="true"/>
    <dynamicField name="*_bm" stored="false" type="boolean" multiValued="true" indexed="true"/>
    <dynamicField name="*_bs" stored="true" type="boolean" multiValued="false" indexed="true"/>
    <dynamicField name="*_bms" stored="true" type="boolean" multiValued="true" indexed="true"/>
    <dynamicField name="*_d" stored="false" type="date" multiValued="false" indexed="true"/>
    <dynamicField name="*_dm" stored="false" type="date" multiValued="true" indexed="true"/>
    <dynamicField name="*_ds" stored="true" type="date" multiValued="false" indexed="true"/>
    <dynamicField name="*_dms" stored="true" type="date" multiValued="true" indexed="true"/>
    <dynamicField name="*_e" stored="false" type="sdouble" multiValued="false" indexed="true"/>
    <dynamicField name="*_em" stored="false" type="sdouble" multiValued="true" indexed="true"/>
    <dynamicField name="*_es" stored="true" type="sdouble" multiValued="false" indexed="true"/>
    <dynamicField name="*_ems" stored="true" type="sdouble" multiValued="true" indexed="true"/>
    <dynamicField name="*_f" stored="false" type="sfloat" multiValued="false" indexed="true"/>
    <dynamicField name="*_fm" stored="false" type="sfloat" multiValued="true" indexed="true"/>
    <dynamicField name="*_fs" stored="true" type="sfloat" multiValued="false" indexed="true"/>
    <dynamicField name="*_fms" stored="true" type="sfloat" multiValued="true" indexed="true"/>
    <dynamicField name="*_i" stored="false" type="sint" multiValued="false" indexed="true"/>
    <dynamicField name="*_im" stored="false" type="sint" multiValued="true" indexed="true"/>
    <dynamicField name="*_is" stored="true" type="sint" multiValued="false" indexed="true"/>
    <dynamicField name="*_ims" stored="true" type="sint" multiValued="true" indexed="true"/>
    <dynamicField name="*_l" stored="false" type="slong" multiValued="false" indexed="true"/>
    <dynamicField name="*_lm" stored="false" type="slong" multiValued="true" indexed="true"/>
    <dynamicField name="*_ls" stored="true" type="slong" multiValued="false" indexed="true"/>
    <dynamicField name="*_lms" stored="true" type="slong" multiValued="true" indexed="true"/>
    <dynamicField name="*_s" stored="false" type="string" multiValued="false" indexed="true"/>
    <dynamicField name="*_sm" stored="false" type="string" multiValued="true" indexed="true"/>
    <dynamicField name="*_ss" stored="true" type="string" multiValued="false" indexed="true"/>
    <dynamicField name="*_sms" stored="true" type="string" multiValued="true" indexed="true"/>
    <dynamicField name="*_it" stored="false" type="tint" multiValued="false" indexed="true"/>
    <dynamicField name="*_itm" stored="false" type="tint" multiValued="true" indexed="true"/>
    <dynamicField name="*_its" stored="true" type="tint" multiValued="false" indexed="true"/>
    <dynamicField name="*_itms" stored="true" type="tint" multiValued="true" indexed="true"/>
    <dynamicField name="*_ft" stored="false" type="tfloat" multiValued="false" indexed="true"/>
    <dynamicField name="*_ftm" stored="false" type="tfloat" multiValued="true" indexed="true"/>
    <dynamicField name="*_fts" stored="true" type="tfloat" multiValued="false" indexed="true"/>
    <dynamicField name="*_ftms" stored="true" type="tfloat" multiValued="true" indexed="true"/>
    <dynamicField name="*_dt" stored="false" type="tdate" multiValued="false" indexed="true"/>
    <dynamicField name="*_dtm" stored="false" type="tdate" multiValued="true" indexed="true"/>
    <dynamicField name="*_dts" stored="true" type="tdate" multiValued="false" indexed="true"/>
    <dynamicField name="*_dtms" stored="true" type="tdate" multiValued="true" indexed="true"/>
    <dynamicField name="*_textv" stored="false" termVectors="true" type="text" multiValued="true" indexed="true"/>
    <dynamicField name="*_textsv" stored="true" termVectors="true" type="text" multiValued="true" indexed="true"/>
    <dynamicField name="*_et" stored="false" termVectors="true" type="tdouble" multiValued="false" indexed="true"/>
    <dynamicField name="*_etm" stored="false" termVectors="true" type="tdouble" multiValued="true" indexed="true"/>
    <dynamicField name="*_ets" stored="true" termVectors="true" type="tdouble" multiValued="false" indexed="true"/>
    <dynamicField name="*_etms" stored="true" termVectors="true" type="tdouble" multiValued="true" indexed="true"/>
  </fields>
  <!-- Field to use to determine and enforce document uniqueness. 
      Unless this field is marked with required="false", it will be a required field
   -->
  <uniqueKey>id</uniqueKey>
  <!-- field for the QueryParser to use when an explicit fieldname is absent -->
  <defaultSearchField>text</defaultSearchField>
  <!-- SolrQueryParser configuration: defaultOperator="AND|OR" -->
  <solrQueryParser defaultOperator="AND"/>
  <!-- copyField commands copy one field to another at the time a document
        is added to the index.  It's used either to index the same field differently,
        or to add multiple fields to the same field for easier/faster searching.  -->
</schema>

{% endhighlight %}

>fieldType 是用来定义 solr 里面的基本数据类型，这些基本数据类型有什么功能，比如是字符串，还是整型，还是 text 可以用来切词 等等

>field 这里的 5 个 field 是 sunspot 用来区分不同 Model 的，而且是必要的。

>dynamicField 这里的动态字段，是 sunspot 的规则进行查询的，在 data-config.xml 里面，所有从数据库里面到 solr 里面的 field 后面都加了一个后缀。比如一个字符串，并且 solr 存储它，那就写 column_ss 第一个 s 就是 string 的意思，第二个 s 是就是 stored 的意思。同理如果是 text 并且存储，就是 column_tests。
这些都是 sunspot 自动生成的，不用多少改动。

###6.搜索
{% highlight ruby %}
result = Sunspot.search(Model) do
  fulltext 'best pizza'
  with :blog_id, 1
  order_by :created_at, :desc
  paginate :page => 2, :per_page => 15
end
{% endhighlight%}
>具体实现去看 [sunspot](http://sunspot.github.com/)

> sunspot [API](http://sunspot.github.com/docs/index.html)