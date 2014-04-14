---
layout: post
title: "Hadoop Combine"
date: 2014-03-17 10:28
comments: true
categories: hadoop
---
之前在面试的时候，有人突然问到了一个问题：Combine 是做什么的？

引用官方的原话[原话](http://wiki.apache.org/hadoop/HadoopMapReduce "Title")

When the map operation outputs its pairs they are already available in memory. For efficiency reasons, sometimes it makes sense to take advantage of this fact by supplying a combiner class to perform a reduce-type function. If a combiner is used then the map key-value pairs are not immediately written to the output. Instead they will be collected in lists, one list per each key value. When a certain number of key-value pairs have been written, this buffer is flushed by passing all the values of each key to the combiner's reduce method and outputting the key-value pairs of the combine operation as if they were created by the original map operation.

For example, a word count MapReduce application whose map operation outputs (word, 1) pairs as words are encountered in the input can use a combiner to speed up processing. A combine operation will start gathering the output in in-memory lists (instead of on disk), one list per word. Once a certain number of pairs is output, the combine operation will be called once per unique word with the list available as an iterator. The combiner then emits (word, count-in-this-part-of-the-input) pairs. From the viewpoint of the Reduce operation this contains the same information as the original Map output, but there should be far fewer pairs output to disk and read from disk.

简单的说，就是在 map 之后，但是数据还没到 reduce 之前，调用一个 reduce-type function 做一些操作，提高性能。