# Operating-system-practicals-
# Operating Systems (DSC08) - Practical File

**Name:** Sarthak Singhal
**Roll No.:** 25570049
**Examination Roll No.:** 25020570028
**Course:** B.Sc. (H) Computer Science
**College:** Ramanujan College

Each question below has its own code block. Copy a block into a Linux terminal to run that practical. The C programs are created with `cat > file.c << 'EOF'`, then compiled with `gcc` and run. The programs that need input get it from a `<< 'IN'` block, so each block runs on its own.

Setup (run once):

```bash
mkdir -p oslab && cd oslab
```

---

## Question 1

Demonstration of various Operating System functions using OS Simulator.

```bash
uname -a
cat /etc/os-release | head -2
uptime
free
ps | head -12
ls /
cat /proc/version
```

---

## Question 2

Execute various LINUX commands for:
i. Information Maintenance: wc, clear, cal, who, date, pwd
ii. File Management: cat, cp, rm, mv, cmp, comm, diff, find, grep, awk
iii. Directory Management: cd, mkdir, rmdir, ls

### 2 (i) Information Maintenance

```bash
pwd
date
cal
who
whoami
id
printf 'Operating Systems\nDSC08 Practical\nLinux Commands\n' > sample.txt
wc sample.txt
wc -l sample.txt
wc -w sample.txt
wc -c sample.txt
clear
```

### 2 (ii) File Management

```bash
mkdir -p files
cd files
printf 'apple\nbanana\ncherry\ndate\n' > a.txt
printf 'banana\ncherry\nelderberry\nfig\n' > b.txt
cat a.txt
cp a.txt c.txt
ls
cmp a.txt c.txt; echo $?
cmp a.txt b.txt
comm a.txt b.txt
diff a.txt b.txt
find . -name '*.txt'
grep an a.txt b.txt
grep -n e a.txt
awk '{print NR": "$0" ("length($0)" chars)"}' b.txt
mv c.txt d.txt; ls
rm d.txt; ls
cd ..
```

### 2 (iii) Directory Management

```bash
mkdir mydir
cd mydir
pwd
mkdir sub1 sub2
ls
ls -l
cd ..
rmdir mydir/sub2
ls mydir
rmdir mydir
rm -r mydir
ls
```

---

## Question 3

Execute various LINUX commands for:
i. Process Control: fork, getpid, ps, kill, sleep
ii. Communication: Input-output redirection, Pipe
iii. Protection Management: chmod, chown, chgrp

### 3 (i) Process Control

```bash
cat > p3.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main()
{
    pid_t pid;

    printf("Before fork: process id = %d\n", getpid());
    pid = fork();
    if (pid == 0)
        printf("Child : my pid = %d, parent pid = %d\n", getpid(), getppid());
    else if (pid > 0)
    {
        wait(NULL);
        printf("Parent: my pid = %d, child pid = %d\n", getpid(), pid);
    }
    else
        perror("fork");
    return 0;
}
EOF
gcc -o p3 p3.c
./p3
echo $$
sleep 100 &
ps
kill $!
ps
date; sleep 3; date
```

### 3 (ii) Communication (redirection and pipe)

```bash
mkdir -p files
cd files
printf 'apple\nbanana\ncherry\ndate\n' > a.txt
printf 'banana\ncherry\nelderberry\nfig\n' > b.txt
echo 'Hello OS' > out.txt
cat out.txt
date >> out.txt
cat out.txt
wc -l < out.txt
ls nosuchfile 2> err.txt; cat err.txt
cat a.txt b.txt | sort | uniq
ls | wc -l
cat a.txt | grep a | wc -l
cd ..
```

### 3 (iii) Protection Management

Use `sudo` for `chown` and `chgrp`. If your simulator is already root, remove the word `sudo`.

```bash
mkdir -p files
cd files
touch file.txt; ls -l file.txt
chmod 755 file.txt; ls -l file.txt
chmod go-rwx file.txt; ls -l file.txt
sudo chown nobody file.txt; ls -l file.txt
sudo chgrp daemon file.txt; ls -l file.txt
chmod u=r,g=r,o=r file.txt; ls -l file.txt
cd ..
```

---

## Question 4

Write a program (using fork() and/or exec() commands) where parent and child execute:
i. same program, same code.
ii. same program, different code.
iii. before terminating, the parent waits for the child to finish its task.

### 4 (i) Same program, same code

```bash
cat > p4a.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main()
{
    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        return 1;
    }

    /* parent and child both execute this same code */
    printf("Hello from process %d, my parent is %d\n", getpid(), getppid());
    return 0;
}
EOF
gcc -o p4a p4a.c
./p4a
```

### 4 (ii) Same program, different code

```bash
cat > p4b.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main()
{
    pid_t pid = fork();
    int i, sum = 0;

    if (pid < 0) {
        perror("fork failed");
        return 1;
    }
    else if (pid == 0) {
        /* child: replaces its image with the "ls -l" program */
        printf("Child (pid %d) running ls -l using exec\n", getpid());
        execlp("ls", "ls", "-l", NULL);
        perror("exec failed");
    }
    else {
        /* parent: does different work - sums the numbers 1 to 10 */
        for (i = 1; i <= 10; i++)
            sum += i;
        printf("Parent (pid %d): sum of 1 to 10 = %d\n", getpid(), sum);
    }
    return 0;
}
EOF
gcc -o p4b p4b.c
./p4b
```

### 4 (iii) Parent waits for the child

```bash
cat > p4c.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main()
{
    pid_t pid = fork();
    int status, i;

    if (pid < 0) {
        perror("fork failed");
        return 1;
    }
    else if (pid == 0) {
        printf("Child  (pid %d): started working\n", getpid());
        for (i = 1; i <= 3; i++) {
            printf("Child  (pid %d): step %d\n", getpid(), i);
            sleep(1);
        }
        printf("Child  (pid %d): finished\n", getpid());
        return 3;
    }
    else {
        printf("Parent (pid %d): waiting for child %d\n", getpid(), pid);
        wait(&status);
        printf("Parent (pid %d): child exited with status %d\n",
               getpid(), WEXITSTATUS(status));
        printf("Parent (pid %d): terminating now\n", getpid());
    }
    return 0;
}
EOF
gcc -o p4c p4c.c
./p4c
```

---

## Question 5

Write a program to report behaviour of Linux kernel including kernel version, CPU type and CPU information.

```bash
cat > p5.c << 'EOF'
#include <stdio.h>
#include <string.h>
#include <sys/utsname.h>

int main()
{
    struct utsname u;
    FILE *fp;
    char line[256];

    if (uname(&u) < 0) {
        perror("uname");
        return 1;
    }
    printf("System name  : %s\n", u.sysname);
    printf("Node name    : %s\n", u.nodename);
    printf("Kernel rel.  : %s\n", u.release);
    printf("Kernel ver.  : %s\n", u.version);
    printf("Machine type : %s\n\n", u.machine);

    printf("---- CPU information (from /proc/cpuinfo) ----\n");
    fp = fopen("/proc/cpuinfo", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }
    while (fgets(line, sizeof(line), fp) != NULL) {
        if (strncmp(line, "processor", 9) == 0 ||
            strncmp(line, "vendor_id", 9) == 0 ||
            strncmp(line, "cpu family", 10) == 0 ||
            strncmp(line, "model name", 10) == 0 ||
            strncmp(line, "cpu MHz", 7) == 0 ||
            strncmp(line, "cache size", 10) == 0)
            printf("%s", line);
    }
    fclose(fp);
    return 0;
}
EOF
gcc -o p5 p5.c
./p5
```

---

## Question 6

Write a program to report behaviour of Linux kernel including information on configured memory, amount of free and used memory. (Memory information)

```bash
cat > p6.c << 'EOF'
#include <stdio.h>
#include <string.h>
#include <sys/sysinfo.h>

int main()
{
    struct sysinfo si;
    FILE *fp;
    char line[128];
    unsigned long unit;

    if (sysinfo(&si) != 0) {
        perror("sysinfo");
        return 1;
    }
    unit = si.mem_unit;

    printf("---- Memory information using sysinfo() ----\n");
    printf("Total RAM    : %lu KB\n", si.totalram * unit / 1024);
    printf("Free RAM     : %lu KB\n", si.freeram * unit / 1024);
    printf("Used RAM     : %lu KB\n", (si.totalram - si.freeram) * unit / 1024);
    printf("Shared RAM   : %lu KB\n", si.sharedram * unit / 1024);
    printf("Buffer RAM   : %lu KB\n", si.bufferram * unit / 1024);
    printf("Total swap   : %lu KB\n", si.totalswap * unit / 1024);
    printf("Free swap    : %lu KB\n", si.freeswap * unit / 1024);
    printf("Processes    : %d\n", si.procs);
    printf("Uptime       : %ld seconds\n\n", si.uptime);

    printf("---- From /proc/meminfo ----\n");
    fp = fopen("/proc/meminfo", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }
    while (fgets(line, sizeof(line), fp) != NULL) {
        if (strncmp(line, "MemTotal", 8) == 0 ||
            strncmp(line, "MemFree", 7) == 0 ||
            strncmp(line, "Buffers", 7) == 0 ||
            strncmp(line, "Cached:", 7) == 0)
            printf("%s", line);
    }
    fclose(fp);
    return 0;
}
EOF
gcc -o p6 p6.c
./p6
```

---

## Question 7

Write a program to copy files using system calls.

```bash
cat > p7.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

#define BUFSIZE 512

int main(int argc, char *argv[])
{
    int src, dst;
    ssize_t n;
    long total = 0;
    char buf[BUFSIZE];

    if (argc != 3) {
        fprintf(stderr, "Usage: %s <source> <destination>\n", argv[0]);
        exit(1);
    }

    src = open(argv[1], O_RDONLY);
    if (src < 0) {
        perror("open source");
        exit(1);
    }

    dst = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dst < 0) {
        perror("open destination");
        close(src);
        exit(1);
    }

    while ((n = read(src, buf, BUFSIZE)) > 0) {
        if (write(dst, buf, n) != n) {
            perror("write");
            exit(1);
        }
        total += n;
    }
    if (n < 0)
        perror("read");

    close(src);
    close(dst);
    printf("%ld bytes copied from %s to %s\n", total, argv[1], argv[2]);
    return 0;
}
EOF
gcc -o p7 p7.c
printf 'Operating Systems Lab\nCopy using system calls\nDSC08 Practical 7\n' > src.txt
./p7 src.txt dst.txt
cat dst.txt
cmp src.txt dst.txt && echo identical
./p7 src.txt
./p7 nofile.txt out2.txt
```

---

## Question 8

Write a program to implement First Come First Serve (FCFS) scheduling algorithm.

```bash
cat > p8.c << 'EOF'
#include <stdio.h>

int main()
{
    setbuf(stdout, NULL);
    int n, i, j, t, time = 0;
    int pid[20], at[20], bt[20], ct[20], tat[20], wt[20];
    float avg_tat = 0, avg_wt = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter arrival times : ");
    for (i = 0; i < n; i++) {
        scanf("%d", &at[i]);
        pid[i] = i + 1;
    }
    printf("Enter burst times   : ");
    for (i = 0; i < n; i++)
        scanf("%d", &bt[i]);

    /* arrange processes in order of arrival (FCFS queue) */
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - i - 1; j++) {
            if (at[j] > at[j + 1]) {
                t = at[j];  at[j] = at[j + 1];  at[j + 1] = t;
                t = bt[j];  bt[j] = bt[j + 1];  bt[j + 1] = t;
                t = pid[j]; pid[j] = pid[j + 1]; pid[j + 1] = t;
            }
        }
    }

    for (i = 0; i < n; i++) {
        if (time < at[i])          /* CPU idle till the process arrives */
            time = at[i];
        time += bt[i];
        ct[i] = time;
        tat[i] = ct[i] - at[i];
        wt[i] = tat[i] - bt[i];
        avg_tat += tat[i];
        avg_wt += wt[i];
    }

    printf("\nProcess  AT  BT  CT  TAT  WT\n");
    for (i = 0; i < n; i++)
        printf("  P%d     %2d  %2d  %2d  %3d  %2d\n",
               pid[i], at[i], bt[i], ct[i], tat[i], wt[i]);

    printf("\nGantt chart:\n|");
    for (i = 0; i < n; i++)
        printf(" P%d |", pid[i]);
    printf("\n");

    printf("\nAverage turnaround time = %.2f\n", avg_tat / n);
    printf("Average waiting time    = %.2f\n", avg_wt / n);
    return 0;
}
EOF
gcc -o p8 p8.c
./p8 << 'IN'
4
0 1 2 3
6 8 7 3
IN
```

---

## Question 9

Write a program to implement Shortest Job First (SJF) scheduling algorithm.

```bash
cat > p9.c << 'EOF'
#include <stdio.h>

int main()
{
    setbuf(stdout, NULL);
    int n, i, k = 0, idx, done = 0, time = 0;
    int at[20], bt[20], ct[20], tat[20], wt[20];
    int finished[20] = {0}, order[20];
    float avg_tat = 0, avg_wt = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter arrival times : ");
    for (i = 0; i < n; i++)
        scanf("%d", &at[i]);
    printf("Enter burst times   : ");
    for (i = 0; i < n; i++)
        scanf("%d", &bt[i]);

    while (done < n) {
        idx = -1;
        /* among the arrived processes pick the one with shortest burst */
        for (i = 0; i < n; i++) {
            if (!finished[i] && at[i] <= time) {
                if (idx == -1 || bt[i] < bt[idx])
                    idx = i;
            }
        }
        if (idx == -1) {           /* nobody has arrived yet */
            time++;
            continue;
        }
        time += bt[idx];
        ct[idx] = time;
        tat[idx] = ct[idx] - at[idx];
        wt[idx] = tat[idx] - bt[idx];
        avg_tat += tat[idx];
        avg_wt += wt[idx];
        finished[idx] = 1;
        order[k++] = idx;
        done++;
    }

    printf("\nProcess  AT  BT  CT  TAT  WT\n");
    for (i = 0; i < n; i++)
        printf("  P%d     %2d  %2d  %2d  %3d  %2d\n",
               i + 1, at[i], bt[i], ct[i], tat[i], wt[i]);

    printf("\nOrder of execution: ");
    for (i = 0; i < n; i++)
        printf("P%d ", order[i] + 1);
    printf("\n");

    printf("\nAverage turnaround time = %.2f\n", avg_tat / n);
    printf("Average waiting time    = %.2f\n", avg_wt / n);
    return 0;
}
EOF
gcc -o p9 p9.c
./p9 << 'IN'
4
0 1 2 3
7 4 1 4
IN
```

---

## Question 10

Write a program to implement non-preemptive priority-based scheduling algorithm.

```bash
cat > p10.c << 'EOF'
#include <stdio.h>

int main()
{
    setbuf(stdout, NULL);
    int n, i, k = 0, idx, done = 0, time = 0;
    int at[20], bt[20], pr[20], ct[20], tat[20], wt[20];
    int finished[20] = {0}, order[20];
    float avg_tat = 0, avg_wt = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter arrival times : ");
    for (i = 0; i < n; i++)
        scanf("%d", &at[i]);
    printf("Enter burst times   : ");
    for (i = 0; i < n; i++)
        scanf("%d", &bt[i]);
    printf("Enter priorities    : ");
    for (i = 0; i < n; i++)
        scanf("%d", &pr[i]);

    /* a smaller number means a higher priority */
    while (done < n) {
        idx = -1;
        for (i = 0; i < n; i++) {
            if (!finished[i] && at[i] <= time) {
                if (idx == -1 || pr[i] < pr[idx])
                    idx = i;
            }
        }
        if (idx == -1) {
            time++;
            continue;
        }
        time += bt[idx];          /* non-preemptive: runs till completion */
        ct[idx] = time;
        tat[idx] = ct[idx] - at[idx];
        wt[idx] = tat[idx] - bt[idx];
        avg_tat += tat[idx];
        avg_wt += wt[idx];
        finished[idx] = 1;
        order[k++] = idx;
        done++;
    }

    printf("\nProcess  AT  BT  PR  CT  TAT  WT\n");
    for (i = 0; i < n; i++)
        printf("  P%d     %2d  %2d  %2d  %2d  %3d  %2d\n",
               i + 1, at[i], bt[i], pr[i], ct[i], tat[i], wt[i]);

    printf("\nOrder of execution: ");
    for (i = 0; i < n; i++)
        printf("P%d ", order[i] + 1);
    printf("\n");

    printf("\nAverage turnaround time = %.2f\n", avg_tat / n);
    printf("Average waiting time    = %.2f\n", avg_wt / n);
    return 0;
}
EOF
gcc -o p10 p10.c
./p10 << 'IN'
4
0 1 2 3
6 4 2 3
3 1 4 2
IN
```

---

## Question 11

Write a program to calculate sum of n numbers using Pthreads.

```bash
cat > p11.c << 'EOF'
#include <stdio.h>
#include <pthread.h>

int numbers[50];
int n;
long sum = 0;                 /* shared by the main thread and the worker */

void *runner(void *param)
{
    int i;
    for (i = 0; i < n; i++)
        sum += numbers[i];
    pthread_exit(0);
}

int main()
{
    setbuf(stdout, NULL);
    pthread_t tid;
    int i;

    printf("Enter how many numbers: ");
    scanf("%d", &n);
    printf("Enter %d numbers: ", n);
    for (i = 0; i < n; i++)
        scanf("%d", &numbers[i]);

    pthread_create(&tid, NULL, runner, NULL);
    pthread_join(tid, NULL);

    printf("Sum of the %d numbers = %ld\n", n, sum);
    return 0;
}
EOF
gcc -o p11 p11.c -lpthread
./p11 << 'IN'
5
10 20 30 40 50
IN
```

---

## Question 12

Write a program to implement first-fit, best-fit and worst-fit allocation strategies.

```bash
cat > p12.c << 'EOF'
#include <stdio.h>

#define MAX 20

/* mode 0 = first fit, 1 = best fit, 2 = worst fit */
void allocate(int mode, int blocks[], int m, int proc[], int n)
{
    int rem[MAX], alloc[MAX];
    int i, j, pick, waste = 0;

    for (j = 0; j < m; j++)
        rem[j] = blocks[j];

    for (i = 0; i < n; i++) {
        alloc[i] = -1;
        pick = -1;
        for (j = 0; j < m; j++) {
            if (rem[j] >= proc[i]) {
                if (mode == 0) {                     /* first fit */
                    pick = j;
                    break;
                }
                if (pick == -1 ||
                    (mode == 1 && rem[j] < rem[pick]) ||    /* best fit  */
                    (mode == 2 && rem[j] > rem[pick]))      /* worst fit */
                    pick = j;
            }
        }
        if (pick != -1) {
            alloc[i] = pick;
            rem[pick] -= proc[i];
        }
    }

    printf("Process  Size  Block\n");
    for (i = 0; i < n; i++) {
        printf("  P%d     %3d   ", i + 1, proc[i]);
        if (alloc[i] != -1)
            printf("B%d\n", alloc[i] + 1);
        else
            printf("Not allocated\n");
    }
    for (j = 0; j < m; j++)
        waste += rem[j];
    printf("Memory left free in the blocks = %d\n", waste);
}

int main()
{
    setbuf(stdout, NULL);
    int blocks[MAX], proc[MAX];
    int m, n, i;

    printf("Enter number of memory blocks: ");
    scanf("%d", &m);
    printf("Enter block sizes   : ");
    for (i = 0; i < m; i++)
        scanf("%d", &blocks[i]);
    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter process sizes : ");
    for (i = 0; i < n; i++)
        scanf("%d", &proc[i]);

    printf("\n--- First Fit ---\n");
    allocate(0, blocks, m, proc, n);
    printf("\n--- Best Fit ---\n");
    allocate(1, blocks, m, proc, n);
    printf("\n--- Worst Fit ---\n");
    allocate(2, blocks, m, proc, n);
    return 0;
}
EOF
gcc -o p12 p12.c
./p12 << 'IN'
5
100 500 200 300 600
4
212 417 112 426
IN
```
