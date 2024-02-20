# Patroni Cluster with TPA

If TPA was configured correctly, you can easily deploy a M1 cluster with Patroni as Failover Manager using Docker container by running:

```
tpaexec configure m1-patroni -a M1 --enable-patroni --postgresql 16 --platform docker --no-git
```

## Edit config.yml

I would suggest to edit the initially created config.yml and use meaningfull instance names like below:

```
---
architecture: M1
cluster_name: m1-patroni
cluster_tags: {}

cluster_vars:
  enable_pg_backup_api: false
  etcd_location: main
  failover_manager: patroni
  postgres_flavour: postgresql
  postgres_version: '16'
  preferred_python_version: python3
  use_volatile_subscriptions: true

locations:
- Name: main
- Name: dr

instance_defaults:
  image: tpa/rocky:8
  platform: docker
  vars:
    ansible_user: root

instances:
- Name: pg1
  backup: barman
  location: main
  node: 1
  role:
  - primary
- Name: pg2
  location: main
  node: 2
  role:
  - replica
  upstream: pg1
- Name: barman
  location: main
  node: 3
  role:
  - barman
  - log-server
  - monitoring-server
- Name: pg3
  location: dr
  node: 4
  role:
  - replica
  upstream: pg2
- Name: etcd1
  location: dr
  node: 5
  role:
  - etcd
  vars:
    etcd_location: main
- Name: etcd2
  location: main
  node: 6
  role:
  - etcd
  vars:
    etcd_location: main
- Name: etcd3
  location: dr
  node: 7
  role:
  - etcd
  vars:
    etcd_location: main
```

## Provision and Deploy

Like already known, you can now provision and deploy the TPA cluster:

```
tpaexec provision .
tpaexec deploy .
```

**List Container**

After deploying the cluster, you can verify if all docker containers are running as expected:

`tapexec list-containers .`
```
CONTAINER ID   NAMES     IMAGE         Cluster      STATUS
2f981c6ecef7   etcd3     tpa/rocky:8   m1-patroni   Up 40 minutes
7da4b172aca7   etcd2     tpa/rocky:8   m1-patroni   Up 40 minutes
32aa173d53c1   etcd1     tpa/rocky:8   m1-patroni   Up 40 minutes
c37219b6d07e   pg3       tpa/rocky:8   m1-patroni   Up 40 minutes
4a41a88be967   barman    tpa/rocky:8   m1-patroni   Up 40 minutes
4e2b0753bfe5   pg2       tpa/rocky:8   m1-patroni   Up 40 minutes
b36169aae21b   pg1       tpa/rocky:8   m1-patroni   Up 40 minutes

PLAY [Check for old repositories] ***********************************************************************************************************************************************************

TASK [assert] *******************************************************************************************************************************************************************************
ok: [pg1 -> localhost] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP **********************************************************************************************************************************************************************************
pg1                        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

