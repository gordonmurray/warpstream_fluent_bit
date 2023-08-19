### Fluent Bit, Nginx access logs and WarpStream

A quick example to see if fluent bit can send logs to WarpSteam, an s3 based alternative to Kafka.

The following was tested on Ubuntu 22.04

Install nginx

```
sudo apt install nginx-core -y
```

Install Fluent bit

```
wget -qO - https://packages.fluentbit.io/fluentbit.key | sudo apt-key add -

echo "deb https://packages.fluentbit.io/ubuntu/$(lsb_release -cs) $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/fluentbit.list

sudo apt update

sudo apt install td-agent-bit -y
```

Edit the config

```
sudo nano /etc/td-agent-bit/td-agent-bit.conf
```

Add your inputs and outputs

```
[SERVICE]

[..]

[INPUT]
    name              tail
    path              /var/log/nginx/access.log

[OUTPUT]
    Name        kafka
    Match       *
    Brokers     localhost:9092
    Topics      nginx
```

Optionally, add a log file to the config

```
[SERVICE]
    Log_File /path/to/fluent-bit.log

```

Start the fluent bit service / agent

```
sudo systemctl restart td-agent-bit
```


Set an Env var to disable WarpStream load balancing functionality, or you'll see errors like this one:

```
[error] [output:kafka:kafka.0] fluent-bit#producer-1: [thrd:app]: fluent-bit#producer-1: localhost:9092/bootstrap: Disconnected (after 30794ms in state UP, 1 identical error(s) suppressed)
```

```
export WARPSTREAM_KAFKA_LOAD_BALANCING_INTERVAL=8760h
```

Start the WarpStream agent

```
warpstream agent -agentPoolName apn_default -bucketURL mem://my-test-bucket -apiKey aks_xxxxxxxxxx -defaultVirtualClusterID vci_xxxxxxxxxx
```


Pre-create the topic in WarpStream as it isn't created automatically

```
./kafka-topics.sh --bootstrap-server api-xxxxxxxxxx.discovery.prod-z.us-east-1.warpstream.com:9092 --create --topic nginx --partitions 3
```

At this point, get some traffic going to your nginx service to contribue some access logs.

Read some of the messages off the topic

```
./kafka-console-consumer.sh --bootstrap-server api-xxxxxxxxxx.discovery.prod-z.us-east-1.warpstream.com:9092 --topic nginx --from-beginning
```
