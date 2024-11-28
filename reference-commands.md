# Docker Volumes Command Reference Guide

- [Section 1: Core Project Workflow](#section-1-core-project-workflow)
- [Section 2: Advanced Operations (Optional)](#section-2-advanced-operations)
- [Section 3: Production Guide (Optional)](#section-3-production-guide)

> **Author**: Md Toriqul Islam  
> **Description**: A guide for working with Docker volumes using Apache Cassandra, including the core project workflow and optional advanced scenarios  
> **Learning Focus**: Understanding Docker volume management and data persistence with databases  
> **Note**: This is a reference guide. Do not execute commands directly without understanding their implications.

## Section 1: Core Project Workflow

### Step 1: Volume Creation
```bash
# Create a Docker volume for Cassandra data
docker volume create \
    --driver local \
    --label example=cassandra \
    cass-shared
```

### Step 2: Running Cassandra Container
```bash
# Run Cassandra container with volume mount
docker run -d \
    --volume cass-shared:/var/lib/cassandra/data \
    --name cass1 \
    cassandra:2.2
```

### Step 3: Connecting to Cassandra
```bash
# Connect to Cassandra using CQLSH
docker run -it --rm \
    --link cass1:cass \
    cassandra:2.2 \
    cqlsh cass
```

### Step 4: Database Operations
```bash
# Check for existing keyspace
select *
from system.schema_keyspaces
where keyspace_name = 'docker_hello_world';

# Create new keyspace
create keyspace docker_hello_world
with replication = {
    'class' : 'SimpleStrategy',
    'replication_factor': 1
};

# Verify keyspace creation
select *
from system.schema_keyspaces
where keyspace_name = 'docker_hello_world';
```

### Step 5: Container Cleanup
```bash
# Exit CQLSH
quit

# Stop and remove container
docker stop cass1
docker rm -vf cass1
```

### Step 6: Data Recovery Test
```bash
# Create new container with same volume
docker run -d \
    --volume cass-shared:/var/lib/cassandra/data \
    --name cass2 \
    cassandra:2.2

# Connect to new container
docker run -it --rm \
    --link cass2:cass \
    cassandra:2.2 \
    cqlsh cass

# Verify data persistence
select *
from system.schema_keyspaces
where keyspace_name = 'docker_hello_world';
```

### Step 7: Final Cleanup
```bash
# Exit CQLSH
quit

# Remove container and volume
docker rm -vf cass2
docker volume rm cass-shared
```

## Section 2: Advanced Operations (Optional)

### Advanced Volume Management
```bash
# List all volumes
docker volume ls

# Filter volumes by label
docker volume ls --filter label=example=cassandra

# Inspect volume details
docker volume inspect cass-shared

# Create volume with custom options
docker volume create \
    --driver local \
    --opt type=none \
    --opt device=/custom/path \
    --opt o=bind \
    custom-cass-vol
```

### Advanced Database Operations
```bash
# Create table in keyspace
create table docker_hello_world.users (
    id uuid PRIMARY KEY,
    name text,
    created_at timestamp
);

# Backup keyspace schema
docker exec cass1 cqlsh -e "DESC KEYSPACE docker_hello_world" > schema_backup.cql

# Basic maintenance commands
docker exec cass1 nodetool status
docker exec cass1 nodetool repair
docker exec cass1 nodetool cleanup
```

## Section 3: Production Guide (Optional)

### Production Setup
```bash
# Run Cassandra with resource limits
docker run -d \
    --name cass-prod \
    --volume cass-prod-vol:/var/lib/cassandra/data \
    --memory 4g \
    --cpus 2 \
    --restart unless-stopped \
    --health-cmd "cqlsh -e 'describe cluster'" \
    --health-interval 30s \
    cassandra:latest
```

### Monitoring
```bash
# Check cluster health
docker exec cass-prod nodetool status
docker exec cass-prod nodetool info

# Monitor system resources
docker stats cass-prod
```

### Backup Operations
```bash
# Create snapshot
docker exec cass-prod nodetool snapshot --tag daily-backup

# Export snapshot
docker run --rm \
    --volumes-from cass-prod \
    -v backup-vol:/backup \
    debian:buster-slim \
    tar czf /backup/snapshot.tar.gz /var/lib/cassandra/data
```

## Learning Notes

1. Docker volumes persist data independently of container lifecycle
2. Volumes can be shared between containers
3. Volume mounting points are specified during container creation
4. Data in volumes survives container removal
5. Proper cleanup is essential to avoid unused volume accumulation

---

> 💡 **Best Practice**: Always use named volumes instead of anonymous volumes for better management and tracking.

> ⚠️ **Warning**: Removing a volume permanently deletes all data stored in it. Ensure proper backups before deletion.

> 📝 **Note**: Follow the core workflow steps in order for the best learning experience.