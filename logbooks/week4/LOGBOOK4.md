# Trabalho realizado na Semana #4

## Task 1: Manipulating Environment Variables

Using printev to print out environment variables:

![printenv](printenv.png)

Using printenv to print out PWD environment variable:

![printenv PWD](printenvPWD.png)

Using export on PWD environment variable (notice that the blue text changed):

![export PWD](exportPWD.png)

And using unset to get it back to normal:

![unset PWD](unsetPWD.png)

## Task 2: Passing Environment Variables from Parent Process to Child Process

### Task 2 - Step 1

After compiling and running myprintenv.c, we get the following output:

![myprintenv1](myprintenv1.png)

By analizing this, we observe that the child process has some environment
variables set.

### Task 2 - Step 2

After moving the printenv() funciton to the parent part of the code, compiling
again and running gives us the following output:

![myprintenv2](myprintenv2.png)

### Task 2 - Step 3

We seem to get an output very similar to the previous. By using the diff
command to compare them we get (almost) no output, which comfirms that the
outputs are indeed (almost) the same:

![myprintenvdiff](myprintenvdiff.png)

Our conclusion is, that after using a fork(), the environment variables of the
child process become the same of its parent.

## Task 3: Environment Variables and execve()

### Task 3 - Step 1

After compiling and running the program, we seem to get no output:

![myenv1](myenv1.png)

From this we conclude that the new loaded program probably does not automatically
inherit the environment variables of the calling process.

### Task 3 - Step 2

But after replacing NULL with environ, we observe that this time the new
program has some environment variables set:

![myenv2](myenv2.png)

### Task 3 - Step 3

From analizing the previous results, we suspect that the new program loaded by
execve() gets its environment variables from the third parameter of execve().

By checking the manual pages for execve, we confirm our suspicion ( *envp* is the
third argument of execve()):

![myenvmanual](myenvmanual.png)

## Task 4: Environment Variables and system()

As expected, we verify that the new program loaded by the calling process
seems to keep all the environment variables:

![systemenv](systemenv.png)

## Task 5: Environment Variable and Set-UID Programs

As we have already seen in task 2, the parent’s environment variables are
inherited by the child process, after calling fork(). So since, the shell
calls fork() to create a child process, and uses the child process to run
the program, we are expecting that the program will print out all the
environment variables, including those with the modified values.

![task5](task5.png)

But if we look carefuly, we can't seem to find the LD_LIBRARY_PATH environment
variable! Wonder why that is the case...

After scavenging through the manual pages, we came to the conclusion that our
binary is executed in "secure-execution mode". We can see the reason on why
that is the case in the manual pages for ld.so:

![ldmanual1](ldmanual1.png)

Here we can cleary see that, as a result of executing a set-user-ID program,
the process's real and effective user IDs differ, causing *AT_SECURE* to have
a non-zero value, which causes the program to execute in secure-execution mode.

But what does secure-execution mode have to do with LD_LIBRARY_PATH? Well,
after exploring the same manual page some more, we found the following:

![ldmanual2](ldmanual2.png)

Ah! So, since LD_LIBRARY_PATH is ignored during secure-execution mode, and our
program is executed in this mode due to making a system call to set-user-ID,
now it finally becomes clear why we couldn't seem to find the LD_LIBRARY_PATH
variable in the first screenshot of this section.
