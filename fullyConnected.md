# Optimizing Dense/Fully Connected Operation

six unprivileged CSRs (`vstart`, vxsat, vxrm, `vtype`, `vl`, vlenb) to a base scalar RISC-V ISA.

vstart : Vector start position
vl : Vector length
vtype: Vector data type register
vlenb : VLEN/8 (vector register length in bytes)

The `vsetvli` instruction sets the `vtype` and `vl` CSRs based on its arguments, and writes the new value of `vl` into `rd`.

ELEN : The maximum size of a single vector element in bits
VLEN : vector register length in bits
SLEN : The striping distance in bits (VLEN ≥ SLEN ≥ 32)
LMUL : number of vector registers in a group
AVL : application vector length
```
vect_add:    
	.globl   vect_add
	vsetvli t0, a0, e8, m4, d1 			# Set vector length based on 8-bit vectors, group 4 VecReg together for efficiency 
	vle.v v4, (a1)          			# Get first vector
	  add a1, a1, t0         			# Bump pointer
	vle.v v8, (a2)          			# Get second vector
	  add a2, a2, t0       			  	# Bump pointer    
	vsadd.vv v12, v4, v8     			# Sum vectors    
	  sub a0, a0, t0        		 	# Decrement number done
	vse.v v12, (a3)         		 	# Store result      
      add a3, a3, t0       			 	# Bump pointer      
      bnez a0, vect_add   				# Loop back      
      ret                  			  	# Finished
```

m4 : `vlmul` - Vector Register Grouping
d1 : `vediv` - Divided Element Extension. The divided element extension allows each element to be treated as a packed sub-vector of narrower elements.

![fully-connected](img/fully-connected.png)


N =30, out = 40

Standard
```
 Number of cycles to run just vector_operations: 2007

 Number of cycles to run just matrix/pooling_operations: 112948

 Number of cycles to run just conv2D_operations: 1989963

 Number of cycles to run just fully_connected_operations: 15234

 Number of cycles to run all NN_operations: 2104918

 Total Number of cycles to run testbenchs: 11763850
```
 
Vector
```
 Number of cycles to run just vector_operations: 482

 Number of cycles to run just matrix/pooling_operations: 50317

 Number of cycles to run just conv2D_operations: 314753

 Number of cycles to run just fully_connected_operations: 7033

 Number of cycles to run all NN_operations: 365552

 Total Number of cycles to run testbenchs: 9971403
```

Speedup : x2.166074222

N = 512, out = 512

Standard
```
 Number of cycles to run just vector_operations: 2007

 Number of cycles to run just matrix/pooling_operations: 112948

 Number of cycles to run just conv2D_operations: 1989963

 Number of cycles to run just fully_connected_operations: 3156002

 Number of cycles to run all NN_operations: 2104918

 Total Number of cycles to run testbenchs: 32436924
```

Vector

```
 Number of cycles to run just vector_operations: 482

 Number of cycles to run just matrix/pooling_operations: 50317

 Number of cycles to run just conv2D_operations: 314753

 Number of cycles to run just fully_connected_operations: 1224737

 Number of cycles to run all NN_operations: 365552

 Total Number of cycles to run testbenchs: 28723174
```

Speedup : x2.576881404

