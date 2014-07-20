# Payment channel server

WARNING: This is "alpha" quality demo software.

## Included programs

* `serverd`: Payment channel server. Run with `--init` to create db.
* `serveradm`: Server administration utility.
* `servercli`: Client utility, a thin wrapper over the RPC calls

Run with `--help` to get a command line summary.

Run the utilities with no arguments, to obtain a list of commands.

Server events are logged via the flexible `'winston'` module.

See comments in `example-serverd.cfg` for further server configuration.

## Calling the JSON RPC server over HTTP:

```
POST http://localhost:12882 

{
  "method": ""
  "params": [
    {
      // Required params here (see below)
    }
  ]
}
```

Where "method" can be one of:
* `"channel.open"`
* `"channel.setRefund"`
* `"channel.commit"`
* `"channel.pay"`

### `"channel.open"`

#### POST data

`"params"` can be an empty array

#### result

```
{
  "pubkey": , // pub key of the server for this channel (also: "channel.id" for later"
  "timelock.min": , // earliest time the refund transaction can be processed
  "timelock.max": , // latest time the refund transaction can be processed
  "timelock.prefer": , // preferred time the refund transaction should be processed
}
```

### `"channel.setRefund"`

#### POST data

```
"params": {
  "channel.id": , // The id returned from "channel.open"
  "pubkey": , // pubkey of client
  "tx": , // the refund transaction, hex encoded (unsigned)
  "txInIdx": // the input id of the T1 transaction (that the server doesn't yet know about)
}
```

#### result

```
{
  "signature": // the hex encoded signature from when the server signed T1
}
```

### `"channel.commit"`

#### POST data

```
"params": {
  "channel.id": , // The id returned from "channel.open"
  "tx.commit": , // The hex encoded signature from when the client signed T1 (committing to the agreement)
  "tx.firstPayment": , // The signed transaction (T3) for the first payment, which is just a full refund payment to the client
}
```

#### result

```
true // on success
```

### `"channel.pay"`

#### POST data

```
"params": {
  "channel.id": , // The id returned from "channel.open"
  "signature": // the hex encoded signature from when the client resigned T3 with the new amount
  "amount": , // The new amount the client wants to pay to the server in T3
}
```

#### result

```
true // on success
```
