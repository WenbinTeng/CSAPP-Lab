# CS:APP Shell Lab

### 0. Preface

A shell is an interactive command-line interpreter that runs programs on behalf of the user. A shell repeatedly prints a prompt, waits for a command line on *stdin*, and then carries out some action, as directed by the contents of the command line.

Your `tsh` shell should support the following built-in commands:

- The `quit` command terminates the shell.
- The `jobs` command lists all background jobs.
- The `bg <job>`  command restarts `<job>` by sending it a SIGCONT signal, and then runs it in the background. The `<job>` argument can be either a PID or a JID.
- The `fg <job>`  command restarts `<job>` by sending it a SIGCONT signal, and then runs it in the foreground. The `<job>` argument can be either a PID or a JID.



### 1. Built-in commands handlers

##### Recognizes and interprets the built-in commands

```c
/* 
 * builtin_cmd - If the user has typed a built-in command then execute
 *    it immediately.  
 */
int builtin_cmd(char **argv) 
{
    if (!strcmp(argv[0], "quit"))
    {
		exit(0);
	}
    if (!strcmp(argv[0], "jobs"))
    {
		listjobs(jobs);
		return 1;
	}
    if (!strcmp(argv[0], "fg") || !strcmp(argv[0], "bg"))
    {
		do_bgfg(argv);
		return 1;
	}
	if (!strcmp(argv[0], "&"))
    {
		return 1;
	}
    return 0;     /* not a builtin command */
}
```



#####  Implements the `bg` and `fg` built-in commands

```c
/* 
 * do_bgfg - Execute the builtin bg and fg commands
 */
void do_bgfg(char **argv) 
{
    if (argv[1] == NULL)
    {
        printf("%s command requires <PID> or %%<JID> argument\n", argv[0]);
        return;
    }

    sigset_t mask_all;
    sigset_t prev_all;
    sigfillset(&mask_all);
    sigprocmask(SIG_BLOCK, &mask_all, &prev_all);

    int pid = 0;
    int jid = 0;

    if (argv[1][0] == '%')
    {
        jid = atoi(argv[1] + 1);
    }
    else
    {
        pid = atoi(argv[1]);
    }

    if (!(pid || jid))
    {
        printf("%s: argument must be a <PID> or %%<JID>\n", argv[0]);
        sigprocmask(SIG_SETMASK, &prev_all, NULL);
        return;
    }

    struct job_t *job;

    if (pid) job = getjobpid(jobs, pid);
    if (jid) job = getjobjid(jobs, jid);

    if (job == NULL)
    {
        if (pid) printf("(%s): No such process\n", argv[1]);
        if (jid) printf("%s: No such job\n", argv[1]);
        sigprocmask(SIG_SETMASK, &prev_all, NULL);
        return;
    }

    if (!strcmp(argv[0], "bg"))
    {
        job->state = BG;
        printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
        kill(-(job->pid), SIGCONT);
        sigprocmask(SIG_SETMASK, &prev_all, NULL);
    }
    if (!strcmp(argv[0], "fg"))
    {
        if (job->state == ST) kill(-(job->pid), SIGCONT);
        job->state = FG;
        sigprocmask(SIG_SETMASK, &prev_all, NULL);
        waitfg(job->pid);
    }

    return;
}
```

#####  

##### Waits for a foreground job to complete

```c
/* 
 * waitfg - Block until process pid is no longer the foreground process
 */
void waitfg(pid_t pid)
{
    sigset_t mask;
    sigemptyset(&mask);
    fg_child_flag = 0;
    while (!fg_child_flag)
    {
        sigsuspend(&mask);
    }
    return;
}
```



### 2. Signal handlers

##### Catches SIGCHILD signals

```c
/* 
 * sigchld_handler - The kernel sends a SIGCHLD to the shell whenever
 *     a child job terminates (becomes a zombie), or stops because it
 *     received a SIGSTOP or SIGTSTP signal. The handler reaps all
 *     available zombie children, but doesn't wait for any other
 *     currently running children to terminate.  
 */
void sigchld_handler(int sig) 
{
    int olderrno = errno;

    int status;
    pid_t pid;
    sigset_t mask_all；
    sigset_t prev_all;
    sigfillset(&mask_all);

    while ((pid = waitpid(-1, &status, WUNTRACED | WNOHANG)) > 0)
    {
        sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
        struct job_t *job = getjobpid(jobs, pid);

        if (job->state == FG)
        {
            fg_child_flag = 1;
        }

        if (WIFSIGNALED(status))
        {
            printf("Job [%d] (%d) terminated by signal %d\n", pid2jid(pid), pid, WTERMSIG(status));
            deletejob(jobs, pid);
        }
        else if (WIFSTOPPED(status))
        {
            job->state = ST;
            printf("Job [%d] (%d) stopped by signal %d\n", pid2jid(pid), pid, WSTOPSIG(status));
        }
        else
        {
            deletejob(jobs, pid);
        }

        sigprocmask(SIG_SETMASK, &prev_all, NULL);
    }

    errno = olderrno;

    return;
}
```



#####  Catches SIGINT (`ctrl-c`) signals

```c
/* 
 * sigint_handler - The kernel sends a SIGINT to the shell whenver the
 *    user types ctrl-c at the keyboard.  Catch it and send it along
 *    to the foreground job.  
 */
void sigint_handler(int sig) 
{
    int olderrno = errno;

    sigset_t mask_all;
    sigset_t prev_all;
    sigfillset(&mask_all);
    sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
    pid_t pid = fgpid(jobs);
    sigprocmask(SIG_SETMASK, &prev_all, NULL);
    if (pid > 0) kill(-pid, SIGINT);
    
    errno = olderrno;

    return;
}
```



##### Catches SIGTSTP (`ctrl-z`) signals

```c
/*
 * sigtstp_handler - The kernel sends a SIGTSTP to the shell whenever
 *     the user types ctrl-z at the keyboard. Catch it and suspend the
 *     foreground job by sending it a SIGTSTP.  
 */
void sigtstp_handler(int sig) 
{
    int olderrno = errno;

    sigset_t mask_all;
    sigset_t prev_all;
    sigfillset(&mask_all);
    sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
    pid_t pid = fgpid(jobs);
    sigprocmask(SIG_SETMASK, &prev_all, NULL);
    if (pid > 0) kill(-pid, SIGTSTP);

    errno = olderrno;

    return;
}
```



### 3. Main routine

```c
/* 
 * eval - Evaluate the command line that the user has just typed in
 * 
 * If the user has requested a built-in command (quit, jobs, bg or fg)
 * then execute it immediately. Otherwise, fork a child process and
 * run the job in the context of the child. If the job is running in
 * the foreground, wait for it to terminate and then return.  Note:
 * each child process must have a unique process group ID so that our
 * background children don't receive SIGINT (SIGTSTP) from the kernel
 * when we type ctrl-c (ctrl-z) at the keyboard.  
*/
void eval(char *cmdline) 
{
    char* argv[MAXARGS];
    char buf[MAXLINE];
    int bg;
    pid_t pid;

    strcpy(buf, cmdline);
    bg = parseline(cmdline, argv);
    if (argv[0] == NULL) return;

    if (!builtin_cmd(argv))
    {
		sigset_t mask_chld;
        sigset_t mask_all;
        sigset_t prev_all;
		sigemptyset(&mask_chld);
		sigaddset(&mask_chld, SIGCHLD);
		sigfillset(&mask_all);

        sigprocmask(SIG_BLOCK, &mask_chld, &prev_mask);

        if ((pid = fork()) == 0)
        {
            sigprocmask(SIG_SETMASK, &prev_mask, NULL);
			setpgid(0, 0);
			if (execve(argv[0], argv, environ) <= 0) {
				printf("%s: Command not found\n", argv[0]);
				exit(0);
			}
        }

        sigprocmask(SIG_SETMASK, &mask_all, NULL);
		addjob(jobs, pid, bg?BG:FG, cmdline);
		sigprocmask(SIG_SETMASK, &prev_mask, NULL);

        sigprocmask(SIG_BLOCK, &mask_chld, NULL);
        if (!bg)
        {
            waitfg(pid);
        }
        else
        {
            sigprocmask(SIG_SETMASK, &mask_all, NULL);
            printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
        }
        sigprocmask(SIG_SETMASK, &prev_mask, NULL);
    }

    return;
}
```



### Reference

[shlab.dvi (cmu.edu)](http://csapp.cs.cmu.edu/3e/shlab.pdf)

