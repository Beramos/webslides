// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
Create time

Create mesh for time = 0


SIMPLE: no convergence criteria found. Calculations will run for 300 steps.

without Debug:

#0  Foam::error::printStack(Foam::Ostream&) at ??:?
#1  Foam::sigFpe::sigHandler(int) at ??:?
#2  ? in "/lib/x86_64-linux-gnu/libc.so.6"

with Debug:

Program received signal SIGFPE, Arithmetic exception.
0x00007ffff6668e5f in Foam::divide<double> (res=..., f1=...,
    s2=@0xa8b428: 0)
    at /home/bram/OpenFOAM/openfoam4_debug/src/OpenFOAM/lnInclude/FieldFunctions.C:631
631	BINARY_TYPE_OPERATOR_FS(Type, Type, scalar, /, divide)


notice: specific useful Info, e.g. file location!

without Debug:

meme handen in het haar

with Debug:

https://wutcode.files.wordpress.com/2013/12/31804326.jpg?w=284&h=284

Still hard but let's introduce gdb

Let's see what is going on, let's get the error stack:

>> (gdb) where
#0  0x00007ffff6668e5f in Foam::divide<double> (res=..., f1=...,
    s2=@0xa8b428: 0)
    at /home/bram/OpenFOAM/openfoam4_debug/src/OpenFOAM/lnInclude/FieldFunctions.C:631
#1  0x00007ffff668a0d3 in Foam::operator/<double> (tf1=...,
    s2=@0xa8b428: 0)
    at /home/bram/OpenFOAM/openfoam4_debug/src/OpenFOAM/lnInclude/FieldFunctions.C:631
#2  0x00007fffec2b3168 in Foam::fluxBalanceFvPatchScalarField::updateCoeffs (this=0xa8b0c0) at fluxBalanceFvPatchScalarField.C:269
#3  0x000000000046331f in Foam::GeometricField<double, Foam::fvPatchField, Foam::volMesh>::Boundary::updateCoeffs (this=0x7fffffffa548)
    at /home/bram/OpenFOAM/openfoam4_debug/src/OpenFOAM/lnInclude/GeometricBoundaryField.C:409

So I'm working on a boundary condition i implemented fluxBalance and It seems that there is something wrong here, makes sense.

Let's put a breakpoint on that specific line:

>> (gdb) b fluxBalanceFvPatchScalarField.C:269

and rerun the program

>> (gdb) run

Breakpoint 1, Foam::fluxBalanceFvPatchScalarField::updateCoeffs (this=0xa8b0c0) at fluxBalanceFvPatchScalarField.C:269
269	    this->refGrad() = -1./D_ * (i_.component(0)*T_Mem_/z_/F_ - i_.component(0)*transportNum/z_/F_); // this line is equivalent to //mixedFvPatchScalarField::refGrad() =...

code I wrote yesterday:

```c++
this->refGrad() = -1./D_ * (i_.component(0)*T_Mem_/z_/F_ - i_.component(0)*transportNum/z_/F_);
*

```

let's check what's in these variables:
>> (gdb) call D_
$1 = 9.9999999999999995e-07

>> call T_Mem_/z_/F_
$4 = inf

Hmm ok that's odd

>> (gdb) call z_
$5 = 0

oops...

Cause, I forgot to give the value of z at the boundary condition and the default value is set to 0...


Other ways to use GDB:

for developing applications:
Sandboxing with autocomplete

what are the member functions of the function where i'm programming

set breakpoint

>> gdb icoFoam
>> (gdb) b fluxBalanceFvPatchScalarField.C:260

use tab complete

>> (gdb) call this->patch.<tab>

>>Cf
Cn
New
Sf
_vptr.fvPatch
boundaryMesh
boundaryMesh_
constraintType
constraintTypes
constructpolyPatchConstructorTables
coupled
debug
delta
...
 
# References
* http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2008/debugging.pdf
* http://www.cimec.org.ar/ojs/index.php/mc/article/viewFile/3244/3167
* http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2007/FabianPengKarrholm/gdbfoam.pdf
* Stack overflow


VSCCode:
* Info on implementation of functions without the need of searchign in all the declaration on how it is implemented
* Build in git diff tool is amazing
* Open source, more lightweight than atom when using a lot of packages, more active community
