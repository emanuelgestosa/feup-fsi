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
