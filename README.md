#elk-index-size-tests

Supporting files for testing Elasticsearch index sizes. You can find more details on this blog post: [http://peter.mistermoo.com/2015/01/05/hardware-sizing-or-how-many-servers-do-i-really-need/](http://peter.mistermoo.com/2015/01/05/hardware-sizing-or-how-many-servers-do-i-really-need/).


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
		
* Optimize the index to 1 segment (for a consistently comparable size) by calling POST elk_workshop/_optimize?max_num_segments=1

		curl -XPOST http://localhost:9200/elk_workshop/_optimize?max_num_segments=1
		
* Get the index size on disk by calling GET elk_workshop/_stats

		curl -XGET http://localhost:9200/elk_workshop/_stats?pretty
		
* Remove the index by calling DELETE elk_workshop

		curl -XDELETE http://localhost:9200/elk_workshop
		

