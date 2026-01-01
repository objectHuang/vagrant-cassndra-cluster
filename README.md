# Vagrant Cassandra Cluster

A Vagrant configuration to provision a 3-node Apache Cassandra cluster for development and testing purposes.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads) (2.2+)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (6.0+)

## Cluster Configuration

| Node | Hostname | IP Address | Role |
|------|----------|------------|------|
| 1 | cassandra-node1 | 192.168.8.16 | Seed |
| 2 | cassandra-node2 | 192.168.8.17 | - |
| 3 | cassandra-node3 | 192.168.8.18 | - |

### Specifications

- **OS:** Ubuntu 22.04 (Jammy)
- **Cassandra Version:** 4.1
- **Java:** OpenJDK 11
- **Memory:** 2GB RAM per node
- **CPUs:** 2 cores per node
- **Cluster Name:** TestCluster
- **Snitch:** GossipingPropertyFileSnitch
- **Datacenter:** dc1
- **Rack:** rack1

## Quick Start

### Start the cluster

```bash
vagrant up
```

This will provision all 3 nodes. The first node (seed) should be started first, then the others will join the cluster.

### Check cluster status

```bash
vagrant ssh cassandra-node1 -c "nodetool status"
```

Expected output shows all nodes with status `UN` (Up/Normal):

```
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address       Load       Tokens  Owns   Host ID   Rack
UN  192.168.8.16  xxx KB     16      xx%    ...       rack1
UN  192.168.8.17  xxx KB     16      xx%    ...       rack1
UN  192.168.8.18  xxx KB     16      xx%    ...       rack1
```

### Connect with cqlsh

```bash
vagrant ssh cassandra-node1 -c "cqlsh 192.168.8.16"
```

Or from any node:

```bash
vagrant ssh cassandra-node2 -c "cqlsh 192.168.8.17"
```

## Usage Examples

### Create a keyspace with replication

```sql
CREATE KEYSPACE my_keyspace WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};
```

### Create a table

```sql
USE my_keyspace;

CREATE TABLE users (
  id UUID PRIMARY KEY,
  name TEXT,
  email TEXT,
  created_at TIMESTAMP
);
```

### Insert and query data

```sql
INSERT INTO users (id, name, email, created_at) 
VALUES (uuid(), 'John Doe', 'john@example.com', toTimestamp(now()));

SELECT * FROM users;
```

## Management Commands

### SSH into a node

```bash
vagrant ssh cassandra-node1
vagrant ssh cassandra-node2
vagrant ssh cassandra-node3
```

### Restart Cassandra on a node

```bash
vagrant ssh cassandra-node1 -c "sudo systemctl restart cassandra"
```

### View Cassandra logs

```bash
vagrant ssh cassandra-node1 -c "sudo tail -f /var/log/cassandra/system.log"
```

### Stop the cluster

```bash
vagrant halt
```

### Destroy the cluster

```bash
vagrant destroy -f
```

## Troubleshooting

### Node not joining cluster

1. Check if the seed node is running:
   ```bash
   vagrant ssh cassandra-node1 -c "nodetool status"
   ```

2. Check Cassandra logs for errors:
   ```bash
   vagrant ssh cassandra-node2 -c "sudo tail -50 /var/log/cassandra/system.log"
   ```

3. Restart the node:
   ```bash
   vagrant ssh cassandra-node2 -c "sudo systemctl restart cassandra"
   ```

### Token collision error

If you see token collision errors, clear the data and restart:

```bash
vagrant ssh cassandra-node3 -c "sudo systemctl stop cassandra && sudo rm -rf /var/lib/cassandra/data/system/* && sudo rm -rf /var/lib/cassandra/commitlog/* && sudo systemctl start cassandra"
```

### Connection refused on nodetool

Wait a few minutes for Cassandra to fully start, then retry:

```bash
vagrant ssh cassandra-node1 -c "sleep 60 && nodetool status"
```

## License

MIT
