# Atomic_Subroutines--Using_Integers_Efficiently
Fortran 2008 coarray programming with unordered execution segments (user-defined ordering) - Atomic Subroutines: Using Integers Efficiently

# Overview
This GitHub repository aims to show a simple programming technique that allows to transmit more than just one single integer scalar value with a single call to atomic_define (or atomic_ref).

# The limitation of atomic subroutines:
Fortran 2008 atomic subroutines (atomic_define / atomic_ref) do allow the use of single scalar integer values only with it. (This limitation is not a deficiency of the Fortran programming language itself, but merely due to hardware limitations.) That is fairly enough to program some simple one-sided customized (spin-wait loop) synchronization method, but does not meet the real-world requirements of customized (unordered) execution segment programming where we must be able to transmit exactly two integer values with a single call to atomic_define (or atomic_ref) at many times.

# Solution:
To overcome that limitation we use a simple programming technique from the past (that can be made bulletproof later on) : Storing two (limited-size) integer values into a single scalar integer variable. To do so, we use a simple pack/unpack technique.
The following code snippets show that basically.

Firstly, we define an integer-based enumeration using a Fortran 95 enumeration technique (that is explain here: https://github.com/MichaelSiehl/How-to-Code-Enumerations-in-Fortran) with a small modification: The first enum value contains the enum's step width. The example's enum step width value of 1000000 means that we can use an additional integer value up to 999999 to be packed together with the enum value:

