***************************************************
Deliverable JULES Profiling
***************************************************

Overview of deliverable
--------------
- JULES is a well established code, with hybrid MPI/OpenMP parallelization, and hold an important place in NWP and climate modelling as well as being a stand-alone model in its own right
- The Barcelona Supercomputing Centre have run their POP analysis, concluding that MPI communication is not a limiting factor, although they point to IO as a possible cause of performance loss [cite?]
- IO is being considered in a separate ExaJules workpackage
- The original proposal called for: 
```A first step will be to profile these regions to get a grip on how much work is done in each, and spend some time manually inspecting the regions which do the most work to sort and categorise the nature of the looping. These kernels and categories are likely where we can get the most optimisation value.``` 
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
- Single core performance, at root

First ideas for code changes
--------------
- A couple of diffs are provided (- is the old code, + is a new proposed change)
```
src/science/soil/calc_fsat_mod.F90
-        DO nti = 1,mti
-          ti_sc = (nti-0.5) * dti_sc
-          IF ((alf_ksat-1.0) * LOG(ti_sc) > LOG(TINY(ti_sc))) THEN
-            calc = (alf_ksat-1.0) * (LOG(ti_sc) + LOG(ti_sc_const))            &
-                   - (alf_ksat * ti_sc)
-            cum  = cum + EXP(calc)
-          END IF
-        END DO
+        ti_sc_vec = [((nti-0.5) * dti_sc, nti=1,mti )]
+        cum = sum(EXP((alf_ksat-1.0) * (LOG(ti_sc_vec) + LOG(ti_sc_const)) - (alf_ksat * ti_sc_vec)), &
+                  MASK=(alf_ksat-1.0) * LOG(ti_sc_vec) > LOG(TINY(ti_sc_vec)))
```
- Advisor ranks loops and functions which take the longest, and the above, in `calc_fsat` was the number one target for improvement.
- The code change, which uses a temporary vector and the intrinsic SUM/MASK syntax, above was tried and found to improve the self-time (time not calling other functions and subroutines) by 2.5s to 2.0s.
- This provides an example of using the tool, but going through the code suggesting tweaks is probably not the main thrust of this project

```
src/science/surface/root_frac_jls_mod.F90
-  DO n = 1,nlayer
-   z1 = z2
-   z2 = z2 + dz(n)
-   ztot = ztot + dz(n)
-   f_root(n) = EXP(-p * z1 / rootd) - EXP(-p * z2 / rootd)
-  END DO
-  ftot = 1.0 - EXP(-p * ztot / rootd)
-  DO n = 1,nlayer
-   f_root(n) = f_root(n) / ftot
-  END DO
+  temp_depth(0) = 0.0
+  DO n = 1,nlayer
+    temp_depth(n) = temp_depth(n-1) + dz(n)
+  END DO
+  ftot = 1.0 - EXP(-p * temp_depth(nlayer) / rootd)
+  DO n = 1,nlayer
+    f_root(n) = (EXP(- temp_depth(n-1)/rootd) -  EXP(- temp_depth(n)/rootd))/ftot
+  END DO
``` 
- Another loop which was highlighted is the above. In the original, the first loop is sequential and does a lot of work, which is expensive
- In the changed version a temporary vector is made sequentially (although this could presumably be precalculated), and then the iterations of the loop doing the work are independent.
- This lowers the cost from 0.6s to lower than 0.2s in total, but this example is probably of more use than the above. Getting loops which are independent of the order of calculation allows the compiler scope to optimize the code, especially in the light of the next section of this deliverable, and also opens more doors for parallelization of JULES in general.

Further observations and compiler flag changes
--------------
- JULES is used in critical applications, such as NWP, for which performance isn't the main concern. The default flags (Intel) are `-O2 `, although `-O3` has been used above.
- It is noticable that a lot of time is spent calculating expensive functions, such as LOG, EXP etc. Whilst this is inherent in JULES equations, options exist to reduce
- A lot of the code, even without being rewritten, could be compiled with instructions which are specific to the architecture
- By compiling with `-O3 -fast-transcendentals -fp-model fast=2 -xhost` the performance is better. Accepting that these changes will affect results slightly, 


Proposed further work
--------------

