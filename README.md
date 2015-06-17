# Logstash Input Plugin for Informatica log files

The goal was, starting with the standard Logstash input template, create a plugin to parse the binary Informatica log files, but it didn't get very far.

I may or may not get back to it, but the following config file works fine on the exported text log file.

####logstash_informatica_text.conf

````
input {
	file {
		path => "/path.to.informatica.text.file/*.txt"
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
			match => { "message" =>  "%{TIMESTAMP_ISO8601:timestamp} : %{WORD:severity} : \(%{NUMBER:pip} \| %{WORD:thread}\) : \(%{WORD:serviceType} \| %{WORD:serviceName}\) : %{WORD:clientNode} : %{WORD:messageCode} : %{GREEDYDATA:text}"}
		}
		mutate {
           	remove_field => ["message"]
           	remove_field => ["tags"]
		}
	}
}

output {
	stdout {
		codec => rubydebug
	}
	elasticsearch {
		type => "informatica_text"
	    protocol => "http"
	    host => "localhost"
	}
}
````


logstash -f logstash_informatica_text.conf