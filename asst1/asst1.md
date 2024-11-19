# Standford CS149 

## asst1

### task

+ try to make mandelbrot set built with Multi-Thread has the same
Result as the Serial method.
+ add code to workerThreadStart function to accomplish this task.  
+ just change the `fn workerThreadStart`

### need todo


### done

#### 2 thread

We split the picture generating to 2 part. The first Thread deals with
half of the picture in the top, and other draws who in the bottom.

Result:

+ Serial: 316.964ms
+ 2-Threads: 163.130ms

*Almost two times faster*

```cpp
void workerThreadStart(WorkerArgs * const args) {

    // NOTE:  in a program that uses two threads, thread 0 could compute the top
    // half of the image and thread 1 could compute the bottom half.
    printf("[thread %d] working\n", args->threadId);
    if (!args->threadId){

    mandelbrotSerial(args->x0, args->y0, args->x1, args->y1,
                     args->width,  args->height, 
                     0,  args->height / 2,
                     args->maxIterations, args->output);
    }
    else{

    mandelbrotSerial(args->x0, args->y0, args->x1, args->y1,
                     args->width,  args->height, 
                     args->height /2 ,  args->height / 2,
                     args->maxIterations, args->output);
    }
}
```

#### more thread using split method

This Idea just divides the picture horizontally to different part (match the number
of threads).

Result:

+ 8-Threads: 86.779ms

Actually this doesn't give me 8 times faster, just 4 times faster.

**When thread's number is bigger, the context switch and scheduling for cpu is more complex?**

```cpp
void workerThreadStart(WorkerArgs * const args) {

    // NOTE: Each thread make a call to mandelbrotSerial()
    // to compute a part of the output image. The part is
    // decided by number of threads totally

    printf("[thread %d] working\n", args->threadId);
    if ( !args->threadId ) {
        mandelbrotSerial(args->x0, args->y0, args->x1, args->y1,
                     args->width,  args->height, 
                     0,  args->height / args->numThreads,
                     args->maxIterations, args->output);
    }
    else {
        mandelbrotSerial(args->x0, args->y0, args->x1, args->y1,
                     args->width,  args->height, 
                     args->height / args->numThreads * args->threadId,
                     args->height / args->numThreads, args->maxIterations, args->output);

    }
}
```
