
Zero - Apply for a Consortium Account
    - WestGrid for using Cedar System
    - SHARCNET for using Graham System

NOTE: Some WestGrid systems have job time limits as short as one day. It is recommended that you design your program to include a checkpoint and restart capability.

Login node: graham.computecanada.ca
Globus endpoint: computecanada#graham-dtn
Data mover node (rsync, scp, sftp,...): gra-dtn1.computecanada.ca

1 - Graham System:

By policy, Graham's compute nodes cannot access the internet. If you need an exception to this rule, contact technical support with the following information:

IP:
Port/s:
Protocol:  TCP or UDP
Contact:
Removal Date:

2 - Attached storage systems:

Home space
    Location of home directories.
    Each home directory has a small, fixed quota.
    Not allocated via RAS or RAC. Larger requests go to Project space.
    Has daily backup.

Scratch space (3.6PB total volume and Parallel high-performance filesystem)
    For active or temporary (/scratch) storage.
    Not allocated.
    Large fixed quota per user.
    Inactive data will be purged.

Project space (External persistent storage)
    Part of the National Data Cyberinfrastructure.
    Allocated via RAS or RAC.
    Not designed for parallel I/O workloads. Use Scratch space instead.
    Large adjustable quota per project.
    Has daily backup.

3 - Node types and characteristics

count	Node type	cores	available memory	hardware detail
160	GPU	        32	124G or 127518M	         like base nodes but also two NVIDIA P100 Pascal GPUs (12GB HBM2 memory, 1.6TB NVMe SSD

Best practice for local on-node storage is to use the temporary directory generated by Slurm, $SLURM_TMPDIR. Note that this directory and its contents will disappear upon job completion.

Note that the amount of available memory is less than the "round number" suggested by hardware configuration. For instance, "base" nodes do have 128 GiB of RAM, but some of it is permanently occupied by the kernel and OS. To avoid wasting time by swapping/paging, the scheduler will never allocate jobs whose memory requirements exceed the specified amount of "available" memory. Please also note that the memory allocated to the job must be sufficient for IO buffering performed by the kernel and filesystem - this means that an IO-intensive job will often benefit from requesting somewhat more memory than the aggregate size of processes.


USEFUL TIPS:

1 - Useful commands for running jobs:

    qsub: to processes the job, it assigns it a job ID and places the job in a queue to await execution.

    showq: to check on the status of all the jobs on the system.

    showq -u username: to limit the listing to show just the jobs associated with your user name.

    qdel jobid: to delete a job, use the qdel command with the jobid assigned from qsub.

    NOTE: See https://www.westgrid.ca/support/job_monitoring for more information on commands that can be use to monitor job queues and machines.


2 - On Compute Canada clusters, the job scheduler is the Slurm Workload Manager. Some commands include:

    sbatch: to submit a job'. For example, submitting job inside simple_job script:

        [someuser@host ~]$ sbatch simple_job.sh
            Submitted batch job 123456

        File: simple_job.sh

            #!/bin/bash
            #SBATCH --time=00:01:00
            #SBATCH --account=def-someuser
            echo 'Hello, world!'
            sleep 30

        NOTE: Directives (or "options") in the job script are prefixed with #SBATCH and must precede all executable commands. Compute Canada policies require that you supply at least a time limit (--time) for each job. You may also need to supply an account name (--account).

        NOTE: Memory may be requested with --mem-per-cpu (memory per core) or --mem (memory per node). We recommend that you specify memory in megabytes (e.g. 8000M) rather than gigabytes (e.g. 8G). In many circumstances specifying memory requests in thousands of megabytes will result in shorter queue wait times than specifying gigabytes. For example, requesting --mem=128G (equivalent to 131072M) is more memory than is available for jobs on nodes with a nominal 128G . A job requesting --mem=128G qualifies for fewer nodes than one requesting --mem=128000M, and is therefore likely to wait longer. Similar reasoning also applies to --mem-per-cpu requests.

        The acceptable time formats include "minutes", "minutes:seconds", "hours:minutes:seconds", "days-hours", "days-hours:minutes" and "days-hours:minutes:seconds".


    squeue: TO lists pending and running jobs. Supply your username as an argument with -u to list only your own jobs:

        [someuser@host ~]$ squeue -u $USER
            JOBID PARTITION      NAME     USER ST   TIME  NODES NODELIST(REASON)
            123456 cpubase_b  simple_j someuser  R   0:03      1 cdr234
            123457 cpubase_b  simple_j someuser PD             1 (Priority)

        NOTE: The ST column of the output shows the status of each job. The two most common states are "PD" for "pending" or "R" for "running".


Email notification
You can ask to be notified by email of certain job conditions by supplying options to sbatch:

#SBATCH --mail-user=<email_address>
#SBATCH --mail-type=BEGIN
#SBATCH --mail-type=END
#SBATCH --mail-type=FAIL
#SBATCH --mail-type=REQUEUE
#SBATCH --mail-type=ALL

Cancelling jobs
Use scancel with the job ID to cancel a job:

 scancel <jobid>
You can also use it to cancel all your jobs, or all your pending jobs:

 scancel -u $USER
scancel -t PENDING -u $USER


Storage and file management

File Transfer

1 - use scp to copy individul files and directories

    $ scp filename username@cedar.computecanada.ca:/path/to
    $ scp username@cedar.computecanada.ca:/path/to/filename localPath

2 - use rsync to sync files or directories

    $ flags='-av --progress --delete'
    $ rsync $flags localPath/*pattern* username@cedar.computecanada.ca:/path/to
    $ rsync $flags username@cedar.computecanada.ca:/path/to/*pattern* localPath



Attaching to a running job

Suppose you want to run the utility nvidia-smi to monitor GPU usage on a node where you have a job running. The following command runs watch on the node assigned to the given job, which in turn runs nvidia-smi every 30 seconds, displaying the output on your terminal.

    [name@server ~]$ srun --jobid 123456 --pty watch -n 30 nvidia-smi

It is possible to launch multiple monitoring commands using tmux. The following command launches htop and nvidia-smi in separate panes to monitor the activity on a node assigned to the given job.

    [name@server ~]$ srun --jobid 123456 --pty tmux new-session -d 'htop -u $USER' \; split-window -h 'watch nvidia-smi' \; attach


Getting Job Usage Statistics
    sacct: to provide some usage statistics for jobs that are running, and those that have completed. Output can be filtered and formatted to provide specific information, including requested memory and peak memory used during job execution.

        sacct --jobs=383 --format=User,JobID,account,Timelimit,elapsed,ReqMem,MaxRss,ExitCode

        [ttrojan@hpc-login3 ~]$ sacct --jobs=383 --format=User,JobID,account,Timelimit,elapsed,ReqMem,MaxRss,ExitCode
            User         JobID      Account     Timelimit      Elapsed       ReqMem      MaxRSS ExitCode
        --------- ------------- ------------ ------------- ------------ ------------ ----------- --------
        ttrojan 383                lc_usc1      02:00:00     01:28:59  	 1Gc                  0:0
                383.extern         lc_usc1                   01:28:59          1Gc                  0:0
        [ttrojan@hpc-login3 ~]$



