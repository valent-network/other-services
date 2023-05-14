
## Postgres

Keep in mind providing `postgresql.conf`, `pg_hba.conf` and `recovery.signal` to data directory if necessary. Examples are in this repo.

## Redis

Possible parameters to adjust (using `redis.conf`, `overrides.conf` or `args` ):

* maxmemory 60gb
* maxmemory-policy noeviction
* tcp-backlog 511
* tcp-keepalive 60
* timeout 300

## RedisGraph

TODO: Switch to redis-stack (new binary from redislabs). But it seems built-in algos are a bit different.

## Cron

Business logic is handled by `Clockwork` inside of Carbo, but PostgreSQL maintenance is scheduled using simple cron.

```bash
# backup
0 4 * * 1 curl -X POST "https://captain.dev.viktorvsk.com/api/v2/user/apps/webhooks/triggerbuild?namespace=captain&token=$TOKEN"

# backup cleanup
0 4 * * 2 curl -X POST "https://captain.dev.viktorvsk.com/api/v2/user/apps/webhooks/triggerbuild?namespace=captain&token=$TOKEN"

# restore and check
0 3 * * 7 curl -X POST "https://captain.dev.viktorvsk.com/api/v2/user/apps/webhooks/triggerbuild?namespace=captain&token=$TOKEN"

# wal cleanup
0 4 1,5,10,15,20,25,28 * *  curl -X POST "https://captain.dev.viktorvsk.com/api/v2/user/apps/webhooks/triggerbuild?namespace=captain&token=$TOKEN"
```

## Caprover

Currently [Caprover](https://github.com/caprover/caprover) is used to deploy recar.io. Its a self-hosted PaaS on top of docker swarm, nginx and letsencrypt

## Pre-requisits

* Install docker (e.g. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)
* Setup UFW (don't forget SSH port)
* Ensure SMTP ports are not blocked by provider


### Custom commands

If you want to run different workers from a single Dockerfile:

```yaml
TaskTemplate:
  ContainerSpec:
    Command: ["bundle", "exec", "sidekiq"]
```

### One-time commands (e.g. `rails db:migrate`)
```yaml
TaskTemplate:
  RestartPolicy:
    Condition: none
```

They can also be combined together (More details [here](https://caprover.com/docs/service-update-override.html)):

```yaml
TaskTemplate:
  ContainerSpec:
    Command: ["rails", "db:migrate"]
  RestartPolicy:
    Condition: none
```

### Limitations

* Shared ENV variables do not exist here. For server/worker/migrate/rollback/etc ENV variables should be managed separately.
* Callbacks after success/fail deploy (migrate/rollback) are not implemented so migrations are executed at the same time as the main deploy. It makes develop backward-compatible migrations and adjust database in multiple steps. It would be much better to avoid db rollbacks completely in such setup.
* Labels override. Caprover updates docker service in a "stateless" way. So if you assign LABEL_1=A and then you assign LABEL_2=B — service will have 2 labels assigned. It brings some difficulties for `ofelia`, `traefik` and other labels-based services.

## Redis HA

Redis Sentinel is configured already in Carbo and Borax and only requeris Sentinel and Master to connect to (with optional Replicas).

## PostgreSQL HA

Currently single instance is used but HA setup with `Pgpool-II`, `Patroni` and manual replica with streaming replication was configured and tested in case its necesasry.
