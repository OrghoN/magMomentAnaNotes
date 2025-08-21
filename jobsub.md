# Jobsub SystemGuide 

This document provides a comprehensive guide to **Jobsub**, Fermilab's primary grid job submission system, with specific application to the **NOvA experiment**.
It covers fundamental concepts, system architecture, practical usage tutorials, and techniques for efficient computational workload management in neutrino physics research.

## 1 Introduction to Jobsub

### 1.1 What is Jobsub?

**Jobsub** is Fermilab's job submission system designed specifically for managing computational workloads across distributed computing resources.
Built on top of **HTCondor**, it provides a streamlined interface for submitting, managing, and monitoring large-scale computational jobs that are essential for modern high-energy physics experiments like NOvA.
Jobsub abstracts the complexities of grid computing, allowing researchers to focus on their scientific analysis rather than computational infrastructure.

The system operates as a **client-server architecture** where users interact with the system through command-line tools or web interfaces, while the underlying infrastructure handles job scheduling, resource allocation, and data management across Fermilab's computing facilities and opportunistic resources.

Its integration with Fermilab's authentication systems, storage infrastructure, and monitoring tools creates a cohesive ecosystem for scientific computing.

## 2 NOvA Experiment Context

Within the NOvA computing model, Jobsub serves as the **primary workload management system** for most simulation and analysis tasks.
The experiment utilizes Jobsub for:

- **Production processing**: Large-scale Monte Carlo simulation campaigns that generate simulated neutrino interactions using detailed detector models .
- **Analysis tasks**: Researcher-submitted jobs for extracting physics results from both real and simulated data, including complex statistical analyses.
- **Data transformation**: Workflows that convert between data formats, apply calibration constants, and perform data reduction.

The integration between Jobsub and NOvA's data management systems ensures that jobs automatically access appropriate input data sets and store output in correctly managed locations, with comprehensive metadata tracking for reproducibility of results.

## 3 Jobsub System Architecture

### 3.1 Core Components

Jobsub's architecture consists of several interconnected components that work together to provide a seamless job management experience:

1.
 **Client Tools**: The `jobsub_submit` command and related utilities that users interact with directly.
These tools validate submission parameters, communicate with central services, and provide status feedback.

2.
 **HTCondor Infrastructure**: The underlying batch system that handles job scheduling, resource allocation, and workload distribution across worker nodes.
Jobsub extends HTCondor with experiment-specific policies and configurations.

3.
 **Authentication and Authorization**: Integration with Fermilab's Kerberos and SciToken systems to ensure secure access to computing resources and data .

4.
 **Data Management**: Automated handling of input and output data, with integration to Fermilab's dcache system and data placement services.

5.
 **Monitoring and Logging**: Comprehensive systems for tracking job status, performance metrics, and output retrieval through services like Fifemon .

### 3.2 Integration with FermiGrid Infrastructure

Jobsub deeply integrates with Fermilab's computing infrastructure, creating a cohesive ecosystem for scientific computing:

- **Resource Provisioning**: Jobs can request resources from dedicated FermiGrid allocations, opportunistic resources, or offsite resources through the Open Science Grid, depending on project needs and availability .

- **Software Environment**: Through integration with CVMFS (CernVM File System), jobs automatically access validated software versions without manual distribution of code packages.

- **Singularity Containers**: All jobs run within Singularity containers, ensuring environment consistency across diverse worker nodes and isolation from host system variations .

## 4 Common Use Cases for NOvA Researchers

### 4.1 Simulation Production

NOvA relies heavily on detailed Monte Carlo simulations to model detector response and understand systematic uncertainties.
A typical simulation job may involve:

```bash
jobsub_submit -G nova -M -N 1000 --memory=4000MB --disk=10GB --cpu=1 \
--expected-lifetime=8h --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE \
-l '+SingularityImage="/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest"' \
--append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_nova_opensciencegrid_org==true)' \
file:///nova/app/users/$USER/simulation_job.sh
```

This command submits 1000 jobs with each requesting 4GB memory, 10GB disk space, and 8 hours of runtime.
The `-G nova` flag specifies the NOvA group for resource accounting, while the `resource-provides` parameter allows the jobs to run on dedicated, opportunistic, or offsite resources .

### 4.2 Data Analysis Workflows

Physics analysis in NOvA often involves processing subsets of data through multiple stages to extract physics signals.
A typical analysis job might:

1.
 **Extract specific events** from large datasets based on selection criteria
2.
 **Apply calibration constants** and alignment corrections
3.
 **Calculate reconstructed quantities** such as energies and directions
4.
 **Apply statistical analyses** to extract physics parameters

These jobs typically use custom code that researchers develop and test before submitting at scale through Jobsub.
The system manages dependencies between processing stages and handles failures through automated retry mechanisms.

## 5 Beginner's Tutorial to Using Jobsub

Before submitting jobs, you need to setup your computing environment under novagpvm. Once ssh'd into a novagpvm machine, you can do the following

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
spack load nova-grid-utils

```

This sources the software for NOvA  and grabs the utilities that jobsub relies on and loads it into the current environment.

To make sure that signatures are up to date, run

```bash
setup_fnal_security
```

The general syntax  for running macros over the grid is

```bash
 submit_cafana.py -n <NUMBER_OF_JOBS> --print_jobsub \
--rel development -o <OUT_DIR> \
--user_tarball <PATH_TO_TARBALL> \
<MACRO_PATH>
```

Where

- `<NUMBER_OF_JOBS>` is to  be replaced by a integer that specifies how many jobs to run
- `<OUT_DIR>` is to be replaced by location of output directory
- `<PATH_TO_TARBALL>` is to be replaced by path to tarball of test release
- `<MACRO_PATH>` is to be replaced by the path to the macro you wish to run

### Best Practices

1.
 **Start small**: Test with a few jobs before submitting large batches
2.
 **Request appropriate resources**: Ask for slightly more memory and time than you expect to use to avoid unnecessary failures
3.
 **Use descriptive filenames**: Make your scripts and output files easy to identify
4.
 **Keep logs**: Save submission commands and job IDs for future reference
5.
 **Use version control**: Maintain your scripts in git or similar systems

## 6 Job Monitoring and Management

### 6.1 Checking Job Status

After submission, monitor jobs using several methods:

1.
 **Using jobsub_q**: Check the status of your submitted jobs:

```bash
jobsub_q --user=$USER
```

This command shows all your jobs with their current status (running, idle, completed, or held).

2.
 **Using Fifemon**: The web-based Fifemon service provides detailed visualizations of job progress and resource usage .

3.
 **Cluster-specific monitoring**: For larger submissions, you can monitor specific clusters:

```bash
jobsub_q -G nova 12345678.0
```

### 6.2 Handling Held Jobs

Jobs may enter a "held" state for various reasons.
Check hold reasons with:

```bash
jobsub_q --user=$USER --held
```

Common hold reasons and solutions:

- **Memory exceeded**: Request more memory or fix memory leaks in code
- **Disk exceeded**: Request more disk space or reduce output size
- **Time exceeded**: Request longer runtime or optimize code performance
- **Resource unavailable**: Wait for resources to become available or adjust requirements

Release held jobs after addressing the issue:

```bash
jobsub_release 12345678.0
```

## 7 Log Management and Troubleshooting

### 7.1 Accessing stdout and stderr

Jobsub automatically captures standard output and error streams from jobs.
These logs are stored in dCache and are accessible through several methods:

1.
 **Via Fifemon**: The primary method for accessing logs is through the Fifemon web interface :
    - Navigate to the Job Submission Summary dashboard for your job cluster
    - Click "View sandbox files" to see all log files
    - Click "View job event log" for detailed execution history and direct links to stdout/stderr

2.
 **Command line access**: 
To download job logs
```bash
jobsub_fetchlog --jobid=<jobid> --unzipdir=<output-dir>
```

Where jobid and output dir are to be filled in.

Log files are typically available for 20-30 days after job completion before being automatically cleaned up by dcache's least-recently-used deletion policy .

### 7.2 Common Errors and Solutions

*Table: Common Jobsub Errors and Resolution Strategies*
| **Error Type** | **Symptoms** | **Resolution Strategies** |
|----------------|--------------|---------------------------|
| **Authorization Failure** | "User authorization has failed" errors  | Verify group membership (-G flag), check Kerberos ticket validity, confirm role assignments |
| **Resource Exhaustion** | Jobs going on hold with memory/disk/time exceeded messages | Increase requested resources, optimize code efficiency, reduce output data volume |
| **Software Dependencies** | Missing libraries or modules in job environment | Use Singularity image options, verify CVMFS availability in requirements, include setup commands in job script |
| **Data Access Issues** | Failures to read input or write output files | Verify file paths exist, check permissions, use ifdh commands for reliable data transfer |
| **Job Scheduling Delays** | Jobs remaining idle for extended periods | Adjust resource requests, use opportunistic queues, check for system-wide issues |

### 7.3 Getting Help

When encountering persistent issues:

1.
 **Collect relevant information**: Job IDs, error messages, and log snippets
2.
 **Check known issues**: Consult experiment documentation and computing forums
3.
 **Contact support**: For NOvA-specific issues, contact the computing team through appropriate channels
4.
 **Service Desk**: For general Jobsub or infrastructure issues, submit a ticket through the Fermilab Service Desk 

## 8 Advanced Topics

### 8.1 Using RCDS for Code Distribution

For jobs requiring custom code not available in standard releases, use the Rapid Code Distribution Service (RCDS) to efficiently distribute code to jobs:

```bash
# Create a tarball of your custom code
tar -czf mycode.tar.gz mycode/

# Submit with the tarball option
jobsub_submit -G nova --tar_file_name=dropbox:///nova/app/users/$USER/mycode.tar.gz \
file:///nova/app/users/$USER/job_script.sh
```

The tarball will be automatically extracted in the job's working directory, making your custom code available to the job .

### 8.2 Job Arrays and Parameter Variation

For workflows requiring many similar jobs with slight parameter variations, use job arrays and arguments:

```bash
# Script that accepts parameters
#!/bin/bash
# job_script.sh
echo "Processing parameter value: $1"
nova -c config.tcl -p $1 output_$1.root

# Submit with varying arguments
jobsub_submit -G nova -N 5 --arg="10" --arg="20" --arg="30" --arg="40" --arg="50" \
file:///nova/app/users/$USER/job_script.sh
```

Each job instance will receive a different argument value, allowing efficient parameter scans and systematic studies.

## 9 Conclusion

Jobsub serves as the critical interface between NOvA researchers and Fermilab's distributed computing resources.
Its comprehensive feature set enables efficient management of diverse computational workloads from small interactive analyses to large-scale production processing.
By mastering Jobsub usage, NOvA researchers can effectively leverage the experiment's computing infrastructure to advance the scientific goals of the neutrino physics program.

As computing needs continue to evolve, Jobsub continues to incorporate new capabilities and optimizations.
Researchers are encouraged to stay informed about system updates through experiment computing meetings and documentation resources.

## 10 Additional Resources

- **NOvA Computing Documentation**: [https://nova.comp.fnal.gov/](https://nova.comp.fnal.gov/)
- **Fermilab Service Desk**: [https://fermi.servicenowservices.com/](https://fermi.servicenowservices.com/)
- **Jobsub Client Documentation**: [https://cdcvs.fnal.gov/redmine/projects/jobsub_client/wiki](https://cdcvs.fnal.gov/redmine/projects/jobsub_client/wiki)
- **Fifemon Monitoring**: [https://fifemon.fnal.gov/](https://fifemon.fnal.gov/)
- **Open Science Grid**: [https://opensciencegrid.org/](https://opensciencegrid.org/)

