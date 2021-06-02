---
title: "Working with the scheduler"
teaching: 50
exercises: 30
questions:
- "What is a scheduler and why are they used?"
- "How do I launch a program to run on any one node in the cluster?"
- "How do I capture the output of a program that is run on a node in the
  cluster?"
objectives:
- "Run a simple Hello World style program on the cluster."
- "Submit a simple Hello World style script to the cluster."
- "Use the batch system command line tools to monitor the execution of your
  job."
- "Inspect the output and error files of your jobs."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request more resources than you will need."
---

## Job Scheduler

An HPC system might have thousands of nodes and thousands of users. How do we
decide who gets what and when? How do we ensure that a task is run with the
resources it needs? This job is handled by a special piece of software called
the scheduler. On an HPC system, the scheduler manages which jobs run where and
when.

The following illustration compares these tasks of a job scheduler to a waiter
in a restaurant. If you can relate to an instance where you had to wait for a
while in a queue to get in to a popular restaurant, then you may now understand
why sometimes your job do not start instantly as in your laptop.

{% include figure.html max-width="75%" caption=""
   file="/fig/restaurant_queue_manager.svg"
   alt="Compare a job scheduler to a waiter in a restaurant" %}

The scheduler used in this lesson is {{ site.sched.name }}. Although
{{ site.sched.name }} is not used everywhere, running jobs is quite similar
regardless of what software is being used. The exact syntax might change, but
the concepts remain the same.

## Running a Batch Job

The most basic use of the scheduler is to run a command non-interactively. Any
command (or series of commands) that you want to run on the cluster is called a
*job*, and the process of using a scheduler to run the job is called *batch job
submission*.

In this case, the job we want to run is just a shell script. Let's create a
demo shell script to run as a test. The landing pad will have a number of
terminal-based text editors installed. Use whichever you prefer. Unsure? `nano`
is a pretty good, basic choice.

```
{{ site.remote.prompt }} nano example-job.sh
{{ site.remote.prompt }} chmod +x example-job.sh
{{ site.remote.prompt }} cat example-job.sh
```
{: .language-bash}

```
{{ site.remote.bash_shebang }}

echo -n "This script is running on "
hostname
```
{: .output}

> ## Creating Our Test Job
>
> Run the script. Does it execute on the cluster or just our login node?
>
> > ## Solution
> >
> > ```
> > {{ site.remote.prompt }} ./example-job.sh
> > ```
> > {: .language-bash}
> > ```
> > This script is running on {{ site.remote.host }}
> > ```
> > {: .output}
> >
> > This job runs on the login node.
> {: .solution}
{: .challenge}

If you completed the previous challenge successfully, you probably realise that
there is a distinction between running the job through the scheduler and just
"running it". To submit this job to the scheduler, we use the
`{{site.sched.submit.name }}` command.

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} {{ site.sched.submit.options }} example-job.sh
```
{: .language-bash}
```
sbatch: Warning: Your job has no time specification (--time=) and the default time is short. You can cancel your job with 'scancel <JOB_ID>' if you wish to resubmit.
sbatch: Warning: It appears your working directory may be on the home filesystem. It is /home2/home/ta028/ta028/userid. This is not available from the compute nodes - please check that this is what you intended. You can cancel your job with 'scancel <JOBID>' if you wish to resubmit.
Submitted batch job 286949
```
{: .output}

Ah! What went wrong here? Slurm is telling us that the file system we are currently on, `/home`, is not available
on the compute nodes and that we are getting the default, short runtime. We will deal with the runtime 
later, but we need to move to a different file system to submit the job and have it visible to the 
compute nodes. On ARCHER2, this is the `/work` file system. The path is similar to home but with 
`/work` at the start. Lets move there now, copy our job script across and resubmit:

```
{{ site.remote.prompt }} cd /work/ta028/ta028/userid
{{ site.remote.prompt-work }} cp ~/example-job.sh .
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} {{ site.sched.submit.options }} example-job.sh
```
{: .language-bash}
```
{% include {{ site.snippets }}/scheduler/basic-job-script.snip %}
```
{: .output}

That's better! And that's all we need to do to submit a job. Our work is done &mdash; now the
scheduler takes over and tries to run the job for us. While the job is waiting
to run, it goes into a list of jobs called the *queue*. To check on our job's
status, we check the queue using the command
`{{ site.sched.status }} {{ site.sched.flag.user }}`.

```
{{ site.remote.prompt-work }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

{% include {{ site.snippets }}/scheduler/basic-job-status.snip %}

The best way to check our job's status is with `{{ site.sched.status }}`. Of
course, running `{{ site.sched.status }}` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch`
command. `watch` reruns a given command at 2-second intervals. This is too
frequent, and will likely upset your system administrator. You can change the
interval to a more reasonable value, for example 15 seconds, with the `-n 15`
parameter. Let's try using it to monitor another job.

```
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} {{ site.sched.submit.options }} example-job.sh
{{ site.remote.prompt-work }} watch -n 15 {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

You should see an auto-updating display of your job's status. When it finishes,
it will disappear from the queue. Press `Ctrl-c` when you want to stop the
`watch` command.

> ## Where's the Output?
>
> On the login node, this script printed output to the terminal &mdash; but
> when we exit `watch`, there's nothing. Where'd it go?
>
> HPC job output is typically redirected to a file in the directory you
> launched it from. Use `ls` to find and read the file.
{: .discussion}

## Customising a Job

The job we just ran used some of the scheduler's default options. In a
real-world scenario, that's probably not what we want. The default options
represent a reasonable minimum. Chances are, we will need more cores, more
memory, more time, among other special considerations. To get access to these
resources we must customize our job script.

Comments in UNIX shell scripts (denoted by `#`) are typically ignored, but
there are exceptions. For instance the special `#!` comment at the beginning of
scripts specifies what program should be used to run it (you'll typically see
`{{ site.local.bash_shebang }}`). Schedulers like {{ site.sched.name }} also
have a special comment used to denote special scheduler-specific options.
Though these comments differ from scheduler to scheduler,
{{ site.sched.name }}'s special comment is `{{ site.sched.comment }}`. Anything
following the `{{ site.sched.comment }}` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the
script, but the `{{ site.sched.flag.name }}` option can be used to change the
name of a job. Add an option to the script:

```
{{ site.remote.prompt-work }} cat example-job.sh
```
{: .language-bash}

```
{{ site.remote.bash_shebang }}
{{ site.sched.comment }} {{ site.sched.flag.name }} new_name

echo -n "This script is running on "
hostname
echo "This script has finished successfully."
```
{: .output}

Submit the job and monitor its status:

```
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} {{ site.sched.submit.options }} example-job.sh
{{ site.remote.prompt-work }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}
```
{% include {{ site.snippets }}/scheduler/job-with-name-status.snip %}
```
{: .output}

Fantastic, we've successfully changed the name of our job!

### Resource Requests

But what about more important changes, such as the number of cores and memory
for our jobs? One thing that is absolutely critical when working on an HPC
system is specifying the resources required to run a job. This allows the
scheduler to find the right time and place to schedule our job. If you do not
specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources, which is probably not what you want.

The following are several key resource requests:

{% include {{ site.snippets }}/scheduler/option-flags-list.snip %}

Note that just *requesting* these resources does not make your job run faster,
nor does it necessarily mean that you will consume all of these resources. It
only means that these are made available to you. Your job may end up using less
memory, or less time, or fewer tasks or nodes, than you have requested, and it
will still run.

It's best if your requests accurately reflect your job's requirements. We'll
talk more about how to make sure that you're using resources effectively in a
later episode of this lesson.

> ## Command line options or job script options?
> All of the options we specify can be supplied on the command line (as we
> do here for `--partition=standard`) or in the job script (as we have done
> for the job name above). These are interchangeable. It is often more convenient
> to put the options in the job script as it avoids lots of typing at the command
> line.
{: .callout}

> ## Submitting Resource Requests
>
> Modify our `hostname` script so that it runs for a minute, then submit a job
> for it on the cluster. You should also move all the options we have been specifying
> on the command line (e.g. `--partition`) into the script at this point.
>
> > ## Solution
> >
> > ```
> > {{ site.remote.prompt-work }} cat example-job.sh
> > ```
> > {: .language-bash}
> >
> > ```
> > {{ site.remote.bash_shebang }}
> > {{ site.sched.comment }} {{ site.sched.flag.time }} 00:01:15
> > {{ site.sched.comment }} --partition=standard
> > {{ site.sched.comment }} --qos=standard
> > {{ site.sched.comment }} --reservation={{ site.sched.reservation }}
> >
> > echo -n "This script is running on "
> > sleep 60 # time in seconds
> > hostname
> > echo "This script has finished successfully."
> > ```
> > {: .output}
> >
> > ```
> > {{ site.remote.prompt }} {{ site.sched.submit.name }} example-job.sh
> > ```
> > {: .language-bash}
> >
> > Why are the {{ site.sched.name }} runtime and `sleep` time not identical?
> {: .solution}
{: .challenge}

{% include {{ site.snippets }}/scheduler/print-sched-variables.snip %}

Resource requests are typically binding. If you exceed them, your job will be
killed. Let's use walltime as an example. We will request 30 seconds of
walltime, and attempt to run a job for two minutes.

```
{{ site.remote.prompt-work }} cat example-job.sh
```
{: .language-bash}

```
{{ site.remote.bash_shebang }}
{{ site.sched.comment }} {{ site.sched.flag.name }} long_job
{{ site.sched.comment }} {{ site.sched.flag.time }} 00:00:30
{{ site.sched.comment }} --partition=standard
{{ site.sched.comment }} --qos=standard
{{ site.sched.comment }} --reservation={{ site.sched.reservation }}

echo "This script is running on ... "
sleep 120 # time in seconds
hostname
echo "This script has finished successfully."
```
{: .output}

Submit the job and wait for it to finish. Once it is has finished, check the
log file.

```
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} example-job.sh
{{ site.remote.prompt-work }} watch -n 15 {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

```
{% include {{ site.snippets }}/scheduler/runtime-exceeded-job.snip %}
{% include {{ site.snippets }}/scheduler/runtime-exceeded-output.snip %}
```
{: .output}

Our job was killed for exceeding the amount of resources it requested. Although
this appears harsh, this is actually a feature. Strict adherence to resource
requests allows the scheduler to find the best possible place for your jobs.
Even more importantly, it ensures that another user cannot use more resources
than they've been given. If another user messes up and accidentally attempts to
use all of the cores or memory on a node, {{ site.sched.name }} will either
restrain their job to the requested resources or kill the job outright. Other
jobs on the node will be unaffected. This means that one user cannot mess up
the experience of others, the only jobs affected by a mistake in scheduling
will be their own.

> ## But how much does it cost?
> Although your job will be killed if it exceeds the selected runtime,
> a job that completes within the time limit is only charged for the
> time it actually used. However, you should always try and specify a
> wallclock limit that is close to (but greater than!) the expected
> runtime as this will enable your job to be scheduled more
> quickly.
> If you say your job will run for an hour, the scheduler has
> to wait until a full hour becomes free on the machine. If it only ever
> runs for 5 minutes, you could have set a limit of 10 minutes and it
> might have been run earlier in the gaps between other users' jobs.
{: .callout}

## Cancelling a Job

Sometimes we'll make a mistake and need to cancel a job. This can be done with
the `{{ site.sched.del }}` command. Let's submit a job and then cancel it using
its job number (remember to change the walltime so that it runs long enough for
you to cancel it before it is killed!).

```
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} example-job.sh
{{ site.remote.prompt-work }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

```
{% include {{ site.snippets }}/scheduler/terminate-job-begin.snip %}
```
{: .output}

Now cancel the job with its job number (printed in your terminal). Absence of any
job info indicates that the job has been successfully cancelled.

```
{{ site.remote.prompt-work }} {{site.sched.del }} 38759
# It might take a minute for the job to disappear from the queue...
{{ site.remote.prompt-work }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

```
{% include {{ site.snippets }}/scheduler/terminate-job-cancel.snip %}
```
{: .output}

{% include {{ site.snippets }}/scheduler/terminate-multiple-jobs.snip %}

## Other Types of Jobs

Up to this point, we've focused on running jobs in batch mode.
{{ site.sched.name }} also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. Creating an
entire job script might be overkill, but the amount of resources required is
too much for a login node to handle. A good example of this might be building a
genome index for alignment with a tool like
[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml). Fortunately, we can
run these types of tasks as a one-off with `{{ site.sched.interactive }}`.

{% include {{ site.snippets }}/scheduler/using-nodes-interactively.snip %}

## Running parallel jobs using MPI

As we have already seen, the power of HPC systems comes from *parallelism*, i.e. having lots of
processors/disks etc. connected together rather than having more powerful components than your
laptop or workstation. Often, when running research programs on HPC you will need to run a
program that has been built to use the MPI (Message Passing Interface) parallel library. The MPI
library allows programs to exploit multiple processing cores in parallel to allow researchers
to model or simulate faster on larger problem sizes. The details of how MPI work are not important
for this course or even to use programs that have been built using MPI; however, MPI programs 
typically have to be launched in job submission scripts in a different way to serial programs and
users of parallel programs on HPC systems need to know how to do this. Specifically, launching
parallel MPI programs typically requires four things:

  - A special parallel launch program such as `mpirun`, `mpiexec`, `srun` or `aprun`.
  - A specification of how many processes to use in parallel. For example, our parallel program
    may use 256 processes in parallel.
  - A specification of how many parallel processes to use per compute node. For example, if our
    compute nodes each have 32 cores we often want to specify 32 parallel processes per node.
  - The command and arguments for our parallel program.

To illustrate this process, we will use a simple MPI parallel program that sharpens an image.
(We will meet this example program in more detail in a later episode.) Here is a job submission
script that runs the sharpen program across two compute nodes on the cluster. Create a file
(e.g. called: `run-sharpen.slurm`) with the contents of this script in it.

{% include {{ site.snippets }}/scheduler/parallel-script.snip %}

The parallel launch line for the sharpen program can be seen towards the bottom of the script:

{% include {{ site.snippets }}/scheduler/parallel-launch-desc.snip %}

As for our other jobs, we launch using the `{{ site.sched.submit.name }}` command.

```
{{ site.remote.prompt-work }} {{ site.sched.submit.name }} run-sharpen.slurm
```
{: .language-bash}


If your job runs correctly, you should see an output file called
`sharpened.pgm`

```
{{ site.remote.prompt-work }} ls -l *.pgm
```
{: .language-bash}
```
{% include {{ site.snippets }}/scheduler/ls-pgm-output.snip %}
```
{: .output}

If you only see `fuzzy.pgm` and not `sharpened.pgm` then look at
the job log files to work out what went wrong.

{% include {{ site.snippets }}/scheduler/parallel-challenge.snip %}

{% include {{ site.snippets }}/scheduler/parallel-challenge2.snip %}

{% include links.md %}
