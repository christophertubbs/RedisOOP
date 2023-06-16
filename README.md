# RedisOOP
Wrapper for `redis-py` that returns results in standard objects rather than nebulous and encoded data structures

## Motivation

In `redis-cli`, what does the following yield?

```redis
XINFO GROUPS EVENTS
```

If you guessed:

```
1) 1) "name"
   2) ""
   3) "consumers"
   4) "0"
   5) "pending"
   6) "0"
   7) "last-delivered-id"
   8) "1686590285713-0"
   9) "entries-read"
   10) "null"
   11) "lag"
   12) "null"
2) 1) "name"
   2) "Event Example.20230613_162628:EVENTS:EchoBus"
   3) "consumers"
   4) "0"
   5) "pending"
   6) "0"
   7) "last-delivered-id"
   8) "1686590285713-0"
   9) "entries-read"
   10) "null"
   11) "lag"
   12) "null"
```

You'd be correct.

In python, if you called:

```python
connection = redis.Redis()
connection.xinfo_groups(name="EVENTS")
```

What would you get? You'd be correct if you guessed the following:

```
[{'name': b'', 'consumers': 0, 'pending': 0, 'last-delivered-id': b'1686590285713-0', 'entries-read': None, 'lag': None}, {'name': b'Event Example.20230613_162628:EVENTS:EchoBus', 'consumers': 0, 'pending': 0, 'last-delivered-id': b'1686590285713-0', 'entries-read': None, 'lag': None}]
```

The aim of this library is to make a simply call like the above instead return the following:

```
[
    GroupInfo(
      name: "",
      consumers: property((self) -> typing.Sequence[ConsumerInfo]),
      pending_messages: property((self) -> typing.Sequence[StreamMessageInfo]),
      last_delivered_id: "1686590285713-0",
      entries_read: 0,
      lag: None
    ),
    GroupInfo(
      name: "Event Example.20230613_162628:EVENTS:EchoBus",
      consumers: property((self) -> typing.Sequence[ConsumerInfo]),
      pending_messages: property((self) -> typing.Sequence[StreamMessage]),
      last_delivered_id: "1686590285713-0,
      entries_read: 0,
      lag: None
    )
]
```

A command like:

```python
connection.xinfo_consumers(name="EVENTS", groupname="EventBus.e2Da.20230608_1025")
```

would look like:

```
[
  ConsumerInfo(
    name: B,
    pending_messages: property((self) -> typing.Sequence[StreamMessage]),
    idle_time: datetime.timedelta(days=3, seconds=84818, microseconds=758000)
  )
]
```

instead of:

```
1) 1) "name"
   2) "B"
   3) "pending"
   4) "0"
   5) "idle"
   6) "343936374"
```

or 

```
[{'name': b'B', 'pending': 0, 'idle': 345108622}]
```
