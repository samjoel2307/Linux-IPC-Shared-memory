# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written;
    char some_text[TEXT_SZ];
};

int main() {
    int shmid;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;

    // Create shared memory
    shmid = shmget(IPC_PRIVATE, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget failed");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory id = %d\n", shmid);

    // Attach
    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        perror("shmat failed");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {   // Child (Consumer)
        while (1) {
            // Wait until producer writes
            while (shared_stuff->written == 0)
                usleep(100000); // sleep 0.1s to reduce CPU usage

            printf("Consumer received: %s\n", shared_stuff->some_text);

            if (strcmp(shared_stuff->some_text, "end") == 0)
                break;

            shared_stuff->written = 0;
        }

        if (shmdt(shared_memory) == -1)
            perror("shmdt failed (child)");

        exit(EXIT_SUCCESS);
    } else {   // Parent (Producer)
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter Some Text: ");
            if (fgets(buffer, TEXT_SZ, stdin) == NULL) {
                perror("fgets failed");
                break;
            }

            // Remove newline from fgets
            buffer[strcspn(buffer, "\n")] = '\0';

            strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
            shared_stuff->written = 1;

            printf("Producer sent: %s\n", shared_stuff->some_text);

            if (strcmp(buffer, "end") == 0)
                break;

            // Wait until consumer resets written
            while (shared_stuff->written == 1)
                usleep(100000);
        }

        wait(NULL);

        if (shmdt(shared_memory) == -1)
            perror("shmdt failed (parent)");

        if (shmctl(shmid, IPC_RMID, NULL) == -1)
            perror("shmctl failed");

        exit(EXIT_SUCCESS);
    }
}



## OUTPUT
![Alt text](<Screenshot at 2026-05-14 05-59-29.png>)

![Alt text](<Screenshot at 2026-05-14 06-00-06.png>)
# RESULT:
The program is executed successfully.
