# Docker Volumes Command Reference Guide

- [Section 1: Volume Operations](#section-1-volume-operations)
- [Section 2: Container Volume Management](#section-2-container-volume-management)
- [Section 3: Volume Driver Operations](#section-3-volume-driver-operations)
- [Section 4: Volume Monitoring](#section-4-volume-monitoring)
- [Section 5: Backup and Restore](#section-5-backup-and-restore)
- [Section 6: Security Operations](#section-6-security-operations)
- [Section 7: Performance Optimization](#section-7-performance-optimization)

> **Author**: Md Toriqul Islam  
> **Description**: Comprehensive guide for Docker volume management commands  
> **Learning Focus**: Master Docker volume operations and data persistence  
> **Note**: Review command implications before execution in production environments

## Section 1: Volume Operations

### Basic Volume Management
```bash
# Create a new volume
docker volume create my-volume

# List all volumes
docker volume ls

# Inspect volume details
docker volume inspect my-volume

# Remove a volume
docker volume rm my-volume

# Remove all unused volumes
docker volume prune
```

### Advanced Volume Operations
```bash
# Create volume with custom driver
docker volume create \
    --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    nfs-volume

# Create volume with size limit
docker volume create \
    --driver local \
    --opt size=10G \
    sized-volume
```

## Section 2: Container Volume Management

### Volume Mounting
```bash
# Mount volume to container
docker run -d \
    --name myapp \
    --volume my-volume:/app/data \
    nginx

# Mount with read-only option
docker run -d \
    --name myapp-readonly \
    --volume my-volume:/app/data:ro \
    nginx

# Bind mount host directory
docker run -d \
    --name myapp-bind \
    --mount type=bind,source=$(pwd)/data,target=/app/data \
    nginx
```

### Volume Sharing
```bash
# Share volume between containers
docker run -d \
    --name container1 \
    --volume shared-data:/shared \
    nginx

docker run -d \
    --name container2 \
    --volumes-from container1 \
    nginx
```

## Section 3: Volume Driver Operations

### Local Driver Management
```bash
# List available volume drivers
docker info | grep "Volume Driver"

# Create volume with specific driver
docker volume create \
    --driver local \
    local-volume
```

### Network Storage Integration
```bash
# Create NFS volume
docker volume create \
    --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    nfs-volume

# Create Amazon EFS volume
docker volume create \
    --driver local \
    --opt type=efs \
    --opt device=fs-abc123:/ \
    efs-volume
```

## Section 4: Volume Monitoring

### Basic Monitoring
```bash
# Check volume usage
docker system df -v

# Monitor volume events
docker events --filter type=volume

# List volumes with filters
docker volume ls --filter driver=local
```

### Detailed Inspection
```bash
# Get volume mount point
docker volume inspect \
    --format '{{ .Mountpoint }}' \
    volume-name

# Check volume labels
docker volume inspect \
    --format '{{ .Labels }}' \
    volume-name
```

## Section 5: Backup and Restore

### Volume Backup
```bash
# Backup volume data
docker run --rm \
    --volume my-volume:/source:ro \
    --volume $(pwd):/backup \
    alpine tar czf /backup/volume-backup.tar.gz -C /source .

# Backup with timestamp
docker run --rm \
    --volume my-volume:/source:ro \
    --volume $(pwd):/backup \
    alpine tar czf /backup/volume-backup-$(date +%Y%m%d).tar.gz -C /source .
```

### Volume Restore
```bash
# Restore volume data
docker run --rm \
    --volume my-volume:/target \
    --volume $(pwd):/backup \
    alpine sh -c "cd /target && tar xzf /backup/volume-backup.tar.gz"
```

## Section 6: Security Operations

### Access Control
```bash
# Create volume with specific permissions
docker volume create \
    --driver local \
    --opt type=none \
    --opt o=bind \
    --opt device=/secure/data \
    secure-volume

# Set volume labels for access control
docker volume create \
    --label com.example.access=restricted \
    restricted-volume
```

### Security Auditing
```bash
# Audit volume mounts
docker ps --format '{{.Names}}: {{.Mounts}}'

# Check volume permissions
docker run --rm \
    --volume my-volume:/check \
    alpine ls -la /check
```

## Section 7: Performance Optimization

### Performance Tuning
```bash
# Create volume with performance options
docker volume create \
    --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=1g,nr_inodes=10k \
    high-perf-volume

# Monitor volume I/O
docker stats $(docker ps --format {{.Names}})
```

## Learning Notes

1. Always use named volumes for persistent data
2. Implement regular backup strategies
3. Monitor volume usage and growth
4. Use appropriate volume drivers for use cases
5. Follow security best practices for sensitive data

---

> 💡 **Best Practice**: Use named volumes instead of bind mounts for better portability

> ⚠️ **Warning**: Volume removal permanently deletes data. Always backup important data

> 📝 **Note**: Volume names must be unique within a Docker host