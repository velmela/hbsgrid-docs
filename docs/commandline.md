# 🐚 Use the Command-line

The HBS Grid uses [IBM Spectrum LSF](https://www.ibm.com/docs/en/spectrum-lsf) 
to run applications on powerful remote computers. LSF is a large and complex 
set of tools; our goal here is to give you just enough information so that you
can use it to run jobs on our system, without overwhelming you with details and
options.

---

!!! info inline end
    This software environment includes [robust graphical tools](menulaunch.md)
    that reduce the need to use the command line for many interactive tasks. This
    section is for those who prefer the command line, either for aesthetic reasons or
    because they need to submit batch jobs or carry out complex operations that cannot 
    be easily performed using graphical menu-driven tools.

LSF provides `bsub`, a command-line program for running applications on powerful remote
computers. For example, you can use

``` sh
bsub -q short_int -Is R
```

to start an interactive R job on a compute node. Breaking this example down will
make the basics of bsub clear:

-   `bsub` (**b**atch **sub**mission) is the top-level command used to run applications
    on powerful remote machines.
-   `-q short_int` means you want to run on the **short int**eractive **q**ueue (details
    below).
-   `-Is` means we are running an **I**nteractive **s**hell.
-   The rest of the command (`R` in this case) is the command that will be run on
    the remote machine.

<a name='compute-cluster-basics'></a>
!!! info "Compute cluster basics"
    When you first log in to the HBS Grid using *NoMachine* or *ssh* you are running
    on what we call a "login node". The *login nodes* do not have substantial CPU or
    RAM available. All computationally intensive processes should be run on what we
    call "compute nodes". A diagram of the HBS Grid architecture helps make this
    clear:
     
    ![Diagram showing that the login nodes are used to access storage and compute node resources](imgs/griddiagram.png){.media-small}
     
    As this diagram shows, the primary purpose of the *login nodes* is to serve as a hub 
    for launching jobs on powerful *compute nodes*. You can do that from the command line 
    using `bsub` or [from the desktop menu using application launchers](menulaunch.md).
    
    You may sometimes wish to run applications on the *login node*, and this is
    perfectly fine as long as you are not using it for computationally intensive work.
    For example, you may wish to run `ipython` to work out a small code example, or use
    `locate` to find a file you were working on. These low-resource activities can and
    should be done on the *login node*. The important thing to remember is that **`bsub` is
    used to run commands on powerful compute nodes**.


!!! important inline end
    Please keep in mind that **the system reserves the
    resources you select**, e.g., CPUs used by your job become
    unavailable for other users. **Request only 1 CPU** unless you
    know that you are using code or libraries that were written to run
    in parallel. Specific memory requirements depend on the nature of
    the job, but as a rough guide **we recommend requesting RAM 4-10
    times the size of your data**. For example, if you have a 6 Gb
    .csv file you may wish to request 24GB of memory or so.

## Resource requirements

The `bsub` command allows you to specify *RAM* and *CPU* requirements for your job via the `-M` and `-n` arguments. For example, you can run a python job with 50 GB of RAM
and 4 CPUs with

``` sh
bsub -q short_int -M 50G -n 4 -Is python
```

Knowing just these arguments to `bsub` will take you a long way. There is 
[much more to know about bsub](https://www.ibm.com/support/knowledgecenter/SSWRJV_10.1.0/lsf_command_ref/bsub.heading_options.1.html),
but these basics will get you started.

## Interactive and batch queue limits

Machines on the HBS Grid are grouped in **queues** and `bsub` can start jobs in either
*batch* (background) or *interactive* modes. *Batch* jobs make it easier to run
many jobs at once and are more efficient because jobs don't keep running after the
program is executed. Interactive jobs on the other hand tend to be more
convenient, especially for exploratory work or when developing or debugging a
script or program.

**Batch queues** including *short* and *long* are for running commands without interaction. For example

``` sh
bsub -q short Rscript my_r_code.R
```

runs `my_r_code.R` in batch mode, and

``` sh
bsub -q short stata -b my_stata_code.do
```

runs `my_stata_code.do` in batch mode.

!!! info inline end
    The key differences when submitting batch vs interactive jobs are the `-q` and
    `-Is` arguments. For example we used `-q short` for batch and `-q short_int` for 
    interactive. Interactive jobs must also include the `-Is` option.
    

**Interactive queues** like *short_int* and *long_int* are used to run
applications that you will interact with. For example,

``` sh
bsub -q short_int -Is rstudio
```

runs an interactive *RStudio* application, and

``` sh
bsub -q short_int -Is xstata
```

runs an interactive *Stata* application.

Queues have other characteristics<a name='queue-characteristics'> </a>in addition to the batch vs.
interactive distinction. These include the maximum run time and maximum
number of CPUs that can be reserved per job. These queue-level limits
are summarized in the table below.

 | Queue       | Type                   | Length     | Max Cores/Job   | 
 | ----------- | ---------------------- | ---------- | --------------- | 
 | long_int    | interactive            | 3 days     | 4               | 
 | short_int   | interactive            | 1 day      | 12              | 
 | sas_int     | interactive            | no limit   | 4               | 
 | long        | batch                  | 7 days     | 12              | 
 | short       | batch                  | 3 days     | 16              | 
 | gpu         | interactive or batch   | no limit   | 4               | 
 | sas         | batch                  | no limit   | 4               | 
 | unlimited   | interactive or batch   | no limit   | 4               | 


## Troubleshooting Jobs and Resources {#troubleshooting-jobs-and-resources}

A variety of problems can arise when running jobs and applications on
the HBSGrid. LSF provides command-line tools to monitor and inspect your jobs
to help you figure out if something goes wrong.

!!! example "Job troubleshooting steps"

    Open a *Terminal* and the HBS Grid and run the commands below to troubleshoot jobs.
    
    1. Get the JOBID number by running  
       `bjobs`  
       If your job is no longer running use  
       `bhist -a`  
       to list all your recent jobs. The JOBID is the first number in the output`.
    2. Get detailed information about a specific job by running_
       `bjobs -l <JOBID>`  
       where <JOBID> is the number you looked up in step 1.
    3. You can also look at any output produced by your job by running  
       `bpeek <JOBID>`
    4. Older jobs may not appear in `bjobs`. In that case you can still get some
       information by running  
       `bhist -l <JOBID>`  

The `bjobs -l <JOBID>` command give you information about the *state* of the job,
as defined below.

!!! info "Job state definitions"
    
    `PENDING`
     
    :   Job is awaiting a slot suitable for the requested
        resources or you've gone over your limit on resource usage. Jobs
        with high resource demands may spend significant time PENDING if the
        compute grid is busy.
        
    `RUNNING`
        
    :   Job is running.
     
    `COMPLETED`
        
    :    Job has finished and the command(s) have returned
        successfully (i.e., exit code 0).
     
    `CANCELLED`
        
    :   Job has been terminated by the user or administrator
        using `bkill`.
     
    `FAILED`
        
    :   Job finished with an exit code other than 0.


If your job has failed `bjobs` will usually tell you why, but these messages can be cryptic.
The most common are described below.

  Error                                                    | Likely Cause
  ---------------------------------------------------------| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  `JOB <jobid> CANCELLED AT <time> DUE TO TIME LIMIT`      | You did not specify enough time in your submission script. The `-W` option sets time in minutes or can also take HH:MM form (12:30 for 12.5 hours)
  `Job <jobid> exceeded <mem> memory limit, being killed`  | Your job is attempting to use more memory than you've requested for it. Either increase the amount of memory requested or, if possible, reduce the amount your application is trying to use. For example, many Java programs set heap space using the `-Xmx` JVM option. This could potentially be reduced.
  `Exited with exit code N`                                | Your job failed because your application exited with an error. Please look at the job or application logs to determine why your program exited abnormally.

For more detailed information refer to the [official LSF
documentation](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=run-jobs).

## Help and support

If you run into any problems please let us know by posting at
<https://github.com/hbs-rcs/hbsgrid-docs/discussions> and letting us
know so we can fix them! You can can also reach out to us directly via email at 
[research@hbs.edu](mailto:research@hbs.edu).
Our dedicated support team is always available to assist you.
