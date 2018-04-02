# Tuning-Performance-CSC252
                        My Implementation gives mean of 2.4 for rotate() and 3.2 for smooth. Let me know if there is any improments I can make. Here is the description of the project if you want to understand what we are trying to do. I will update explanation of my implementation soon.
                        
This project deals with optimizing memory intensive code.  Image processing offers many examples of functions that can benefit from optimization.  We will consider two image processing operations:  rotate, which rotates an image counter-clockwise by 90° and smooth, which "smooths" or "blurs" an image. 

We will consider an image to be represented as a two-dimensional matrix M, where Mi,j denotes the value of the (i,j)th pixel of M.  Pixel values are triples of red, green, and blue (RGB) values.  We will only consider square images.  Let N denote the number of rows (or columns) of an image.  Rows and columns are numbered, in C-style, from 0 to N-1. 

The rotate operation turns an image 90° counter-clockwise, by moving each element M1[i][j] into M2[N-j-1][i].  

The smooth operation is implemented by replacing every pixel value with the average of all the pixels around it (in a maximum of 3 X 3 window centered at that pixel).  Consider the following figure: 

diagram of elements used in smoothing

The values of pixels M2[1][1] and M2[N-1][N-1] are given below: 

      Value of pixel M2[1][1] 
      
      Value of pixel M2[N-1][N-1]

The only file we will be modifying and handing in is kernels.c.  The driver.c program is a driver program that allows you to evaluate the performance of your solutions.  Use the command make driver to generate the driver code and run it with the command ./driver. 
 
 
Implementation Overview
Data Structures
The core data structure deals with image representation.  A pixel is a struct as shown below: 

      typedef struct {
            unsigned short red;   /* R value */
            unsigned short green; /* G value */
            unsigned short blue;  /* B value */
        } pixel;
  
As can be seen, RGB values have 16-bit representations.  An image I is represented as a one dimensional array of pixels, where the (i,j)th pixel is I[RIDX(i,j,N)].  Here N is the width (height) of the image matrix, and RIDX is a macro defined as follows: 
#define RIDX(i,j,N) ((i)*(N)+(j))
  
See the file defs.h for this code. 
### Rotate
The following C function computes the result of rotating the source image src by 90° and stores the result in destination image dst.  N is the width (height) of the image. 

      void naive_rotate(int N, pixel *src, pixel *dst) {
            int i, j;
            for(i=0; i < N; i++)
                for(j=0; j < N; j++)
                    dst[RIDX(N-1-j,i,N)] = src[RIDX(i,j,N)];
            return;
        }
  
The above code scans the rows of the source image matrix, copying to the columns of the destination image matrix.  Your task is to rewrite this code to make it run as fast as possible using techniques like code motion, loop unrolling and blocking. 
See the file kernels.c for this code. 

### Smooth
The smoothing function takes as input a source image src and returns the smoothed result in the destination image dst.  Here is part of an implementation: 

      void naive_smooth(int N, pixel *src, pixel *dst) {
            int i, j;
            for(i=0; i < N; i++)
                for(j=0; j < N; j++)
                    dst[RIDX(i,j,N)] = avg(N, i, j, src); /* Smooth the (i,j)th pixel */
            return;
        }
  
The function avg returns the average of all the pixels around the (i,j)th pixel.  Your task is to optimize smooth (and avg) to run as fast as possible.  (Note:  The function avg is a local function and you can get rid of it altogether to implement smooth in some other way.) 
This code (and an implementation of avg) is in the file kernels.c. 

### Performance measures

Our main performance measure is CPE or Cycles per Element.  If a function takes C cycles to run for an image of size N X N, the CPE value is C/N2.  Table 1 summarizes the performance of the naive implementations shown above and compares it against an optimized implementation.  Performance is shown for for 5 different values of N.  All measurements were made on cycle2.csug.rochester.edu.  As you know from the last project, these results may be very different on different machines.  Your code will be tested and graded on cycle2 so you should keep this in mind as you program.  Note in particular that the reference baseline CPEs listed are specific to cycle2. 

The ratios (speedups) of the optimized implementation over the naive one will constitute a score of your implementation.  To summarize the overall effect over different values of N, we will compute the geometric mean of the results for these 5 values.  That is, if the measured speedups for N = {32,64,128,256,512} are R32, R64, R128, R256 and R512 then we compute the overall performance as

\begin{eqnarray*} R & = & \sqrt[5]{R_{32} \times R_{64} \times R_{128} \times R_{256} \times R_{512}}\end{eqnarray*}

#### uPEs and Ratios for Optimized vs. Naive Implementations
      Test case	1	2	3	4	5	 
      Method	N	64	128	256	512	1024	Geom. Mean
      Naive rotate (CPE)	 	3.3	4.8	7.0	11.8	14.4	 
      Optimized rotate (CPE)	 	3.0	3.1	3.2	4.0	4.7	 
      Speedup (naive/opt)	 	1.1	1.5	2.2	2.9	3.1	2.0
      Method	N	32	64	128	256	512	Geom. Mean
      Naive smooth (CPE)	 	76.2	77.2	77.4	77.6	77.7	 
      Optimized smooth (CPE)	 	25.1	27.0	27.3	27.5	27.6	 
      Speedup (naive/opt)	 	3.0	2.9	2.8	2.8	2.8	2.9
      
### Assumptions
To make life easier, you may assume that N is a multiple of 32.  Your code must run correctly for all such values of N, but we will measure its performance only for the 5 values shown in Table 1. 

### Infrastructure
We have provided support code to help you test the correctness of your implementations and measure their performance.  This section describes how to use this infrastructure.  The exact details of each part of the assignment are described in the following section.  

Note:  The only source file you will be modifying is kernels.c. 

### Versioning
We will be writing many versions of the rotate and smooth routines.  To help you compare the performance of all the different versions you’ve written, we provide a way of “registering” functions. 

For example, the file kernels.c that we have provided you contains the following function: 

      void register_rotate_functions() {
            add_rotate_function(&rotate, rotate_descr);
        }
  
This function contains one or more calls to add_rotate_function.  In the above example, add_rotate_function registers the function rotate along with a string rotate_descr which is an ASCII description of what the function does.  See the file kernels.c to see how to create the string descriptions.  This string can be at most 256 characters long. 

A similar function for your smooth kernels is provided in the file kernels.c. 

### Driver
The source code you will write will be linked with object code that we supply into a "driver" binary.  To create this binary, you will need to execute the command

      unix> make driver
  
You will need to re-make driver each time you change the code in kernels.c.  To test your implementations, you can then run the command: 
           
      unix> ./driver
  
Some advice:  Look at the assembly code generated for rotate and smooth.  Focus on optimizing the inner loop (the code that gets repeatedly executed in a loop) using the optimization tricks covered in class and in chapters 5 and 6 of the text.  (Note: if you didn't read chapter 5 carefully before the midterm, you'll want to read it now.  You won't do well on this assignment without it.)  The smooth function is more compute-intensive and less memory-sensitive than the rotate function, so the optimizations are of somewhat different flavors.  You may want to consult the authors' "Web Aside" on using blocking to increase temporal locality. 

How might the topic in section 5.5 of the textbook be used to improve the functionality of naive_smooth? 
Turn In Instructions
When you have completed the lab, you will hand in one file, kernels.c, that contains your solution.  Here is how to hand in your solution: 
