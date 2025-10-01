# CLUMrdm2025: Data Management in de.NBI Cloud Workshop

The de.NBI and de.KCD Cloud User Meeting: Data management in de.NBI cloud workshop aims to provide an introduction to data management methods in the de.NBI cloud infrastructure.

## Table of Contents

- [Overview](#overview)
- [Workshop Materials](#workshop-materials)
  - [Part 1: Core Data Management](#part-1-core-data-management)
  - [Part 2: Advanced Storage Solutions](#part-2-advanced-storage-solutions)
- [Prerequisites](#prerequisites)
- [Workshop Organizers](#workshop-organizers)

---

## Overview

This workshop provides hands-on training for managing data in the de.NBI (German Network for Bioinformatics Infrastructure) cloud platform. Participants will learn practical skills for working with various storage solutions, including block storage volumes and object storage containers.

The workshop is designed for researchers and scientists who need to manage large datasets in the de.NBI cloud environment.

---

## Workshop Materials

### Part 1: Core Data Management

[`Part 1`](dm_in_cloud.md)

This tutorial covers the fundamental aspects of data management in the de.NBI cloud platform. Topics include:

1. **Section 1: Create a new VM**
   - Creating virtual machines via SimpleVM webportal
   - Initial VM configuration and package installation
   - Installing OpenStack client tools

2. **Section 2: Create a volume to store data**
   - Working with block storage volumes
   - Creating, attaching, and mounting volumes
   - File system setup and permissions management

3. **Section 3: Using Object Storage**
   - Setting up OpenStack credentials and application credentials
   - Creating S3 credentials for object storage access
   - Configuring the Minio client
   - Creating and managing object storage containers (buckets)
   - Uploading and downloading data using the Minio client

4. **Section 4: Transferring your data into the cloud**
   - Using `scp` for secure file transfers
   - Using `rsync` for efficient bulk data synchronization


### Part 2: Using s3fs to mount containers to VMs

[`Part 2`](dm_in_the_cloud_part2.md)

This tutorial focuses on mounting S3-compatible object storage directly to virtual machines using s3fs-FUSE. Topics include:

1. **Installing s3fs-fuse**
   - Package installation on Ubuntu-based SimpleVM instances

2. **Preparing credentials**
   - Listing and managing OpenStack EC2 credentials
   - Securing credential files with appropriate permissions

3. **Creating mount points**
   - Setting up directories for S3 container mounts
   - Configuring user permissions

4. **Testing foreground mounts**
   - Interactive mounting for testing and validation
   - Verifying read/write permissions

5. **Configuring automatic mounting**
   - Setting up persistent mounts via `/etc/fstab`
   - Configuring your machine for automatic container mounting at boot

---


## Prerequisites

Before starting the workshop, participants should have:

- An active de.NBI cloud account
- SSH keys configured for SimpleVM access
- Basic familiarity with Linux command-line operations
- Basic understanding of cloud computing concepts
- Access to the CLUMrdm2025 workshop project

---

## Workshop Organizers

This workshop is organized by:
- **Sebastian Jünemann**
- **Abhijeet Shah**

For assistance during the workshop, please grant SSH access to the organizers through the SimpleVM interface.

---

## Additional Resources

- [de.NBI Cloud Documentation](https://cloud.denbi.de/)
- [s3fs-fuse Documentation](https://github.com/s3fs-fuse/s3fs-fuse)
- [Minio Client Documentation](https://min.io/docs/minio/linux/reference/minio-mc.html)

---

*Last updated: October 2025*