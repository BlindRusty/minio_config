# MILVUS setup on axonvertex-01

    This document contains the instructions required to set up milvus vector database on premises using docker containers

    
    The following codes for docker-compose.yml and nginx.conf is for standalone architecture. 

    Standalone archtiecture will install etcd, minio instance for milvus , and milvus instances from images that are defined.  

    we will create folders and files for all necessary setups.

## STANDALONE ARCHITECTURE SETUP

**command to create directories for setup**

     mkdir -p /mnt/axonvertexstorage/milvus_db
     mkdir -p /mnt/axonvertexstorage/milvus_db/milvus_config
     mkdir -p /mnt/axonvertexstorage/milvus_db/volumes/
     mkdir -p /mnt/axonvertexstorage/milvus_db/volumes/etcd
     mkdir -p /mnt/axonvertexstorage/milvus_db/volumes/minio
     mkdir -p /mnt/axonvertexstorage/milvus_db/volumes/milvus
     
    

**command to create files for setup**

    
     touch /mnt/axonvertexstorage/milvus_db/milvus_config/docker-compose.yml   
    


**docker-compose.yml** 

    version: '3.5'

    services:
    etcd:
        container_name: milvus-etcd
        image: quay.io/coreos/etcd:v3.5.0
        environment:
        - ETCD_AUTO_COMPACTION_MODE=revision
        - ETCD_AUTO_COMPACTION_RETENTION=1000
        - ETCD_QUOTA_BACKEND_BYTES=4294967296
        volumes:
        - /mnt/axonvertexstorage/milvus_db/volumes/etcd:/etcd
        command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
        restart : always

    minio_milvus:
        container_name: milvus-minio
        image: minio/minio:RELEASE.2020-12-03T00-03-10Z
        environment:
        MINIO_ACCESS_KEY: minioadmin
        MINIO_SECRET_KEY: minioadmin
        volumes:
        - /mnt/axonvertexstorage/milvus_db/volumes/minio:/minio_data
        command: minio server /minio_data
        healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
        interval: 30s
        timeout: 20s
        retries: 3
        restart : always

    standalone:
        container_name: milvus-standalone
        image: milvusdb/milvus:v2.0.2
        command: ["milvus", "run", "standalone"]
        environment:
        ETCD_ENDPOINTS: etcd:2379
        MINIO_ADDRESS: minio_milvus:9000
        volumes:
        - /mnt/axonvertexstorage/milvus_db/volumes/milvus:/var/lib/milvus
        ports:
        - "19530:19530"
        depends_on:
        - "etcd"
        - "minio_milvus"
        restart : always

    networks:
    default:
        name: milvus_network


#### Deploying

**Navigate to folder with docker-compose.yml file**

    cd /mnt/axonvertexstorage/minio_config/milvus_0config

**Start the Docker Containers**

    docker-compose up -d