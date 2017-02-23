# Atomic_Subroutines--Using_Integers_Efficiently
Fortran 2008 coarray programming with unordered execution segments (user-defined ordering) - Atomic Subroutines: Using Integers Efficiently

# Overview
This GitHub repository aims to show a simple programming technique that allows to transmit more than just one single integer scalar value with a single call to atomic_define (or atomic_ref).

# The limitation of atomic subroutines:
Fortran 2008 atomic subroutines (atomic_define / atomic_ref) do allow the use of single scalar integer values only with it. (This limitation is not a deficiency of the Fortran programming language itself, but merely due to hardware limitations.) That is fairly enough to program some simple one-sided customized (spin-wait loop) synchronization method, but does not meet the real-world requirements of customized (unordered) execution segment programming where we must be able to transmit exactly two integer values with a single call to atomic_define (or atomic_ref) at many times.

# Solution:
To overcome that limitation we use a simple programming technique from the past (that can be made bulletproof later on) : Storing two (limited-size) integer values into a single scalar integer variable. To do so, we use a simple pack/unpack technique.
The following code snippets show that basically.

Firstly, we define an integer-based enumeration using a Fortran 95 enumeration technique (that is explain here: https://github.com/MichaelSiehl/How-to-Code-Enumerations-in-Fortran) with a small modification: The first enum value contains the enum's step width. The example's enum step width value of 1000000 means that we can use an additional integer value up to 999999 to be packed together with the enum value:<br />
!___________________________________________________________<br />
!<br />
!*****************************************<br />
!****  ImageActivityFlag - Enumeration: **<br />
!*****************************************<br />
!<br />
type, private :: DontUse1<br />
&nbsp;&nbsp;integer :: Enum_StepWidth ! = 1000000<br />
&nbsp;&nbsp;integer :: InitializeSegmentSynchronization ! = 2000000<br />
&nbsp;&nbsp;integer :: WaitForSegmentSynchronization ! = 3000000<br />
&nbsp;&nbsp;integer :: ContinueSegmentSynchronization ! = 4000000<br />
end type DontUse1<br />
!<br />
type (DontUse1), public, parameter :: Enum_ImageActivityFlag &<br />
&nbsp;&nbsp;&nbsp;&nbsp;= DontUse1 (1000000,2000000,3000000,4000000)<br />
!____________________________________________________________<br />


Next, we use a simple routine to pack an integer enum value (intEnumValue) with an additional limited-size integer value (intAdditionalValue) into a single scalar integer variable (intPackedEnumValue) just by adding the two values: (the packed integer value will be used with atomic_define later on, not shown here)<br />
!__________________________________________________________<br />
! pack an integer ImageActivityFlag-enum value with an additional integer value:<br />
!**********<br />
subroutine PackEnumValue_ImageActivityFlag (intEnumValue, intAdditionalValue, intPackedEnumValue)<br />
&nbsp;&nbsp;! pack the both integer input arguments into a single integer scalar<br />
&nbsp;&nbsp;integer, intent (in) :: intEnumValue<br />
&nbsp;&nbsp;integer, intent (in) :: intAdditionalValue<br />
&nbsp;&nbsp;integer, intent (out) :: intPackedEnumValue<br />
&nbsp;&nbsp;integer :: intEnum_StepWidth<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;! check if the intAdditionalValue argument is to large:<br />
&nbsp;&nbsp;! (ToDo: check if it is negative)<br />
&nbsp;&nbsp;intEnum_StepWidth = Enum_ImageActivityFlag % Enum_StepWidth<br />
&nbsp;&nbsp;if (intAdditionalValue >= intEnum_StepWidth) then<br />
&nbsp;&nbsp;&nbsp;! the intAdditionalValue argument is too large<br />
&nbsp;&nbsp;&nbsp;! raise an error<br />
&nbsp;&nbsp;&nbsp;return<br />
&nbsp;&nbsp;end if<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;! pack the both values:<br />
&nbsp;&nbsp;intPackedEnumValue = intEnumValue + intAdditionalValue<br />
&nbsp;&nbsp;!<br />
end subroutine PackEnumValue_ImageActivityFlag<br />
!__________________________________________________________<br />


And finally, we need to unpack the packed integer after we've accessed it using atomic_ref (not shown here):<br />
!__________________________________________________________<br />
! unpack the packed integer enum value into two integer scalars:<br />
!***********<br />
subroutine UnpackEnumValue (intPackedEnumValue, intEnum_StepWidth, &<br />
&nbsp;&nbsp;&nbsp;&nbsp;intUnpackedEnumValue, intUnpackedAdditionalValue)<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;integer, intent (in) :: intPackedEnumValue<br />
&nbsp;&nbsp;integer, intent (in) :: intEnum_StepWidth<br />
&nbsp;&nbsp;integer, intent (out) :: intUnpackedEnumValue<br />
&nbsp;&nbsp;integer, intent (out) :: intUnpackedAdditionalValue<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;intUnpackedAdditionalValue = mod(intPackedEnumValue, intEnum_StepWidth)<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;intUnpackedEnumValue = intPackedEnumValue - intUnpackedAdditionalValue<br />
&nbsp;&nbsp;!<br />
end subroutine UnpackEnumValue<br />
!<br />
!**********<br />
!__________________________________________________________<br />
