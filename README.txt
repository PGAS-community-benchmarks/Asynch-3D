                           Asynch_3d
                          Version 0.9
                             README

                          February 2015

==============================================================================
Table of contents
==============================================================================

 1.  Overview
 
 2.  Contents of this Distribution
 3.  Hardware and Software Requirements
 4.  Configuration
 5.  Related Documentation - MPI (Message Passing Interface)
 6.  Documentation - GASPI (Global Address Space Programming Interface)
 7.  Implementation details
 8.  Results
 9.  Community involvement

==============================================================================
1. Overview
==============================================================================

This is the release of the Asynch_3d proxy.  This kernel is an attempt at
establishing a small but meaningful proxy/benchmark for notification based 
1-sided communication. 

The kernel calculates a recursive stencil via pipelined parallelism. While 
this works surprisingly well for a shared memory architecture

      array_ELEM(i,j,k) = (array_ELEM(i-1,j,k)
                         + array_ELEM(i,j-1,k) 
                         + array_ELEM(i,j,k-1)) * ONE_THIRD + 1;

the corresponding implementation for distributed memory systems is 
rather challenging.


==============================================================================
2.  Contents of this Distribution
==============================================================================

This release includes source code for MPI and GASPI implementations 
in the ./mpi and ./gaspi directories. 
  
==============================================================================
3. Hardware and Software Requirements
==============================================================================

1) Server platform with InfiniBand HCA or ROCE Ethernet.

2) Linux operating system 

3) MPI. Download and install. This kernel requires MPI to be interoperable 
   with GASPI. So far this has been verified to work with with Openmpi, 
   Mvapich, Mvapich2, Intel MPI and IBM Platform MPI. 

4) GASPI. For installation and download instructions of GPI2 (currently 
   the only full implementation of GASPI) please see 
   https://github.com/cc-hpc-itwm/GPI-2

==============================================================================
4. Configuration
==============================================================================

1) Download and install the required software packages, MPI, GPI2.
   Note : GPI2 needs to be compiled against your MPI library of choice in 
   order to use the  MPI interoperabilty mode. 
   (./install.sh --with-mpi=/../../foo -p PREFIX)

2) Adapt the Makefiles. (MPI_DIR, GPI2_DIR, CFLAGS, etc). The MPI lib will 
   require support for MPI_THREAD_MULTIPLE.
 
3) GASPI can make use of the startup mechanisms of MPI. Start the
   right_synch_p2p_dataflow.exe hybrid MPI/GASPI executable as a regular 
   hybrid OpenMP/MPI application, e.g  
   mpirun -np 2 -machinefile machines -perhost 2 ./right_synch_p2p_dataflow.exe

==============================================================================
5. MPI
==============================================================================

   For MPI Documentation
   http://www.mpi-forum.org/docs/mpi-3.0/mpi30-report.pdf

==============================================================================
6. GASPI
==============================================================================

   For GASPI Documentation
   http://www.gpi-site.com/gpi2/gaspi/

==============================================================================
7. Implementation details
==============================================================================

The MPI implementation is based on MPI_Send/MPI_Recv pairs. 
While this is clearly suboptimal for this type of  benchmark, it is not 
entirely obvious how to do this efficiently with the current MPI-3 standard. 

The GASPI implementation uses notifications. Even though the implementation 
looks similar to the MPI implementation, the actual underlying pattern
is completely different. In the GASPI implementation, the process which
starts the parallel pipeline can potentially issue several thousands 
(nThreads * K-dim) of write  notifications, before the (right) neighbouring 
rank has even finished the first iteration. In this case the parallel pipeline 
will never be stalled for lack of required data from the (left) pipeline 
neighbour.

==============================================================================
8. Results
==============================================================================

See wiki: https://github.com/PGAS-community-benchmarks/Asynch-3D/wiki#results

==============================================================================
9. Community involvement
==============================================================================

We encourage the HPC community to provide new patterns or improve existing
ones. No restrictions apply. Instead we encourage you to improve
this kernel via better threadings models, and/or better communication and/or
more scalable communication API’s and/or better programming languages. Bonus
points are granted for highly scalable implementations which also feature
better programmability.



