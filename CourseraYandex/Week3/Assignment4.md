```
1. %%writefile mapper.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

with open('stop_words_en.txt') as f:
    stop_words = set(f.read().split())

for line in sys.stdin:
    try:
        article_id, text = unicode(line.strip()).split('\t', 1)
    except ValueError as e:
        continue
    words = re.split("\W*\s+\W*", text, flags=re.UNICODE)
    for word in words:
        word = word.lower()
        if word in stop_words:
            continue
        word_sorted = ''.join(sorted(word))
        print "%s\t%d\t%s" % (word_sorted, 1, word)

2. %%writefile reducer.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

current_key = None
current_cnt = 0
words_set = set()

for line in sys.stdin:
    try:
        key, cnt, word = unicode(line.strip()).split('\t')
        cnt = int(cnt)
    except ValueError as e:
        continue
    
    if current_key != key:
        if current_key and (len(words_set) > 1):
            print "%d\t%d\t%s" % (current_cnt, len(words_set), ','.join(sorted(words_set)))
        current_key = key
        words_set = set()
        words_set.add(word)
        current_cnt = cnt
    else:
        words_set.add(word)
        current_cnt += cnt
        
print "%d\t%d\t%s" % (current_cnt, len(words_set), ','.join(sorted(words_set)))

3. %%bash

OUT_DIR="word_groups"
NUM_REDUCERS=8

hdfs dfs -rm -r -skipTrash ${OUT_DIR} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming word groups" \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper.py,reducer.py,/datasets/stop_words_en.txt \
    -mapper "python2 mapper.py" \
    -reducer "python2 reducer.py" \
    -input /data/wiki/en_articles_part \
    -output ${OUT_DIR} > /dev/null
    
hdfs dfs -cat word_groups/* | grep -P '(,|\t)english($|,)'