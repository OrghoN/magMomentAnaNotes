The Fermilab Elastic Analysis Facility (EAF)

## Introduction

Modern high-energy physics experiments require rapid, scalable, and reproducible analysis frameworks.
Fermilab’s EAF is designed to meet these needs by combining container technology and orchestration platforms to support both interactive and batch processing modes.
In particular, the integration of Docker for application encapsulation and Kubernetes (or its derivatives such as OKD) for managing containerized workloads is central to the facility’s design.

## Overview of the Elastic Analysis Facility

EAF is a multi-VO (Virtual Organization) platform available to Fermilab users via their Fermilab services accounts.
Key features include:

- **Container-Based Architecture:** EAF uses Docker containers to encapsulate user applications and analysis environments.
This approach ensures that changes in underlying hardware do not affect the user environment, thereby providing portability and reproducibility.
- **Kubernetes-Oriented Deployment:** A homogeneous layer based on Kubernetes (or its OKD distribution) orchestrates the containerized services, enabling dynamic scaling and resource scheduling.
- **User Interfaces:** A JupyterHub deployment serves as the primary front end, allowing users to launch interactive notebooks that automatically connect to the containerized backend.
- **Multi-Tenant and Elastic Usage:** Users can scale out processing by launching low-latency, columnar analysis workflows that interface with batch systems and remote storage resources.

## Kubernetes Infrastructure

### Container Orchestration

- **Deployment Platform:** EAF is deployed on a heterogeneous, multi-tenant Kubernetes cluster using Red Hat’s OKD (an open-source version of OpenShift).
Kubernetes is responsible for container scheduling, auto-scaling, and resource isolation.
- **Configuration Management:** The facility makes extensive use of Helm charts and ConfigMaps to manage deployment configurations and service definitions.
This enables rapid prototyping as well as consistent production deployments.
- **Service Integration:** The Kubernetes layer seamlessly integrates services such as HTCondor for batch job submission, Dask for distributed computing, and JupyterHub for interactive computing.
This unified layer helps to abstract the complexity of underlying compute resources.

### Networking and Security

- **Multi-Tenancy:** Kubernetes’ built-in multi-tenancy features, along with network policies and secure ingress controllers, ensure that each user’s analysis environment is isolated and secure.
- **Authentication and Authorization:** LDAP and federated identity mechanisms (including OAuth2/JWT tokens) are used to provide secure access and manage user identities across the facility.

## Docker Containerization

### Docker as the Runtime Environment

- **Image-Based Deployment:** Docker is used to package analysis frameworks, custom libraries, and application code into immutable images.
These images are built from a base operating system (often a Fedora CoreOS or SL7/OKD-supported variant) and layered with experiment-specific software.
- **Standardization and Portability:** The containerized approach ensures that the same Docker image can run on any node in the cluster regardless of the underlying hardware.
This greatly simplifies testing and debugging while maintaining consistency between development and production environments.
- **Registry and Distribution:** Docker images are maintained in internal registries (or Docker Hub for collaborative efforts) and are pulled dynamically by the Kubernetes scheduler.
This supports continuous integration (CI) workflows and allows for rapid deployment of updated images.

### Docker Integration with Batch Systems

- **Hybrid Workloads:** Beyond interactive sessions, Docker containers in EAF are also used for batch processing.
Batch jobs are submitted via HTCondor, with Docker images serving as the runtime environment for simulation and data analysis workflows.
- **Performance Considerations:** Studies comparing native and containerized workloads have shown minimal performance overhead when using Docker.
This is crucial for high-throughput analysis, as it ensures that the elastic nature of the facility does not compromise efficiency.

## Underlying Infrastructure Components

### Compute Resources

- **Scalable Compute Farms:** EAF integrates with Fermilab batch computing farms that offer tens of thousands of cores.
Users can scale out their interactive analyses to these resources seamlessly.
- **GPU Integration:** The facility also supports GPU-accelerated workloads.
For instance, A100 GPUs are partitioned into multiple instances to provide low-latency inference and high-performance model training.

### Storage and Data Access

- **Local and Remote Storage:** User data is managed through a combination of persistent user storage (e.g., 25 GB per user area) and scratch spaces (40 GB for GPU jobs).
Additionally, CVMFS mounts and dCache systems provide access to large-scale experiment data.
- **Data Caching:** Tools such as XRootD and xCache help minimize latency by caching frequently accessed datasets, which is particularly important for columnar analysis workflows.

### Monitoring and Metrics

- **Resource Monitoring:** The facility integrates Grafana dashboards and Prometheus to monitor real-time resource usage.
This includes CPU, memory, storage I/O, and network traffic.
- **User Feedback:** In-notebook monitoring tools provide users with immediate feedback on the performance of their analysis tasks, enabling more efficient resource utilization and troubleshooting.

## Deployment and Use Cases

### Interactive Analysis

- **JupyterHub Integration:** Users access EAF via a JupyterHub interface that launches individual notebooks.
These notebooks are backed by Docker containers deployed on the Kubernetes cluster.
- **Low-Latency Workflows:** The architecture is designed to support fast turnaround times for interactive sessions, enabling rapid prototyping and iterative analysis.

### Batch Processing

- **HTCondor and Hybrid Scheduling:** For large-scale data processing, users can convert interactive workflows to batch jobs.
Docker containers ensure that the runtime environment is consistent whether the job runs interactively or as a batch submission.
- **Scalable Resource Provisioning:** Kubernetes orchestrates batch workloads alongside interactive sessions, allowing for efficient use of the overall compute infrastructure.

## ssh access

Fermilab security policies have different security standards for web vs ssh access. 
EAF was set up with web and batch access in mind using condor and therefore could use a different authentication standard than ssh which requires kerberos.

The jupyter hub interface is not accesible for the blind and as a result, fermilab agreed to grant me a security exception for accessibility reasons.

Work was then done on both the client and server side to get the appropriate software set up. The following is a list of instructions to get ssh access to the eaf server.

### EAF SSH instructions

One time setup only:

Download the modified websocat and scripts:
  https://www.dropbox.com/scl/fi/dh9wrni8voxp7p073npy0/eaf-ssh.tgz?rlkey=abgaznnprg5jnieowygeft0qi&dl=1
  and put it the three files there (websocat, encrypt-token.sh, eaf-ssh.sh) in your PATH.
 
Generate ssh key on client:
  If you don't already have ssh keys on your client, generate them with the ssh-keygen command.
  
Authorize ssh key on server:
  Go to https://analytics-hub.fnal.gov and select "Start My Server".  Choose a flavor.
  Start a terminal launcher and copy your public key from your client (~/.ssh/id_rsa.pub) to ~/.ssh/authorized_keys on the jupyter server.
  On the server, chmod 600 ~/.ssh/authorized_keys
  
On a weekly basis, create a token:
  Create a token at https://analytics-hub.fnal.gov/hub/token
  Give it whatever name you like, and set the "Token Expires" to 1 week.
  Copy the token (the string after "Your new API token").
  Run "encrypt-token.sh".  It will prompt you for the token you just copied and a password to encrypt it - choose whatever password you like.
  
To use:
    Go to https://analytics-hub.fnal.gov and select "Start My Server" if one is not already running. 
    Run eaf-ssh.sh - it will prompt you for the password that you encrypted the token with, and then you should have CLI access to your server.

### EAF Quickstart

Generate Token at

```
https://analytics-hub.fnal.gov/hub/token
```

Encrypt token by

```bash
cd eaf-ssh
export PATH=/nashome/o/oneogi/eaf-ssh:$PATH
./encrypt_token.sh
cp ~/eaf-token.enc .
```

Start server at

```
https://analytics-hub.fnal.gov
```

ssh into eaf by

```bash
./eaf-ssh.sh
```

	## References

- citeturn0search0 – [Building an Elastic Analysis Facility with Kubernetes – Maria P.
Acosta (PDF)](https://indico.cern.ch/event/896935/contributions/3782797/attachments/2004968/3348249/Acosta_EF_K8s_jointCMS-ATLAS.pdf)
- citeturn0search1 – [The Elastic Analysis Facility at Fermilab – CMS Indico Slides](https://indico.cern.ch/event/903719/contributions/3803524/attachments/2013546/3364991/Elastic_AF_-_Fermilab.pdf)
- citeturn0search2 – [The Elastic Analysis Facility at Fermilab – USCMS AF meeting (PDF)](https://indico.mit.edu/event/956/contributions/2842/attachments/998/1637/USCMSAF-EAF_LindseyGray_31012024.pdf)
- citeturn0search7 – [Study of a Docker use-case for HEP – Fermilab Technical Memo](https://lss.fnal.gov/archive/test-tm/2000/fermilab-tm-2625-cd.pdf)
- citeturn0search9 – [Collaborative Computing Support for Analysis Facilities Exploiting ...
(arXiv)](https://arxiv.org/abs/2203.10161)
- citeturn0search11 – [The Elastic Analysis Facility (EAF) at Fermilab – Inspire HEP](https://inspirehep.net/files/7e542c7ab03940def7280a4989254f64)

---

This Markdown report provides an in-depth overview of the EAF’s elastic analysis framework with a focus on the use of Kubernetes and Docker, along with the supporting infrastructure that makes it a robust tool for Fermilab’s analysis community.
