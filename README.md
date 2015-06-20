# Logstash Input Plugin for Informatica log files

The goal was, starting with the standard Logstash input template, create a plugin to parse the binary Informatica log files, but it didn't get very far.

I may or may not get back to it, but the following config file works fine on the exported text log file.

####logstash_informatica_text.conf

````
input {
	file {
		path => "/path-to-informatica-exported-log_file/*.txt"
		type => "informatica_text"
		start_position => "beginning"
		codec => multiline {
			pattern => "^%{TIMESTAMP_ISO8601} "
    		negate => true
    		what => previous
		}
	}
}

filter {
	if [type] == 'informatica_text' {
		grok {
			match => { "message" =>  "%{TIMESTAMP_ISO8601:ts} : %{DATA:severity} : \(%{INT:pip} \| %{DATA:thread}\) : \(%{DATA:serviceType} \| %{DATA:serviceName}\) : %{DATA:clientNode} : %{DATA:messageCode} : %{GREEDYDATA:msg}"}
		}
		date {
			match => [ "ts", "YYYY-MM-dd HH:mm:ss" ]
			target => "@timestamp"
			remove_field => [ "ts" ]
		}
		mutate {
           	remove_field => ["message", "tags" ]
		}
	}
}

output {
	stdout {
		codec => rubydebug
	}
	elasticsearch {
	    host => "localhost"
	}
}
````


logstash -f logstash_informatica_text.conf