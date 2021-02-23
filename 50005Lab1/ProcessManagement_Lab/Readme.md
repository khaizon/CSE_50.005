
## 1. Busy Wait
in the busy wait, I implemented a while loop with the condition "task not dispatched"
```C
int task_not_dispatched = 1;
while (task_not_dispatched == 1) {

```
hence the while loop will only stop running when a task is dispatched

for a task to be dispatched, I did a for loop for the number for children to find
a child that is alive and not busy. Once found we can dispatch the task by setting the job type and duration to the job struct. 
```C
for (int i =0; i < number_of_processes; i++) {
    int isalive = waitpid(children_processes[i], NULL, WNOHANG);
    if (shmPTR_jobs_buffer[i].task_status == 0 && !isalive) {
        shmPTR_jobs_buffer[i].task_duration = num;
        shmPTR_jobs_buffer[i].task_type = action;
        shmPTR_jobs_buffer[i].task_status = 1;
        task_not_dispatched = 0; 
        sem_post(sem_jobs_buffer[i]);
        break;
    }
```
### 2. Revival of the dead
If the child is not alive, then we revive the child by calling the fork system call.
```C
    else if (isalive != 0) {
        pid_t childPID = fork();
        if (childPID < 0) {
            fprintf(stderr, "Failed to creat fork");
        } 
        else if (childPID > 0) {
            children_processes[i] = childPID;
        }
        else {
            job_dispatch(i);
        }
    }
```
If the child is not alive, then we revive the child by calling the fork system call.

once the task is found, we can stop the busy wait while loop by setting "task not dispatched" to false, so that the main loop can retrive the next task to be dispatched.
## 3. Termination of the living
To terminate teh worker processes in i did a for loop and a busy wait. The busy wait ensures that all jobs are completed before termination. This is done by ensuring that the "task_status" of the child is no longer 1.
```C
for (int child = 0; child < number_of_processes; child++){
    int alive = waitpid(children_processes[child], NULL, WNOHANG);
    while ((shmPTR_jobs_buffer[child].task_status == 1))
        ;
    if (alive == 0) { //if child is still alive
        shmPTR_jobs_buffer[child].task_status = 1;
        shmPTR_jobs_buffer[child].task_duration = 0;
        shmPTR_jobs_buffer[child].task_type = 'z';
        sem_post(sem_jobs_buffer[child]);
    }
}
```

