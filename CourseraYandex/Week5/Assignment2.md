```
1. #! /usr/bin/env python

from pyspark import SparkConf, SparkContext
sc = SparkContext(conf=SparkConf().setAppName("MyApp").setMaster("local[2]"))

import re
import math

2. stop_file = "/datasets/stop_words_en.txt"
wiki_file = "/data/wiki/en_articles_part/articles-part"
pair_thresh = 500

with open(stop_file, "r") as f:
    stop_words = f.read().splitlines()
    
stop_words_bcast = sc.broadcast(stop_words)

def parse_article(line):
    try:
        article_id, text = line.rstrip().split('\t', 1)
        text = re.sub("^\W+|\W+$", "", text, flags=re.UNICODE)
        words = re.split("\W*\s+\W*", text, flags=re.UNICODE)
        return words
    except ValueError as e:
        return []
    
def lower(words):
    return [word.lower() for word in words]

def filter_stop(words):
    return [word for word in words if word not in stop_words_bcast.value]

def pairs(words):
    out = []
    for w1, w2 in zip(words, words[1:]):
        out.append((w1.lower() + "_" + w2.lower(), 1))
    return out

wiki = (sc.textFile(wiki_file, 16)
         .map(parse_article)  
         .map(lower)
         .map(filter_stop)
        ).cache()

3. words = (wiki.flatMap(lambda wds : [(word, 1) for word in wds])
         .reduceByKey(lambda x,y: x+y)
        ).cache()

words_total = words.map(lambda value: value[1]).sum()
words_total = sc.broadcast(words_total)

words_count_map = words.collectAsMap()
words_count_map = sc.broadcast(words_count_map)

pairs = (wiki.flatMap(pairs)
         .reduceByKey(lambda x,y : x+y)
        ).cache()

pairs_total = pairs.map(lambda value: value[1]).sum()
pairs_total = sc.broadcast(pairs_total)

4. def npmi(value):
    pair, count = value
    w1, w2 = pair.split("_")
    w1_count = words_count_map.value[w1]
    w2_count = words_count_map.value[w2]
    
    pair_prob = float(count) / pairs_total.value
    w1_prob = float(w1_count) / words_total.value
    w2_prob = float(w2_count) / words_total.value
    
    pmi = math.log(pair_prob / (w1_prob * w2_prob))
    npmi = pmi / (-1 * math.log(pair_prob))
    return (pair, npmi)

npmi = (pairs
        .filter(lambda value: value[1] > pair_thresh)
        .map(lambda value: npmi(value))
        .sortBy(lambda value: value[1], ascending=False)
       ).cache()

5. for pair, value in npmi.take(39):
    print(pair)