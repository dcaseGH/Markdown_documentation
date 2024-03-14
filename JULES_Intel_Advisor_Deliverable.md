***************************************************
Deliverable JULES Profiling
***************************************************

Overview of deliverable
--------------
- JULES is a well established code, with hybrid MPI/OpenMP parallelization, and hold an important place in NWP and climate modelling as well as being a stand-alone model in its own right
- The Barcelona Supercomputing Centre have run their POP analysis, concluding that MPI communication is not a limiting factor, although they point to IO as a possible cause of performance loss [cite?]
- IO is being considered in a separate ExaJules workpackage
- The original proposal called for: ::
 “A first step will be to profile these regions to get a grip on how much work is done in each, and spend some time manually inspecting the regions which do the most work to sort and categorise the nature of the looping. These kernels and categories are likely where we can get the most optimisation value.” 
- In the context of the above, this first deliverable seeks to provide an overview of the perfomance of the code at the loop level, to consider the implications for the compiler, and to make suggestions to be explored for the rest of this workpackage
- The Intel Advisor tool (https://www.intel.com/content/www/us/en/developer/tools/oneapi/advisor.html) is useful in this respect, and will
- However it is noted that the test cases which will be used for the final deliverable are still to be defined, so any numbers in this document are to be considered indicative of JULES behavious but not results themselves.
- This initial work is to survey the 'ammunition' that will be fired at the benchmarks, so that good ideas can be tested in the coming deliverables for this project.

Overview of Intel Advisor on ARCHER2 and JASMIN
--------------
- picture
- Explanation
- 
- GUI - overview

First ideas for code changes
--------------



Further observations and compiler flag changes
--------------
- JULES is used in critical applications, such as NWP, for which performance isn't the main concern. The default flags (Intel) are -O2 
- It is noticable that a lot of time is spent calculating expensive functions, such as LOG, EXP etc. Whilst this is inherent in JULES equations, options exist to reduce
- A lot of the code, even without being rewritten, could be compiled with instructions which are specific to the architecture


Proposed further work
--------------

