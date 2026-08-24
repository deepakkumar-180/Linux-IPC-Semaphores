# Linux-IPC-Semaphores
# Ex05-Linux IPC-Semaphores
# NAME: DEEPAKKUMAR  S
# REG NO: 212225230042
# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.

```

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/mman.h>
#include <semaphore.h>

#define SIZE 5

int main()
{
    int *buffer;
    int *in;
    int *out;

    // Shared memory
    buffer = mmap(NULL, SIZE * sizeof(int),
                  PROT_READ | PROT_WRITE,
                  MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    in = mmap(NULL, sizeof(int),
             PROT_READ | PROT_WRITE,
             MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    out = mmap(NULL, sizeof(int),
               PROT_READ | PROT_WRITE,
               MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    *in = 0;
    *out = 0;

    // Semaphores
    sem_t *empty = mmap(NULL, sizeof(sem_t),
                        PROT_READ | PROT_WRITE,
                        MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    sem_t *full = mmap(NULL, sizeof(sem_t),
                       PROT_READ | PROT_WRITE,
                       MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    sem_t *mutex = mmap(NULL, sizeof(sem_t),
                        PROT_READ | PROT_WRITE,
                        MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    sem_init(empty, 1, SIZE);
    sem_init(full, 1, 0);
    sem_init(mutex, 1, 1);

    pid_t pid = fork();

    if (pid < 0)
    {
        perror("fork failed");
        exit(1);
    }

    if (pid == 0)
    {
        // Consumer process
        int item;

        for (int i = 1; i <= 10; i++)
        {
            sem_wait(full);
            sem_wait(mutex);

            item = buffer[*out];
            printf("Consumer consumed: %d\n", item);
            fflush(stdout);

            *out = (*out + 1) % SIZE;

            sem_post(mutex);
            sem_post(empty);

            sleep(1);
        }

        exit(0);
    }
    else
    {
        // Producer process
        for (int i = 1; i <= 10; i++)
        {
            sem_wait(empty);
            sem_wait(mutex);

            buffer[*in] = i;
            printf("Producer produced: %d\n", i);
            fflush(stdout);

            *in = (*in + 1) % SIZE;

            sem_post(mutex);
            sem_post(full);

            sleep(1);
        }

        wait(NULL);

        sem_destroy(empty);
        sem_destroy(full);
        sem_destroy(mutex);

        munmap(buffer, SIZE * sizeof(int));
        munmap(in, sizeof(int));
        munmap(out, sizeof(int));
        munmap(empty, sizeof(sem_t));
        munmap(full, sizeof(sem_t));
        munmap(mutex, sizeof(sem_t));
    }

    return 0;
}

```



## OUTPUT
# $ ./sem.o 
<img width="737" height="572" alt="exp5-os(1)" src="https://github.com/user-attachments/assets/2645495f-b89e-4914-adbc-4d3125cc89bf" />


# $ ipcs
<img width="567" height="137" alt="image" src="https://github.com/user-attachments/assets/857dade0-d82a-4bdc-9ee8-83feb82cafe1" />





# RESULT:
The program is executed successfully.
