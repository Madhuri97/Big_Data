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
        print >> sys.stderr, "reporter:counter:wiki,total_words,%d" % 1
        if word in stop_words:
            print >> sys.stderr, "reporter:counter:wiki,stop_words,%d" % 1
        print "%s\t%d" % (word.lower(), 1)

2. %%writefile get_stopword_percentage.py
#!/usr/bin/python

import sys
import re

output_log = list(map(lambda x: x.strip(), sys.stdin.read().split()))

pattern_tot = 'total_words='
regexp_tot = re.compile(pattern_tot)

pattern_stop = 'stop_words='
regexp_stop = re.compile(pattern_stop)

total_words = [int(x.replace(pattern_tot, '')) for x in output_log if regexp_tot.search(x)][0]
stop_words = [int(x.replace(pattern_stop, '')) for x in output_log if regexp_stop.search(x)][0]

print(stop_words / float(total_words) * 100)

3. %%bash

OUT_DIR="wordcount_result_stopwords"
NUM_REDUCERS=0

hdfs dfs -rm -r -skipTrash ${OUT_DIR} > /dev/null

yarn jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -D mapred.jab.name="Streaming stopwords" \
    -D mapreduce.job.reduces=${NUM_REDUCERS} \
    -files mapper.py,/datasets/stop_words_en.txt \
    -mapper "python2 mapper.py" \
    -input /data/wiki/en_articles_part \
    -output ${OUT_DIR} > /dev/null 2> output.log

4. %%bash

cat output.log | egrep "*_words" | python get_stopword_percentage.py
cat output.log >&2