```bash
$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json cluster_status | jq .running_nodes
[
  "rabbit@rabbitmq-1",
  "rabbit@rabbitmq-2",
  "rabbit@rabbitmq-3"
]
```

```bash
$ for number in {1..3}; do sudo docker compose exec rabbitmq-$number \
rabbitmq-diagnostics --formatter=json status | jq .rabbitmq_version; done
"4.3.2"
"4.3.2"
"4.3.2"

$ for number in {1..3}; do sudo docker compose exec rabbitmq-$number \
rabbitmq-diagnostics --formatter=json status | jq .is_under_maintenance; done
false
false
false
```

```bash
$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_users | jq
[
  {
    "user": "admin",
    "tags": [
      "administrator"
    ]
  }
]

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_permissions | jq
[
  {
    "user": "admin",
    "read": ".*",
    "write": ".*",
    "configure": ".*"
  }
]

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmqctl add_user user
Adding user "user" ...
|...|

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_users | jq
[
  {
    "user": "admin",
    "tags": [
      "administrator"
    ]
  },
  {
    "user": "user",
    "tags": []
  }
]

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmqctl set_permissions user '.*' '.*' '.*'
Setting permissions for user "user" in vhost "/" ...

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_permissions | jq
[
  {
    "user": "admin",
    "read": ".*",
    "write": ".*",
    "configure": ".*"
  },
  {
    "user": "user",
    "read": ".*",
    "write": ".*",
    "configure": ".*"
  }
]
```bash

```bash
$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_queues | jq
[
]

$ for number in {1..3}; do sudo docker compose exec rabbitmq-$number \
rabbitmqadmin --password=$(pass homeware.ovh/docker/rabbitmq/admin) --username=admin \
queues declare --name=queue-$number; done

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_queues | jq
[
  {
    "messages": 0,
    "name": "queue-1"
  },
  {
    "messages": 0,
    "name": "queue-2"
  },
  {
    "messages": 0,
    "name": "queue-3"
  }
]

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmqadmin --password=$(pass homeware.ovh/docker/rabbitmq/admin) --username=admin \
queues declare --arguments='{"x-queue-type":"quorum"}' --name=queue

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_queues | jq
[
  {
    "messages": 0,
    "name": "queue"
  },
  {
    "messages": 0,
    "name": "queue-1"
  },
  {
    "messages": 0,
    "name": "queue-3"
  },
  {
    "messages": 0,
    "name": "queue-2"
  }
]

$ sudo docker compose exec rabbitmq-1 \
rabbitmq-diagnostics --formatter=json list_queues --local | jq
[
  {
    "messages": 0,
    "name": "queue"
  },
  {
    "messages": 0,
    "name": "queue-1"
  }
]

$ sudo docker compose exec rabbitmq-2 \
rabbitmq-diagnostics --formatter=json list_queues --local | jq
[
  {
    "messages": 0,
    "name": "queue-2"
  }
]

$ sudo docker compose exec rabbitmq-3 \
rabbitmq-diagnostics --formatter=json list_queues --local | jq
[
  {
    "messages": 0,
    "name": "queue-3"
  }
]

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_bindings |
jq 'sort_by(.destination_name)'
[
  {
    "arguments": [],
    "source_name": "",
    "source_kind": "exchange",
    "destination_name": "queue",
    "destination_kind": "queue",
    "routing_key": "queue"
  },
  {
    "arguments": [],
    "source_name": "",
    "source_kind": "exchange",
    "destination_name": "queue-1",
    "destination_kind": "queue",
    "routing_key": "queue-1"
  },
  {
    "arguments": [],
    "source_name": "",
    "source_kind": "exchange",
    "destination_name": "queue-2",
    "destination_kind": "queue",
    "routing_key": "queue-2"
  },
  {
    "arguments": [],
    "source_name": "",
    "source_kind": "exchange",
    "destination_name": "queue-3",
    "destination_kind": "queue",
    "routing_key": "queue-3"
  }
]
```

![capture-01](capture-01.webp "capture 01")

```bash
$ for number in {1..3}; do sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmqadmin --password=$(pass homeware.ovh/docker/rabbitmq/admin) \
--username=admin bindings declare --destination=queue-$number --destination-type=queue \
--routing-key=queue-$number --source='amq.direct'; done

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmqadmin --password=$(pass homeware.ovh/docker/rabbitmq/admin) --username=admin \
bindings declare --destination=queue --destination-type=queue --routing-key='' \
--source='amq.direct'

$ sudo docker compose exec rabbitmq-$((RANDOM % 3 + 1)) \
rabbitmq-diagnostics --formatter=json list_bindings |
jq 'map(select(.source_name == "amq.direct")) | sort_by(.destination_name)'
[
  {
    "arguments": [],
    "source_name": "amq.direct",
    "source_kind": "exchange",
    "destination_name": "queue",
    "destination_kind": "queue",
    "routing_key": ""
  },
  {
    "arguments": [],
    "source_name": "amq.direct",
    "source_kind": "exchange",
    "destination_name": "queue-1",
    "destination_kind": "queue",
    "routing_key": "queue-1"
  },
  {
    "arguments": [],
    "source_name": "amq.direct",
    "source_kind": "exchange",
    "destination_name": "queue-2",
    "destination_kind": "queue",
    "routing_key": "queue-2"
  },
  {
    "arguments": [],
    "source_name": "amq.direct",
    "source_kind": "exchange",
    "destination_name": "queue-3",
    "destination_kind": "queue",
    "routing_key": "queue-3"
  }
]
```
