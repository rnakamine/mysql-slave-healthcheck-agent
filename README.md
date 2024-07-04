# mysql-replica-healthcheck-agent

`mysql-replica-healthcheck-agent` is a HTTP daemon to show an information of "SHOW REPLICA STATUS" in JSON format.

This is useful for HAProxy's health check access instead of `option mysql-check`.

## Usage

```
$ mysql-replica-healthcheck-agent
2014/05/21 00:04:28 Listing port 5000
```

```
# Check the status of replica1
$ curl localhost:5000/replica1 | jq .
{
  "Connect_Retry": 60,
  "Exec_Source_Log_Pos": 1048,
  "Last_Errno": 0,
  "Last_Error": "",
  "Source_Host": "db-master",
  "Source_Log_File": "mysql-bin.000006",
  "Source_Port": 3306,
  ...
  "Until_Log_File": "",
  "Until_Log_Pos": 0
}

# Check the status of replica2
$ curl localhost:5000/replica2 | jq .
{
  "Connect_Retry": 60,
  "Exec_Source_Log_Pos": 1048,
  "Last_Errno": 0,
  "Last_Error": "",
  "Source_Host": "db-master",
  "Source_Log_File": "mysql-bin.000006",
  "Source_Port": 3306,
  ...
  "Until_Log_File": "",
  "Until_Log_Pos": 0
}
```

- The query "SHOW REPLICA STATUS" was successful, returning HTTP status 200 and JSON.
- If the agent could not connect to MySQL or if MySQL is not a replica, it returns HTTP status 500.
- If Seconds_Behind_Source exceeds the specified max-seconds-behind-source, it returns HTTP status 500. This allows monitoring of replica lag.

## Options

- `--port` : HTTP listen port number. Default is 5000.
- `--config` : Path to the configuration file. Default is "/etc/mysql-replica-healthcheck-agent/replicas.yml".

The configuration file is a YAML file that defines multiple replica configurations. Example:

```yaml
replica1:
  host: 127.0.0.1
  port: 3307
  user: root
  password: rootpassword
  max-seconds-behind-source: 300
  fail-replica-not-running: true

replica2:
  host: 127.0.0.1
  port: 3308
  user: root
  password: rootpassword
  max-seconds-behind-source: 300
  fail-replica-not-running: true
```

If the replica is not running, it returns HTTP status 500. When the option `fail-replica-not-running: false` is specified, it returns 200.

## License

Apache License 2.0
