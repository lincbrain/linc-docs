# MIT Engaging High Performance Compute Cluster

The Engaging High Performance Compute Cluster is available to LINC team members to process their jobs at scale, including with the use of GPUs.

## Create an account

In order to access the Engaging Cluster, you will need a MIT Sponsored Account.

1. Please contact Kabi at kabi@mit.edu with your organization name, date of birth, and phone number.
2. Once the sponsored account is approved, you will receive an email to complete account registration and establish your MIT Kerberos identity.
3. Please send your Kerberos ID to Kabi so that he can add you to the WebMoira group (`orcd_ug_pg_linc_all`) so that you can access the Engaging Cluster.

## Documentation overview

The MIT Office of Research Computing and Data (ORCD) manage the Engaging Cluster.  Most of the information you will need is in the first link below but there are additional resources:

1. [Engaging Cluster docs](https://engaging-web.mit.edu/eofe-wiki/)
1. [ORCD Docs](https://orcd-docs.mit.edu/)
1. [MGHPCC OpenMind GitHub wiki](https://github.mit.edu/MGHPCC/OpenMind/wiki)
1. [Slurm docs](https://slurm.schedmd.com/overview.html)

## Access the cluster and run jobs

The Engaging Cluster has head/login nodes to access the cluster and submit jobs to the compute nodes which run your resource intensive scripts.  Job orchestration is performed with the Slurm Workload Manager.  The [Engaging Cluster Documentation](https://engaging-web.mit.edu/eofe-wiki/) provides details on these operations, including:

1. [Logging into the cluster](https://engaging-web.mit.edu/eofe-wiki/logging_in/)
1. [Cluster architecture including information on the head/login nodes versus compute nodes](https://engaging-web.mit.edu/eofe-wiki/slurm/cluster_workflow/)
1. [Common commands to interact with the Slurm Job Scheduler](https://engaging-web.mit.edu/eofe-wiki/slurm/slurm/)
1. [Run multiple jobs in parallel with `sbatch`](https://engaging-web.mit.edu/eofe-wiki/slurm/sbatch/)
1. [Run interactive jobs on a single compute node with `srun` or `salloc`](https://engaging-web.mit.edu/eofe-wiki/slurm/srun/)
1. [Access installed software](https://engaging-web.mit.edu/eofe-wiki/software/load_modules/)

Slurm is a common workload manager so you can also refer to the official [Slurm documentation](https://slurm.schedmd.com/overview.html).
## Data storage

Data can be stored under the following path: `/orcd/data/linc/`.  We will be working to create an organization strategy for the LINC project data but for now please store your data under a subdirectory (e.g. `/orcd/data/linc/<username>` or `/orcd/data/linc/<projectname>`).  There are additional locations to store your data including the use of scratch space which can be found under the [Storage](https://engaging-web.mit.edu/eofe-wiki/storage/) page.

## Best practices

1. Please be respectful of these resources as they are used by many groups.
1. Only run resource intensive scripts on the compute nodes and not on the login/head nodes.
1. Only run the steps in your script on a GPU compute node if those steps require a GPU.  All other steps should be run on a CPU-only compute node.
1. Monitor your jobs frequently (`squeue -u <username>`).