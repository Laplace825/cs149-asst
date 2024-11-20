# Standford CS149 

## ASST1 Program 1 ( runned by Apple M3 )

### important things

+  Property of a program where the same instruction sequence applies to many data elements
+ *Coherent execution IS NECESSARY for SIMD* processing resources to be used efficiently
+ *Coherent execution IS NOT NECESSARY for efficient parallelization across different cores*, since each core
has the capability to fetch/decode a different instructions from their thread’s instruction stream

### task

+ try to make mandelbrot set built with Multi-Thread has the same
Result as the Serial method.
+ add code to workerThreadStart function to accomplish this task.  
+ just change the `fn workerThreadStart`

### done

#### notice

Because the height is 1200 (maybe we can change), so if we divide the picture partly
by height div number of threads and can't get a integer result, there will be wrong
error.

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

+ 8-Threads: 3.53x speedup
+ 16-Threads: 5.13x speedup

Actually this doesn't give me 8 times faster, just 4 times faster.

#### some bad things

When we run the view 1 and set to odd thread numbers, the speedup is smaller than
less thread.

e.g. `3-Thread` gets 1.58x speedup at the same time `2-Thread` gets 1.94x.

## ASST2 Program 2 ( runned by Razon7 6800H )

*I must say this is very an interesting assignment. As we just in the lecture for
maybe 2 classes, the SIMD instructions we are not easy to get, but the CS149intrin helps
us. Although it just use fixed-length array to simulate, the code frame helps us analyse
our Vector Utilization which helps us build a SIMD eye in an acceptable way.*

### task

+ using CS149's "fake vector intrinsics" defined in `CS149intrin.h`.
+ vectorize `clampedExpVector` & `arraySumVector` so it can be run on a
machine with SIMD vector instructions.

### done

#### Vectorize Clamped EXP

*THINKS for Serial implement.* My job is just translate the Serial Version to
"CS149 SIMD" Version. Maybe you can see that in `asst1/prog2_vecintrin/main.cpp` 

#### Vectorize Array Sum

*I Like The Hint ❤️*. "You may find the `hadd` and `interleave` operations useful."
That helps me a lot.

This is a part of the code which I use `hadd` and `interleave`.

+ `hadd`
  + Adds up adjacent pairs of elements, so [0 1 2 3] -> [0+1 0+1 2+3 2+3]
+ `interleave`
  + Performs an even-odd interleaving where all even-indexed 
  elements move to front half of the array and odd-indexed to the back half,
  so [0 1 2 3 4 5 6 7] -> [0 2 4 6 1 3 5 7]  

So if I get `1 2 2 1` to get sum, just do this:

+ [[1 2 3 4]] -> [[1+2 1+2 3+4 3+4]] -> [[3 3 7 7]]
+ [[3 3 7 7]] -> [[3 7 3 7]]
+ [[3 7 3 7]] -> [[3+7 3+7 3+7 3+7]] -> result


```cpp
    // O( N / VECTOR_WIDTH )
    for (int i = 0; i < N; i += VECTOR_WIDTH) {
        _cs149_vload_float(temp, values + i, maskAll);
        _cs149_vadd_float(sum, sum, temp, maskAll);
    }

    // O( log2(VECTOR_WIDTH) )
    for (int i = 1; i < VECTOR_WIDTH; i *= 2) {
        _cs149_hadd_float(sum, sum);
        _cs149_interleave_float(sum, sum);
    }
    float result[VECTOR_WIDTH];
    _cs149_vstore_float(result, sum, maskAll);
    return result[0];
```
