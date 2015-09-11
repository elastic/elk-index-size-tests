#elk-index-size-tests

Supporting files for testing Elasticsearch index sizes. You can find more details on this blog post: [https://www.elastic.co/blog/elasticsearch-storage-the-true-story](https://www.elastic.co/blog/elasticsearch-storage-the-true-story).


##Testing steps

* Ingest the log file using Logstash with a simple config

		# Uncompress logs.gz and logs2.gz
		gzip -d logs.gz logs2.gz
		# Symlink index_template.json to index_template_n.json where n is current iteration of test
		ln -sf index_template_n.json index_template.json
		# For ingesting mostly structured log file:
		bin/logstash -f complete.conf < logs
		# For ingesting semi-structured log file containing more text:
		bin/logstash -f complete2.conf < logs2
		# For ingesting mostly structured log file, retaining original event:
		bin/logstash -f complete3.conf < logs
		# For ingesting semi-structured log file containing more text, retaining original event:
		bin/logstash -f complete4.conf < logs2

* Optimize the index to 1 segment (for a consistently comparable size) by calling POST elk_workshop/_optimize?max_num_segments=1

		curl -XPOST 'http://localhost:9200/elk_workshop/_optimize?max_num_segments=1'

* Get the index size on disk by calling GET elk_workshop/_stats

		curl -XGET 'http://localhost:9200/elk_workshop/_stats?pretty=true&filter_path=indices.elk_workshop.primaries.store.size_in_bytes'

* Remove the index by calling DELETE elk_workshop

		curl -XDELETE 'http://localhost:9200/elk_workshop'

##Config file descriptions

###Logstash configs

File|Description
-------------|-------------
complete.conf|Removes the original message
complete2.conf|Adds a randomized unstructured text element to end of log line
complete3.conf|Retains the original message
complete4.conf|Adds a randomized unstructured text element to end of log line, retains the original message

###Elasticsearch index templates

File|Description
-------------|-------------
index_template_1.json|All string fields are indexed in both analyzed and not_analyzed form; _all is enabled
index_template_2.json|All string fields are indexed in both analyzed and not_analyzed form; _all is disabled
index_template_3.json|All string fields are indexed in not_analyzed form; _all is disabled
index_template_3b.json|All string fields are indexed in not_analyzed form except for message field which is analyzed; _all is disabled
index_template_4.json|All string fields are indexed in not_analyzed form except for agent field which is analyzed; _all is disabled


#Acknowledgements

Thanks to my colleague Christian Dahlqvist (@cdahlqvist) for adding single-shard tests and semi-structured test logs.
