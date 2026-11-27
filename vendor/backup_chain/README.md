# Backup Chain

Require a base snapshot and contiguous increments before sealing a restorable backup chain.

Reliability and delivery · framework · 0.1.2

Framework bird: Green peafowl male.

## Drive the workflow

Start with `backup_chain_start()`. For each event call `backup_chain_step(snapshot, event, value)`, persist the returned snapshot, and dispatch `backup_chain_command(snapshot)` to your application’s adapter. The host owns I/O, event ordering, duplicate detection and durable command delivery. A matching event produces one command. Rejected events retain the phase and saved quantity. An invalid snapshot yields an empty vector.

The snapshot contains phase ID, saved quantity and last command ID. Event values are whole numbers from zero through one billion. In the table, **x** is the event value and **r** is the saved quantity. Rules match in listed order. To validate a whole history without running effects use `backup_chain_replay(events, values)`. It returns the final snapshot or an empty vector on rejection.

| Current phase | Event | Guard | Next phase | Command | New quantity |
|---|---|---|---|---|---|
| idle | base | always | copying | copy_base | x |
| copying | verified | x=r | open | accept_increment | r |
| open | increment | x=r+1 | open | record_increment | x |
| open | seal | x=r | done | publish_manifest | r |
| open | gap | x>r+1 | copying | restart_from_base | x |

Phase IDs in order from zero: idle, copying, open, done.

## Replay your own events

After installing the demo, run the following from its directory. Supply alternating event names and integer values. The command prints each requested action and exits with status 1 at the first rejected transition, or 2 for malformed arguments.

```sh
blyxc native-run main.blyx -- base 4 verified 4 increment 5 seal 5
```

## Try the published project

```sh
nest demo backup_chain@0.1.2
blyxc native-run backup_chain-demo/main.blyx
```

Install with `nest add backup_chain@0.1.2` and import `vendor::backup_chain::backup_chain`.

Maintained by Rahul Chaube. [Blyx](https://www.blyx-lang.space) · [GitHub](https://github.com/Rahulchaube1) · [LinkedIn](https://www.linkedin.com/in/rahulchaube1) · [Sponsor](https://github.com/sponsors/Rahulchaube1)
