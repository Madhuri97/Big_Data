```
1. #! /usr/bin/env python

from pyspark import SparkConf, SparkContext
sc = SparkContext(conf=SparkConf().setAppName("MyApp").setMaster("local[2]"))

import re

def parse_article(line):
    try:
        article_id, text = line.rstrip().split('\t', 1)
        text = re.sub("^\W+|\W+$", "", text, flags=re.UNICODE)
        words = re.split("\W*\s+\W*", text, flags=re.UNICODE)
        return words
    except ValueError as e:
        return []
    
def pairs(words):
    out = []
    for w1, w2 in zip(words, words[1:]):
        out.append((w1.lower() + "_" + w2.lower(), 1))
    return out
    
result = (sc.textFile("/data/wiki/en_articles_part/articles-part", 16)
        .map(parse_article)
        .flatMap(pairs)
        .reduceByKey(lambda x,y : x+y)
        .sortByKey()
        .filter(lambda value: value[0][:9] == "narodnaya")
       ).collect()

for key, count in result:
    print("%s\t%d" % (key, count))
