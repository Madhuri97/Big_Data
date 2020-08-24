```1. %%writefile mapper.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

for line in sys.stdin:
    try:
        article_id, text = unicode(line.strip()).split('\t', 1)
    except ValueError as e:
        continue
    words = re.split("\W*\s+\W*", text, flags=re.UNICODE)
    for word in words:
        print >> sys.stderr, "reporter:counter:Wiki stats,Total words,%d" % 1
        print "%s\t%d" % (word.lower(), 1)

Overwriting mapper.py

2. %%writefile reducer.py
#!/usr/bin/python
import sys
current_key = None
word_sum = 0

Overwriting reducer.py

3. %%writefile -a reducer.py

for line in sys.stdin:
    try:
        key, count = line.strip().split('\t', 1)
        count = int(count)
    except ValueError as e:
        continue
    if current_key != key:
        if current_key:
            print "%s\t%d" % (current_key, word_sum)
        word_sum = 0
        current_key = key
    word_sum += count

if current_key:
    print "%s\t%d" % (current_key, word_sum)

Appending to reducer.py

4. %%writefile mapper_rating.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

for line in sys.stdin:
    try:
        word, count = line.strip().split('\t', 1)
        count = int(count)
    except ValueError as e:
        continue
        
    print "%d\t%s" % (count, word)

Overwriting mapper_rating.py

5. %%writefile reducer_rating.py
#!/usr/bin/python

import sys

for line in sys.stdin:
    try:
        count, word = line.strip().split('\t', 1)
        count = int(count)
    except ValueError as e:
        continue
        
    print "%s\t%d" % (word, count)

Overwriting reducer_rating.py

6. %%bash

OUT_DIR="wordcount_result_for_rating"
NUM_REDUCERS=8

hdfs dfs -rm -r -skipTrash ${OUT_DIR} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming wordCount" \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper.py,reducer.py \
    -mapper "python2 mapper.py" \
    -combiner "python2 reducer.py" \
    -reducer "python2 reducer.py" \
    -input /data/wiki/en_articles_part \
    -output ${OUT_DIR} > /dev/null


OUT_DIR_2="wordcount_result_for_rating_2"
NUM_REDUCERS=1

hdfs dfs -rm -r -skipTrash ${OUT_DIR_2} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming wordCount Rating" \
    -D mapreduce.job.output.key.comparator.class=org.apache.hadoop.mapreduce.lib.partition.KeyFieldBasedComparator \
    -D map.output.key.field.separator=\t \
    -D mapreduce.partition.keycomparator.options=-k1,1nr \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper_rating.py,reducer_rating.py \
    -mapper "python2 mapper_rating.py" \
    -reducer "python2 reducer_rating.py" \
    -input ${OUT_DIR} \
    -output ${OUT_DIR_2} > /dev/null

hdfs dfs -cat ${OUT_DIR_2}/part-00000 | head -7 | tail -1
