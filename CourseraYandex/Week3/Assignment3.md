```
1. %%writefile mapper.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

def is_name(word):
    if len(word) < 2:
        return False
    elif (word[0].isalpha()) and (word[0].isupper()) and (word[1:].islower()):
        return True
    else:
        return False

for line in sys.stdin:
    try:
        article_id, text = unicode(line.strip()).split('\t', 1)
    except ValueError as e:
        continue
    words = re.split("\W*\s+\W*", text, flags=re.UNICODE)
    for word in words:
        name_flag = int(is_name(word))
        print "%s\t%d\t%d" % (word.lower(), 1, name_flag)

2. %%writefile reducer.py
#!/usr/bin/python

import sys
import re

reload(sys)
sys.setdefaultencoding('utf-8') # required to convert to unicode

def condition_name(cnt, cnt_name):
    if (cnt - cnt_name) / float(cnt) * 100 < 0.5:
        return True
    else:
        return False

current_key = None
current_cnt = 0
current_cnt_name = 0

for line in sys.stdin:
    try:
        key, cnt, cnt_name = unicode(line.strip()).split('\t')
        cnt = int(cnt)
        cnt_name = int(cnt_name)
    except ValueError as e:
        continue
    
    if current_key != key:
        if current_key and condition_name(current_cnt, current_cnt_name):
            print "%s\t%d" % (current_key, current_cnt)
        current_key = key
        current_cnt = cnt
        current_cnt_name = cnt_name
    else:
        current_cnt += cnt
        current_cnt_name += cnt_name
        
print "%s\t%d" % (current_key, current_cnt)

3. %%writefile mapper_2.py
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

4. %%writefile reducer_2.py
#!/usr/bin/python

import sys

for line in sys.stdin:
    try:
        count, word = line.strip().split('\t', 1)
        count = int(count)
    except ValueError as e:
        continue
        
    print "%s\t%d" % (word, count)

5. %%bash

OUT_DIR="name_count"
NUM_REDUCERS=8

hdfs dfs -rm -r -skipTrash ${OUT_DIR} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming name count" \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper.py,reducer.py \
    -mapper "python2 mapper.py" \
    -reducer "python2 reducer.py" \
    -input /data/wiki/en_articles_part \
    -output ${OUT_DIR} > /dev/null
    
OUT_DIR_2="name_count_sorted"
NUM_REDUCERS=1

hdfs dfs -rm -r -skipTrash ${OUT_DIR_2} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming name count sorting" \
    -D mapreduce.job.output.key.comparator.class=org.apache.hadoop.mapreduce.lib.partition.KeyFieldBasedComparator \
    -D map.output.key.field.separator=\t \
    -D mapreduce.partition.keycomparator.options=-k1,1nr \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper_2.py,reducer_2.py \
    -mapper "python2 mapper_2.py" \
    -reducer "python2 reducer_2.py" \
    -input ${OUT_DIR} \
    -output ${OUT_DIR_2} > /dev/null
    
hdfs dfs -cat ${OUT_DIR_2}/part-00000 | head -5 | tail -1