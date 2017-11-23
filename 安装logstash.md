# 官网
https://www.elastic.co/downloads/logstash

# 安装
```
# cd /usr/local

# tar -zxvf logstash-6.0.0.tar.gz

# mv logstash-6.0.0 logstash

# vim /logstash/config/logstash.conf
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

# 启动
```
# cd logstash
# ./bin/logstash -f config/logstash.conf
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Sending Logstash's logs to /usr/local/logstash/logs which is now configured via log4j2.properties
[2017-11-23T17:07:33,555][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"/usr/local/logstash/modules/fb_apache/configuration"}
[2017-11-23T17:07:33,598][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"/usr/local/logstash/modules/netflow/configuration"}
[2017-11-23T17:07:34,245][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2017-11-23T17:07:35,062][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
[2017-11-23T17:07:39,277][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://localhost:9200/]}}
[2017-11-23T17:07:39,299][INFO ][logstash.outputs.elasticsearch] Running health check to see if an Elasticsearch connection is working {:healthcheck_url=>http://localhost:9200/, :path=>"/"}
[2017-11-23T17:07:39,536][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://localhost:9200/"}
[2017-11-23T17:07:39,642][INFO ][logstash.outputs.elasticsearch] Using mapping template from {:path=>nil}
[2017-11-23T17:07:39,657][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>60001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date"}, "@version"=>{"type"=>"keyword"}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2017-11-23T17:07:39,684][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["//localhost:9200"]}
[2017-11-23T17:07:39,687][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>1, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>125, :thread=>"#<Thread:0x3534496e@/usr/local/logstash/logstash-core/lib/logstash/pipeline.rb:290 run>"}
The stdin plugin is now waiting for input:
[2017-11-23T17:07:39,818][INFO ][logstash.pipeline        ] Pipeline started {"pipeline.id"=>"main"}
[2017-11-23T17:07:39,827][INFO ][logstash.agent           ] Pipelines running {:count=>1, :pipelines=>["main"]}
```
