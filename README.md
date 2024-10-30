# Docker Compose for TheHive and Cortex

> **IMPORTANT** all files in the `testing` folder are meant for prototyping, they **MUST NOT** be used in production

This repository contains a Docker Compose file used to setup TheHive and Cortex on a server for testing purpose.
Later versions will include production-ready Docker Compose files to deploy these containers on separate instances.


## Requirements

Hardware requirements:
- At least 2 vCPUs dedicated to containers
- At least 2GB of RAM per container (8GB total)

Software requirements:
- Docker engine `v23.0.15` and later ([install instructions](https://docs.docker.com/engine/install/))
- Docker Compose plugin `v2.20.2` and later ([install instructions](https://docs.docker.com/compose/install/))

To verify that everything is properly installed, you can do the following commands:
```bash
# Check Docker engine version
docker version

# Check that the current user can run Docker commands
# Else (for Linux) check out https://docs.docker.com/engine/install/linux-postinstall/
docker run hello-world

# Check Docker Compose plugin version
docker compose version
```


## Configuration
### Structure

The testing Docker Compose file is based on the following structure:
```
├── cassandra
│   ├── data
│   └── logs
├── cortex
│   ├── config
│   ├── logs
│   └── neurons
├── docker-compose.yml
├── dot.env.template
├── elasticsearch
│   ├── data
│   └── logs
└── thehive
    ├── config
    ├── data
    └── logs
```

Here are the main takeaways:
- The `dot.env.template` should be copied as `.env` and modify by the client (see [Usage](./README.md#usage))
- TheHive and Cortex are configured using the files in the `config` folders
- Every `config`, `data` and `logs` folder are synchronised with their associated container using [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
    * Permission management of these folders is paramount to prevent errors (see [Permissions](./README.md#permissions))
- Cortex is an exception with additional mountpoints:
    * `/var/run/docker.sock` to allow Cortex access to the host's Docker daemon and launch containers
    * `/tmp/cortex-jobs` (also in the command flags) for Cortex to store jobs


### Permissions

To simplify operations, we recommend to use the host's user in the containers (see [Usage](./README.md#usage)):
- You should make sure that all `config`, `data` and `logs` folders are owned and writable by the host's user
- In case of misconfiguration, clients may need to be sudoers to access / modify / remove data written by containers



## Usage

For testing purpose:

1. Copy the content of the folder `testing` on your server,
2. Run `bash ./scripts/init.sh` to generate .env file and prepare the environment
3. Run:

    ```bash
    docker compose up
    ```

First start will create volumes for:

* Elasticsearch database
* Elasticsearch logs
* Cassandra database
* Cassandra logs

TheHive attachments files are stored in `./thehive/data/files/` and logs are stored in `./thehive/log/`.


## Disclaimer

**Important Notice**
These compose files are designed to be templates and may require adjustments to suit your specific environment. The configuration and Docker Compose files provided in this repository are tailored to a general setup and should be modified to match your unique requirements and infrastructure.

**Key Points to Consider**
* **Environment Variables**: Ensure all environment variables are correctly set for your specific environment.
* **Network Configuration**: Adjust network settings to avoid conflicts with existing services.
* **Volume Mounts**: Modify volume mounts to point to appropriate directories on your host system.
* **Resource Limits**: Review and adjust resource limits (CPU, memory) to align with your system capabilities.
* **Service Dependencies**: Ensure that all dependent services and databases are properly configured and accessible.

Failure to customize these files may lead to improper functioning of the application or conflicts within your environment. Please consult the documentation and seek assistance if needed to make the necessary adjustments.

By using this repository, you acknowledge that it is your responsibility to tailor the configuration to your environment. The maintainers of this repository are not liable for any issues arising from improper configuration.
