---
layout: single
title:  "Cron Jobs with Conda"
date:   2023-11-10 18:00:00 +0800
categories: Guides
tags: Anaconda Miniconda Conda Python Cron Bash
---
The following guide shows you how to use Unix job scheduler utility [cron](https://en.wikipedia.org/wiki/Cron) with Conda commands. It is written for Ubuntu but should be generalizable across other OS'es.

## References
* https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it
* https://stackoverflow.com/questions/36365801/run-a-crontab-job-using-an-anaconda-env

## Assumptions
* You are running on an Ubuntu-like Unix system
* You have Cron installed
* You have Conda installed

## Guide
First let's obtain the absolute path for `bash`. The following command shows that mine is `/bin/bash` which is pretty standard:
```sh
$ whereis bash
bash: /bin/bash /usr/share/man/man1/bash.1
```

Next, let's write the bash script for our crontab to run using your favourite editor:
```sh
#!/bin/bash

echo "Scheduled job start.."
conda activate <my_env>       # replace with the conda environment of your choice
python <my_python_script>.py  # replace with the python script of your choice
#...                          # insert more python scripts here if necessary
conda deactivate
echo "Completed!"
```

In this example I shall save and refer to this script as `</path/to/your/job_script.sh>`. Next we want to make sure that this file is executable:
```sh
$ chmod +x </path/to/your/job_script.sh>
```

When you installed Conda, you should see something resembling the following snippet appended to your `~/.bashrc`. Here is mine:
```sh
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/ubuntu/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/ubuntu/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/home/ubuntu/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/ubuntu/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

We will need to copy that initialization snippet and save it in another file which I recommend to be `~/.bashrc_conda`. For why this is needed, see [here](https://stackoverflow.com/questions/36365801/run-a-crontab-job-using-an-anaconda-env).

Now we are ready to create our cron job via:
```sh
crontab -e
```
If you are using this command for the first time, you will be prompted to select your editor of choice. The default nano is good enough for our purpose.

Upon opening the file, you will be greeted with the documentation header which is pretty self-explanatory and sufficient. We shall specify our scheduled job like so:

```
SHELL=/bin/bash
BASH_ENV=~/.bashrc_conda
* * * * * </path/to/your/job_script.sh> &> </path/to/your/job.log>

```
On why the first two declarations before the job specification, see [here](https://stackoverflow.com/questions/36365801/run-a-crontab-job-using-an-anaconda-env).

In this example I specified the schedule to be `* * * * *` which means it will run every minute. It's good to do this on the first attempt for us to easily check if it is running correctly. Once everything is working, we can modify it to our desired schedule again via `crontab -e`. Here I have also included a `</path/to/your/job.log>` log file to which cron will append any `stdout` and `stderr` from executing our job script. This isn't strictly necessary, but it provides a valuable avenue for debugging.

If this is the only job you are running, make sure there is an empty newline for EOF else the job wouldn't get parsed.

That's it! Once we save the job, it will be scheduled. If you still face any other issues with this example, kindly refer to the links under the references section. Happy job scheduling!
