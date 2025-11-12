# Linux-IPC--Pipes
Linux-IPC-Pipes


# Ex03-Linux IPC - Pipes

# AIM:
To write a C program that illustrate communication between two process using unnamed and named pipes

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - pipe(), fifo()

### Step 3:

Testing the C Program for the desired output. 

# PROGRAM:

## C Program that illustrate communication between two process using unnamed pipes using Linux API system calls
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h> 
#include <sys/stat.h> 
#include <string.h> 
#include <fcntl.h> 
#include <unistd.h>
#include <sys/wait.h>

void server(int rfd, int wfd); 
void client(int wfd, int rfd); 

#define BUFFER_SIZE 2000

int main() 
{ 
    int p1[2], p2[2];
    pid_t pid;
    int status;

    if (pipe(p1) == -1 || pipe(p2) == -1) {
        perror("pipe failed");
        return EXIT_FAILURE;
    }

    pid = fork(); 

    if (pid < 0) {
        perror("fork failed");
        return EXIT_FAILURE;
    }

    if (pid == 0) {
        close(p1[1]); 
        close(p2[0]); 
        
        server(p1[0], p2[1]);
        
        close(p1[0]);
        close(p2[1]);
        
        return 0;
    } 

    close(p1[0]); 
    close(p2[1]); 
    
    client(p1[1], p2[0]); 
    
    close(p1[1]);
    close(p2[0]);
    
    wait(&status); 
    
    if (WIFEXITED(status)) {
        printf("\nClient waited for Server (PID %d) which exited with status %d.\n", pid, WEXITSTATUS(status));
    }
    
    return 0; 
} 

void server(int rfd, int wfd) 
{ 
    char fname[BUFFER_SIZE]; 
    char buff[BUFFER_SIZE];
    ssize_t n;
    int fd;
    const char *error_msg = "Can't open file";

    n = read(rfd, fname, BUFFER_SIZE - 1);

    if (n <= 0) {
        return;
    }
    
    fname[n] = '\0';
    
    printf("Server (PID %d) received file request: '%s'\n", getpid(), fname);

    fd = open(fname, O_RDONLY);

    if (fd < 0) {
        printf("Server: File '%s' not found or cannot be opened.\n", fname);
        write(wfd, error_msg, strlen(error_msg) + 1); 
    } else {
        n = read(fd, buff, BUFFER_SIZE);
        close(fd);

        if (n < 0) {
            write(wfd, "File read error", strlen("File read error") + 1);
        } else {
            printf("Server: Sending %zd bytes of file content.\n", n);
            write(wfd, buff, n); 
        }
    }
}

void client(int wfd, int rfd) {
    char fname[BUFFER_SIZE];
    char buff[BUFFER_SIZE];
    ssize_t n;

    printf("ENTER THE FILE NAME: ");
    
    if (fgets(fname, BUFFER_SIZE, stdin) == NULL) {
        return;
    }
    
    fname[strcspn(fname, "\n")] = 0; 

    printf("CLIENT SENDING THE REQUEST... PLEASE WAIT\n");

    write(wfd, fname, strlen(fname) + 1);

    n = read(rfd, buff, BUFFER_SIZE);

    if (n <= 0) {
        printf("CLIENT: Failed to read response from server.\n");
        return;
    }
    
    printf("\n--- THE SERVER RESPONSE ---\n");
    
    if (n == strlen("Can't open file") + 1 && strncmp(buff, "Can't open file", n) == 0) {
        buff[n] = '\0';
        printf("ERROR: %s\n", buff);
    } else {
        write(1, buff, n);
        printf("\n\n--- END OF FILE CONTENT (%zd bytes) ---\n", n);
    }
}
```


## OUTPUT
<img width="443" height="155" alt="image" src="https://github.com/user-attachments/assets/234e34bf-b283-4dda-be7e-c63653e3bc77" />


## C Program that illustrate communication between two process using named pipes using Linux API system calls
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>

int main() {
    const char *fifo_path = "/tmp/my_fifo";
    mode_t fifo_mode = 0666;
    int res;

    res = mkfifo(fifo_path, fifo_mode);

    if (res == 0) {
        printf("FIFO '%s' created successfully.\n", fifo_path);
    } else {
        if (errno == EEXIST) {
            printf("FIFO '%s' already exists.\n", fifo_path);
        } else {
            perror("mkfifo failed");
            exit(EXIT_FAILURE);
        }
    }

    exit(EXIT_SUCCESS);
}
```


## OUTPUT
<img width="422" height="79" alt="image" src="https://github.com/user-attachments/assets/a02fecc4-3a2d-4f1f-b58d-f8bd0ab3f7de" />


# RESULT:
The program is executed successfully.
