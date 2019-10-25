# HPSC Lab 9
2019-10-25

## Introduction to libCEED

libCEED is a low-level API library for the efficient
high-order discretization methods developed by the ECP co-design [Center for
Efficient Exascale Discretizations (CEED)](http://ceed.exascaleproject.org).

While our focus is on high-order finite elements, the approach is mostly
algebraic and thus applicable to other discretizations in factored form, as
explained in the API documentation portion of the [Doxygen documentation](https://codedocs.xyz/CEED/libCEED/md_doc_libCEEDapi.html).

Clone or download libCEED by running

```
$ git clone https://github.com/CEED/libCEED.git
```

then move to the libCEED's directory by running
```
$ cd libCEED
```

and compile the library by running
```
$ make
```


We are going to look at some libCEED's examples that use some PETSc's capabilities 
(e.g., process partitioning and geometry handling).

Check out my branch for the demo where I made a couple of changes to print more info for the tutorial.

```
$ git checkout valeria/CUHPSC-demo
```

Move to the PETSc's examples folder by running
```
$ cd examples/petsc
```

And compile the examples by running 
```
$ make
```


-----

To run the example solving the Poisson's equation on a structured grid, use
```
$ ./bpsraw -ceed /cpu/self/ref/serial -problem bp3 -degree 1 -local 10000

```

See what happens when you run this in parallel, let's say with 2 processes
```
$ mpiexec -n 2 ./bpsraw -ceed /cpu/self/ref/serial -problem bp3 -degree 1 -local 10000

```
Instead, you can keep the total amount of work roughly constant, when you request more processes, 
but divide the local size of the problem so that each process works roughly the same. For instance, compare 
```
$ ./bpsraw -ceed /cpu/self/ref/serial -problem bp3 -degree 1 -local 10000
```

with 

```
$ mpiexec -n 4 ./bpsraw -ceed /cpu/self/ref/serial -problem bp3 -degree 1 -local 2500
```

-----

### The multigrid example
This example solves the same problem, but by using a preconditioning strategy. We use Chebchev as the smoother (solver) 
with Jacobi as the preconditioner for the smoother.

Run

```
$ ./multigrid -ceed /cpu/self/ref/serial -problem bp3  -cells 1000
```
and 

```
$ mpiexec -n 4 ./multigrid -ceed /cpu/self/ref/serial -problem bp3  -cells 1000
```

What do you see? 

Now let's raise the degree (accuracy of solution). 
This will also raise the number of neighboring points we need information from, i.e., the number of nodes.

Compare

```
$ ./multigrid -ceed /cpu/self/ref/serial -problem bp3 -degree 8 
```

With
```
$ ./multigrid -ceed /cpu/self/ref/serial -problem bp3 -degree 8 -coarsen logarithmic
```

Without specifying a coarsening strategy, it defaults to `-coarsen uniform`. 
This way, the domain is partitioned from finest grid to coarsest grid in a linear fashion, i.e.,
with `-degree 8`, we run all intermediate levels given by

8->7->6->5->...->2->1

Instead, when we use `-coarsen logarithmic` we have fewer subdivisions, using only powers of 2 as intermediate levels

8->4->2->1

-----

Collect your experiments data and try to plot the accuracy gained 
(given by the error, when the actual solution is available, otherwise by the digits of precision gained) vs time to solve
