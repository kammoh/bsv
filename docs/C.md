## C AzureIP Foundation Libraries

Section B defined the standard Prelude package, which is automatically imported into every pack-
age. This section describes BSV’s large and continuously growing collection of AzureIP Foundation
libraries. To use any of these libraries in a package you must explicitly import the library package
using an `import` clause (Section 3).

Bluespec’s AzureIP intellectual property (IP) accelerates hardware design and modeling. All packages in the AzureIP Foundation library are provided as compiled code. Some of the packages are
also provided as BSV source code to facilitate customization. When modifying these files, first copy
them into a local directory and then modify your local copy. Use the `-p` flag when compiling, as
described in the BSV Users Guide, to include the local directory in your path.

There are two AzureIP library families, Foundation and Premium:

- Foundation is an extensive family of components, types and functions that are included with
    the Bluespec toolsets for use in your models and designs – they serve as a foundational base
    for your modeling and implementation work.
- Premium is the designation for Bluespec’s fee-based AzureIP.

### C.1 Storage Structures

#### C.1.1 Register File

Package

```bsv
import RegFile :: * ;
```

Description

This package provides 5-read-port 1-write-port register array modules.

Note: In a design that uses RegFiles, some of the read ports may remain unused. This may generate
a warning in various downstream tool. Downstream tools should be instructed to optimize away the
unused ports.

Interfaces and Methods

The `RegFile` package defines one interface, `RegFile`. The `RegFile` interface provides two methods,
`upd` and `sub`. The `upd` method is an `Action` method used to modify (or update) the value of an
element in the register file. The `sub` method (from ”sub”script) is a `Value` method which reads and
returns the value of an element in the register file. The value returned is of a datatype `data_t`.


|Interface Name | Parameter name | Parameter Description         | Restrictions                |
|---------------|----------------|-------------------------------|-----------------------------|
|RegFile        |  `indextype`   | datatype of the index         | must be in the `Bits` class |
|               |  `datat`       | datatype of the element values| must be in the `Bits` class |


```bsv
interface RegFile #(type index_t, type data_t);
method Action upd(index_t addr, data_t d);
method data_t sub(index_t addr);
endinterface: RegFile
```



| Method        |Arguments
|Name |Type    |Description Name Description
|`upd`| Action |Change or update an element within the register file.
```
```
addr index of the element to be
changed, with a datatype of
index_t
d new value to be stored, with a
datatype ofdata_t
sub datat Read an element from
the register file and re-
turn it.
```
```
addr index of the element, with a
datatype ofindex_t
```
Modules

TheRegFilepackage provides three modules:mkRegFilecreates a RegFile with registers allocated
from thelo_indexto thehi_index;mkRegFileFullcreates a RegFile from the minimum index to
the maximum index; andmkRegFileWCFcreates a RegFile fromlo_indextohi_indexfor which
the reads and the write are scheduled conflict-free. There is a second set of these modules, the
RegFileLoadvariants, which take as an argument a file containing the initial contents of the array.

```
mkRegFile Create a RegFile with registers allocated fromlo_indextohi_index.
lo_indexandhi_indexare of theindex_tdatatype and the elements
are of thedata_tdatatype.
```
```
module mkRegFile#( index_t lo_index, index_t hi_index )
( RegFile#(index_t, data_t) )
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data));
```
```
mkRegFileFull Create a RegFile from min to max index where the index is of a datatype
index_tand the elements are of datatypedata_t. The min and max are
specified by theBoundedtypeclass instance (0 to N-1 for N-bit numbers).
```
```
module mkRegFileFull#( RegFile#(index_t, data_t) )
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data)
Bounded#(index_t) );
```
```
mkRegFileWCF Create a RegFile fromlo_indextohi_indexfor which the reads and the
write are scheduled conflict-free. For the implications of this scheduling,
see the documentation forConfigReg(Section C.1.2).
module mkRegFileWCF#( index_t lo_index, index_t hi_index )
( RegFile#(index_t, data_t) )
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data));
```
TheRegFileLoadvariants provide the same functionality asRegFile, but each constructor function
takes an additional file name argument. The file contains the initial contents of the array using the
Verilog hex memory file syntax, which allows white spaces (including new lines, tabs, underscores,
and form-feeds), comments, binary and hexadecimal numbers. Length and base format must not be
specified for the numbers.


Reference Guide Bluespec SystemVerilog

The generated Verilog for file load variants contains$readmemband$readmemhconstructs. These
statements, as well as initial blocks generally, are considered simulation-only constructs because they
are not supported consistently across synthesis tools. Therefore, in the generated Verilog the initial
blocks are protected with atranslate_offdirective. When using a synthesis tool which supports
these constructs you can remove the directives to allow the tool to processes the$readmemhand
$readmembtasks during synthesis.

```
mkRegFileLoad Create a RegFile using the file to provide the initial contents of the array.
module mkRegFileLoad#
( String file, index_t lo_index, index_t hi_index)
( RegFile#(index_t, data_t) )
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data));
```
```
mkRegFileFullLoad Create a RegFile from min to max index using the file to provide the initial
contents of the array. The min and max are specified by theBounded
typeclass instance (0 to N-1 for N-bit numbers).
```
```
module mkRegFileFullLoad#( String file)
( RegFile#(index_t, data_t))
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data),
Bounded#(index_t) );
```
```
mkRegFileWCFLoad Create a RegFile fromlo_indextohi_indexfor which the reads and
the write are scheduled conflict-free (see Section C.1.2), using the file to
provide the initial contents of the array.
module mkRegFileWCFLoad#
( String file, index_t lo_index, index_t hi_index)
( RegFile#(index_t, data_t) )
provisos (Bits#(index_t, size_index),
Bits#(data_t, size_data));
```
Examples

UsemkRegFileLoadto create Register files and then read the values.

```
Reg#(Cntr) count <- mkReg(0);
```
```
// Create Register files to use as inputs in a testbench
RegFile#(Cntr, Fp64) vecA <- mkRegFileLoad("vec.a.txt", 0, 9);
RegFile#(Cntr, Fp64) vecB <- mkRegFileLoad("vec.b.txt", 0, 9);
```
```
//read the values from the Register files
rule drivein (count < 10);
Fp64 a = vecA.sub(count);
Fp64 b = vecB.sub(count);
uut.start(a, b);
count <= count + 1;
endrule
```

Bluespec SystemVerilog Reference Guide

Verilog Modules

RegFilemodules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name Defined in File
```
```
mkRegFile RegFile RegFile.v
mkRegFileFull
mkRegFileWCF
mkRegFileLoad RegFileLoad RegFileLoad.v
mkRegFileFullLoad
mkRegFileWCFLoad
```
#### C.1.2 ConfigReg

Package

import ConfigReg :: * ;

Description

TheConfigRegpackage provides a way to create registers where each update clobbers the current
value, but the precise timing of updates is not important. These registers are identical to themkReg
registers except that their scheduling annotations allows reads and writes to occur in either order
during rule execution.

Rules which fire during the clock cycle where the register is written read a stale value (that is the
value from the beginning of the clock cycle) regardless of firing order and writes which have occurred
during the clock cycle. Thus if ruler1writes to a ConfigRegcrand ruler2readscrlater in the
same cycle, the old or stale value ofcris read, not the value written inr1. If a standard register
is used instead, ruler2’s execution will be blocked byr1’s execution or the scheduler may create a
different rule execution order.

The hardware implementation is identical for the more common registers (mkReg,mkRegU and
mkRegA), and the module constructors parallel these as well.

Interfaces

TheConfigReginterface is an alias of theReginterface (section B.4.1).

typedef Reg#(a_type) ConfigReg #(type a_type);

Modules

TheConfigRegpackage provides three modules;mkConfigRegcreates a register with a given re-
set value and synchronous reset logic, mkConfigRegUcreates a register without any reset, and
mkConfigRegAcreates a register with a given reset value and asynchronous reset logic.

```
mkConfigReg Make a register with a given reset value. Reset logic is synchronous
```
```
module mkConfigReg#(a_type resetval)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```

Reference Guide Bluespec SystemVerilog

```
mkConfigRegU Make a register without any reset; initial simulation value is alternating
01 bits.
```
```
module mkConfigRegU(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```
```
mkConfigRegA Make a register with a given reset value. Reset logic is asynchronous.
```
```
module mkConfigRegA#(a_type, resetval)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```
```
Scheduling Annotations
mkConfigReg, mkConfigRegU, mkConfigRegA
read write
read CF CF
write CF SBR
```
#### C.1.3 DReg

Package

import DReg :: * ;

Description

TheDRegpackage allows a designer to create registers which store a written value for only a single
clock cycle. The value written to a DReg is available to read one cycle after the write. If more than
one cycle has passed since the register has been written however, the value provided by the register
is instead a default value (that is specified during module instantiation). These registers are useful
when wanting to send pulse values that are only asserted for a single clock cycle. The DReg is the
register equivalent of a DWire B.4.6.

Modules

TheDRegpackage provides three modules;mkDRegcreates a register with a given reset/default value
and synchronous reset logic,mkDRegUcreates a register without any reset (but which still takes a
default value as an argument), andmkDRegAcreates a register with a given reset/default value and
asynchronous reset logic.

```
mkDReg Make a register with a given reset/default value. Reset logic is syn-
chronous
```
```
module mkDReg#(a_type dflt_rst_val)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```
```
mkDRegU Make a register without any reset but with a specified default; initial
simulation value is alternating 01 bits.
```
```
module mkDRegU#(a_type dflt_val)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```

Bluespec SystemVerilog Reference Guide

```
mkDRegA Make a register with a given reset/default value. Reset logic is asyn-
chronous.
```
```
module mkDRegA#(a_type, dflt_rst_val)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```
```
Scheduling Annotations
mkDReg, mkDRegU, mkDRegA
read write
read CF SB
write SA SBR
```
#### C.1.4 RevertingVirtualReg

Package

import RevertingVirtualReg :: * ;

Description

TheRevertingVirtualRegpackage allows a designer to force a schedule when scheduling attributes
cannot be used. Since scheduling attributes cannot be put on methods, this allows a designer to
control the schedule between two methods, or between a method and a rule by adding a virtual
register between the two. The moduleRevertingVirtualRegcreates a virtual register; no actual
state elements are generated.

Modules

TheRevertingVirtualRegpackage provides the modulemkRevertingVirtualReg. The properties
of the module are:

- it schedules exactly like an ordinary register;
- it reverts to its reset value at the end of each clock cycle.

These imply that all allowed reads will return the reset value (since they precede any writes in the
cycle); thus the module neither needs nor instantiates any actual state element.

```
mkRevertingVirtualReg Creates a virtual register reverting to the reset value at the end of
each clock cycle.
```
```
module mkRevertingVirtualReg#(a_type rst)(Reg#(a_type))
provisos (Bits#(a_type, sizea));
```
```
Scheduling Annotations
mkRevertingVirtualReg
read write
read CF SB
write SA SBR
```
ExampleUsemkRevertingVirtualRegto create the execution order of therule followed by themethod


Reference Guide Bluespec SystemVerilog

Reg#(Bool) virtualReg <- mkRevertingVirtualReg(True);

rule the_rule (virtualReg); // reads virtualReg

endrule

method Action the_method;
virtualReg <= False; // writes virtualReg

endmethod

In a given cycle, reads always precede writes for a register. Therefore the reading ofvirtualReg
bythe_rulewill precede the writing ofvirtualReginthe_method. The execution order will be
the_rulefollowed bythe_method.

#### C.1.5 BRAM

Package

import BRAM :: * ;

Description

TheBRAMpackage provides types, interfaces, and modules to support FPGA BlockRams. The
BRAM modules include FIFO wrappers to provide implicit conditions for proper flow control for
the BRAM latency. Specific tools may determine whether modules are mapped to appropriate
BRAM cells during synthesis.

TheBRAMpackage is open-sourced and can be modified by the user. The low-level wrappers to
the BRAM Verilog and Bluesim modules, which are not open-sourced and cannot be modified, are
provided in theBRAMCorepackage, Section C.1.6.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

BRAMConfigure TheBRAM_Configurestructure specifies the underlying modules and their at-
tributes for instantiation. Default values for the BRAM are defined with theDefaultValueinstance
and can easily be modified.


Bluespec SystemVerilog Reference Guide

```
BRAMConfigure Structure
Field Type Description Allowed or
Recommended Values
memorySize Integer Number of words in the BRAM
latency Integer Number of stages in the read 1 (address is registered)
2 (address and data are
registered)
loadFormat LoadFormat Describes the load file None
tagged Hexfilename
tagged Binaryfilename
outFIFODepth Integer The depth of the BypassFIFO af-
ter the BRAM for the BRAMServer
module
```
```
latency+2
```
```
allowWriteResponseBypass
Bool Determines if write responses can di-
rectly be enqueued in the output fifo
(latency = 0for write).
```
The size of the BRAM is determined by thememorySizefield given in number of words. The width
of a word is determined by the polymorphic typedataspecified in the BRAM interface. If the
memorySizefield is 0, then memory size = 2n, wherenis the number of address bits determined
from the address type.

Thelatencyfield has two valid values; 1 indicates that the address on the read is registered, 2
indicates that both the address on the read input and the data on the read output are registered.
When latency = 2, the components in the dotted box in Figure 3 are included.

TheoutFIFODepthis used to determine the depth of the Bypass FIFO after the BRAM in the
mkBRAMServermodule. This value should belatency + 2to allow full pipeline behavior.

TheallowWriteResponseBypassfield, whenTrue, specifies that the write response is issued on the
same cycle as the write request. IfFalse, the write reponse is pipelined, which is the same behavior
as the read request. WhenTrue, the schedule constraints betweenputandgetareput SBR get.
Otherwise, the annotation isget CF put(no constraint).

typedef struct {Integer memorySize ;
Integer latency ; // 1 or 2 can extend to 3
LoadFormat loadFormat; // None, Hex or Binary
Integer outFIFODepth;
Bool allowWriteResponseBypass;
} BRAM_Configure ;

TheLoadFormatdefines the type of the load file (None,HexorBinary). The typeNoneis used
when there is no load file. When the type isHexorBinary, the name of the load file is provided as
aString.

typedef union tagged {
void None;
String Hex;
String Binary;
} LoadFormat
deriving (Eq, Bits);

The default values are defined in this package using theDefaultValueinstance forBRAM_Configure.
You can modify the default values by changing this instance or by modifying specific fields in your
design.


Reference Guide Bluespec SystemVerilog

```
Values defined in defaultValue
Field Type Value Meaning
memorySize Integer 0 2 n, wherenis the number of address
bits
latency Integer 1 address is registered
outFIFODepth Integer 3 latency + 2
loadFormat LoadFormat None no load file is used
allowWriteResponseBypass Bool False the write response is pipelined
```
instance DefaultValue #(BRAM_Configure);
defaultValue = BRAM_Configure {memorySize : 0
,latency : 1 // No output reg
,outFIFODepth : 3
,loadFormat : None
,allowWriteResponseBypass : False };
endinstance

To modify a default configuration for your design, set the field you want to change to the new value.
Example:

BRAM_Configure cfg = defaultValue ; //declare variable cfg
cfg.memorySize = 1024*32 ; //new value for memorySize
cfg.loadFormat = tagged Hex "ram.txt"; //value for loadFormat
BRAM2Port#(UInt#(15), Bit#(16)) bram <- mkBRAM2Server (cfg) ;
//instantiate 32K x 16 bits BRAM module

BRAMRequest TheBRAMpackage defines 2 structures for a BRAM request:BRAMRequest, and
the byte enabled versionBRAMRequestBE.

```
BRAMRequest Structure
Field Type Description
write Bool Indicates whether this operation is a write (True)or
a read (False).
responseOnWrite Bool Indicates whether a response should be received from
this write command
address addr Word address of the read or write
datain data Data to be written. This field is ignored for reads.
```
typedef struct {Bool write;
Bool responseOnWrite;
addr address;
data datain;
} BRAMRequest#(type addr, type data) deriving(Bits, Eq);

BRAMRequestBE The structureBRAMRequestBEallows for the byte enable signal.


Bluespec SystemVerilog Reference Guide

```
BRAMRequestBE Structure
Field Type Description
writeen Bit#(n) Byte-enable indicating whether this operation is a
write (n != 0) or a read (n = 0).
responseOnWrite Bool Indicates whether a response should be received from
this write command
address addr Word address of the read or write
datain data Data to be written. This field is ignored for reads.
```
typedef struct {Bit#(n) writeen;
Bool responseOnWrite;
addr address;
data datain;
} BRAMRequestBE#(type addr, type data, numeric type n) deriving (Bits, Eq);

Interfaces and Methods

The interfaces for the BRAM are built on theServerinterface defined in theClientServerpackage,
Section C.7.3. Some type aliases specific to the BRAM are defined here.

BRAM Server and Client interface types :

typedef Server#(BRAMRequest#(addr, data), data) BRAMServer#(type addr, type data);
typedef Client#(BRAMRequest#(addr, data), data) BRAMClient#(type addr, type data);

Byte-enabled BRAM Server and Client interface types:

typedef Server#(BRAMRequestBE#(addr, data, n), data)
BRAMServerBE#(type addr, type data, numeric type n);
typedef Client#(BRAMRequestBE#(addr, data, n), data)
BRAMClientBE#(type addr, type data, numeric type n);

TheBRAMpackage defines 1 and 2 port interfaces, with write-enabled and byte-enabled versions.
Each BRAM port interface contains aBRAMServer#(addr, data)subinterface and a clear action,
which clears the output FIFO of any pending requests. The data in the BRAM is not cleared.

```
BRAM1Port Interface
1 Port BRAM Interface
Name Type Description
portA BRAMServer#(addr, data) Server subinterface
portAClear Action Method to clear the portA output FIFO
```
interface BRAM1Port#(type addr, type data);
interface BRAMServer#(addr, data) portA;
method Action portAClear;
endinterface: BRAM1Port

```
BRAM1PortBE Interface
Byte enabled 1 port BRAM Interface
Name Type Description
portA BRAMServerBE#(addr, data, n) Byte-enabled server subinterface
portAClear Action Method to clear the portA output FIFO
```

Reference Guide Bluespec SystemVerilog

interface BRAM1PortBE#(type addr, type data, numeric type n);
interface BRAMServerBE#(addr, data, n) portA;
method Action portAClear;
endinterface: BRAM1PortBE

```
BRAM2Port Interface
2 port BRAM Interface
Name Type Description
portA BRAMServer#(addr, data) Server subinterface for port A
portB BRAMServer#(addr, data) Server subinterface for port B
portAClear Action Method to clear the port A output FIFO
portBClear Action Method to clear the port B output FIFO
```
interface BRAM2Port#(type addr, type data);
interface BRAMServer#(addr, data) portA;
interface BRAMServer#(addr, data) portB;
method Action portAClear;
method Action portBClear;
endinterface: BRAM2Port

```
BRAM2PortBE Interface
Byte enabled 2 port BRAM Interface
Name Type Description
portA BRAMServerBE#(addr, data, n) Byte-enabled server subinterface for port A
portB BRAMServerBE#(addr, data, n) Byte-enabled server subinterface for port B
portAClear Action Method to clear the portA output FIFO
portBClear Action Method to clear the portB output FIFO
```
interface BRAM2PortBE#(type addr, type data, numeric type n);
interface BRAMServerBE#(addr, data, n) portA;
interface BRAMServerBE#(addr, data, n) portB;
method Action portAClear;
method Action portBClear;
endinterface: BRAM2PortBE

Modules

The BRAM modules defined in theBRAMCorepackage (Section C.1.6) are wrapped with control
logic to turn the BRAM into a server, as shown in Figure 3. The BRAM Server modules include
an output FIFO and logic to control its loading and to avoid overflow. A single port, single clock
byte-enabled version is provided as well as 2 port and dual clock write-enabled versions.

```
mkBRAM1Server BRAM Server module including an output FIFO and logic to control
loading and to avoid overflow.
```
```
module mkBRAM1Server #( BRAM_Configure cfg )
( BRAM1Port #(addr, data) )
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
DefaultValue#(data) );
```

Bluespec SystemVerilog Reference Guide

```
Figure 3: 1 port of a BRAM Server
```
```
mkBRAM1ServerBE Byte-enabled BRAM Server module.
module mkBRAM1ServerBE #( BRAM_Configure cfg )
( BRAM1PortBE #(addr, data, n) )
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz),
DefaultValue#(data) );
```
```
mkBRAM2Server 2 port BRAM Server module.
```
```
module mkBRAM2Server #( BRAM_Configure cfg )
( BRAM2Port #(addr, data) )
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
DefaultValue#(data) );
```
```
mkBRAM2ServerBE Byte-enabled 2 port BRAM Server module.
```
```
module mkBRAM2ServerBE #( BRAM_Configure cfg )
( BRAM2PortBE #(addr, data, n) )
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```

Reference Guide Bluespec SystemVerilog

```
mkSyncBRAM2Server 2 port, dual clock, BRAM Server module. TheportAsubinterface and
portAClearmethods are in theclkAdomain; theportBsubinterface
andportBClearmethods are in theclkBdomain.
```
```
(* no_default_clock, no_default_reset *)
module mkSyncBRAM2Server #( BRAM_Configure cfg,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB
) (BRAM2Port #(addr, data) )
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
DefaultValue#(data) );
```
```
mkSyncBRAM2ServerBE 2 port, dual clock, byte-enabled BRAM Server module. TheportA
subinterface andportAClearmethods are in theclkAdomain; the
portBsubinterface andportBClearmethods are in theclkBdomain.
```
```
(* no_default_clock, no_default_reset *)
module mkSyncBRAM2ServerBE #(BRAM_Configure cfg,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB )
(BRAM2PortBE #(addr, data, n))
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```
Example: Using a BRAM

import BRAM::*;
import StmtFSM::*;
import Clocks::*;

function BRAMRequest#(Bit#(8), Bit#(8)) makeRequest(Bool write, Bit#(8) addr, Bit#(8) data);
return BRAMRequest{
write: write,
responseOnWrite:False,
address: addr,
datain: data
};
endfunction

(* synthesize *)
module sysBRAMTest();
BRAM_Configure cfg = defaultValue;
cfg.allowWriteResponseBypass = False;
BRAM2Port#(Bit#(8), Bit#(8)) dut0 <- mkBRAM2Server(cfg);
cfg.loadFormat = tagged Hex "bram2.txt";
BRAM2Port#(Bit#(8), Bit#(8)) dut1 <- mkBRAM2Server(cfg);

```
//Define StmtFSM to run tests
```

Bluespec SystemVerilog Reference Guide

```
Stmt test =
(seq
delay(10);
```
```
action
dut1.portA.request.put(makeRequest(False, 8’h02, 0));
dut1.portB.request.put(makeRequest(False, 8’h03, 0));
endaction
action
$display("dut1read[0] = %x", dut1.portA.response.get);
$display("dut1read[1] = %x", dut1.portB.response.get);
endaction
```
delay(100);
endseq);
mkAutoFSM(test);
endmodule

#### C.1.6 BRAMCore

Package

import BRAMCore :: * ;

Description

TheBRAMCorepackage, along with theBRAMpackage (Section C.1.5) provides types, interfaces, and
modules to support FPGA BlockRAMS. Specific tools may determine whether modules are mapped
to appropriate BRAM cells during synthesis.

Most designs should use the theBRAMpackage instead ofBRAMCore, as theBRAMpackage provides
implicit conditions provided by FIFO wrappers. TheBRAMCorepackage should be used only if you
want the low-level core BRAM modules without implicit conditions.

TheBRAMCorepackage contains the low-level wrappers to the BRAM Verilog and Bluesim modules.
Components are provided for single and dual port, byte-enabled, loadable, and dual clock versions.

Interfaces and Methods

TheBRAMCorepackage defines four variations of a BRAM interface to support single and dual port
BRAMs, as well as byte-enabled BRAMs.

TheBRAM_PORTinterface declares two methods; an Action methodput, and a value methodread.

TheBRAM_DUAL_PORTinterface is defined as twoBRAM_PORTsubinterfaces, one for each port.

```
BRAMPORT Interface
Method Arguments
Name Type Description Name Description
put Action Read or write values
in the BRAM.
```
```
write Write enable for the port; ifTruethe ac-
tion is write, ifFalse, the action is read.
address Index of the element, with a datatype of
addr.
datain Value to be written, with a datatype of
data. This value is ignored if the action
is read.
read data Returns a value of
typedata.
```

Reference Guide Bluespec SystemVerilog

interface BRAM_PORT#(type addr, type data);
method Action put(Bool write, addr address, data datain);
method data read();
endinterface: BRAM_PORT

interface BRAM_DUAL_PORT#(type addr, type data);
interface BRAM_PORT#(addr, data) a;
interface BRAM_PORT#(addr, data) b;
endinterface

Byte-enabled Interfaces

TheBRAM_PORT_BEandBRAM_DUAL_PORT_BEinterfaces are the byte-enabled versions of the BRAM
interfaces. In this version, the argumentwritenis of typeBit#(n), wherenis the number of byte-
enables. Your synthesis tools and targeted technology determine the restriction of data size and byte
enable size. Ifn= 0, the action is a read.

TheBRAM_DUAL_PORT_BEinterface is defined as twoBRAM_PORT_BEsubinterfaces, one for each port.

```
BRAMPORTBE Interface
Method Arguments
Name Type Description Name Description
put Action Read or write values
in the BRAM.
```
```
writeen Byte-enable for the port; ifn!= 0 write
the specified bytes, ifn= 0 read.
address Index of the elements to be read or writ-
ten, with a datatype ofaddr.
datain Value to be written, with a datatype of
data. This value is ignored if the action
is read.
read data Returns a value of
typedata.
```
(* always_ready *)
interface BRAM_PORT_BE#(type addr, type data, numeric type n);
method Action put(Bit#(n) writeen, addr address, data datain);
method data read();
endinterface: BRAM_PORT_BE

interface BRAM_DUAL_PORT_BE#(type addr, type data, numeric type n);
interface BRAM_PORT_BE#(addr, data, n) a;
interface BRAM_PORT_BE#(addr, data, n) b;
endinterface

Modules

TheBRAMCorepackage provides 1 and 2 port BRAM core modules, in both write-enabled and byte-
enabled versions. Note that there are no implicit conditions on the methods of these modules; if
these are required consider using the modules in theBRAMpackage (Section C.1.5).

TheBRAMCorepackage requires the caller to ensure the correct cycle to capture the read data, as
determined by thehasOutputRegisterflag. IfhasOutputRegisterisTrue, both the read address
and the read data are registered; ifFalse, only the read address is registered.

- If the output is registered (hasOutputRegisterisTrue), the latency is 2; the read data is
    available 2 cycles after the request.


Bluespec SystemVerilog Reference Guide

- If the output is not registered (hasOutputRegisterisFalse), the latency is 1; the read data
    is available 1 cycle after the request.

The other argument required ismemSize, anIntegerspecifying the memory size in number of words
of typedata.

The loadable BRAM modules require two additional arguments:

- fileis aStringcontaining the name of the load file.
- binaryis aBoolindicating whether the data type of the load file is binary (True) or hex
    (False).

```
mkBRAMCore1 Single port BRAM
```
```
module mkBRAMCore1#(Integer memSize,
Bool hasOutputRegister)
(BRAM_PORT#(addr, data))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz));
```
```
mkBRAMCore1BE Byte-enabled, single port BRAM.
```
```
module mkBRAMCore1BE#(Integer memSize,
Bool hasOutputRegister )
(BRAM_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz));
```
```
mkBRAMCore1Load Loadable, single port BRAM where the initial contents are infile.
The parameterbinaryindicates whether the contents of fileare
binary (True) or hex (False).
```
```
module mkBRAMCore1Load#(Integer memSize,
Bool hasOutputRegister,
String file, Bool binary )
(BRAM_PORT#(addr, data))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz) );
```
```
mkBRAMCore1BELoad Loadable, single port, byte-enabled BRAM.
```
```
module mkBRAMCore1BELoad#(Integer memSize,
Bool hasOutputRegister,
String file, Bool binary)
(BRAM_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```

Reference Guide Bluespec SystemVerilog

```
mkBRAMCore2 Dual port, single clock BRAM.
```
```
module mkBRAMCore2#(Integer memSize,
Bool hasOutputRegister )
(BRAM_DUAL_PORT#(addr, data))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz) );
```
```
mkBRAMCore2BE Byte-enabled, dual port BRAM.
```
```
module mkBRAMCore2BE#(Integer memSize,
Bool hasOutputRegister
) (BRAM_DUAL_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```
```
mkSyncBRAMCore2 Dual port, dual clock BRAM.
```
```
module mkSyncBRAMCore2#(Integer memSize,
Bool hasOutputRegister,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB )
(BRAM_DUAL_PORT#(addr, data))
provisos(Bits#(addr, addr_sz),Bits#(data, data_sz));
```
```
mkSyncBRAMCore2BE Dual port, dual clock byte-enabled BRAM.
```
```
module mkSyncBRAMCore2BE#(Integer memSize,
Bool hasOutputRegister,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB)
(BRAM_DUAL_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```
```
mkBRAMCore2Load Dual port, single clock, BRAM where the initial contents are infile.
The parameterbinaryindicates whether the contents of fileare
binary (True) or hex (False).
```
```
module mkBRAMCore2Load#(Integer memSize,
Bool hasOutputRegister,
String file, Bool binary)
(BRAM_DUAL_PORT#(addr, data))
provisos(Bits#(addr, addr_sz),Bits#(data, data_sz));
```

Bluespec SystemVerilog Reference Guide

```
mkBRAMCore2BELoad Dual port, single clock, byte-enabled BRAM where the initial contents
are infile. The parameterbinaryindicates whether the contents of
fileare binary (True) or hex (False).
```
```
module mkBRAMCore2BELoad#(Integer memSize,
Bool hasOutputRegister,
String file, Bool binary )
(BRAM_DUAL_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```
```
mkSyncBRAMCore2Load Dual port, dual clock BRAM with initial contents infile.
```
```
module mkSyncBRAMCore2Load#(Integer memSize,
Bool hasOutputRegister,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB,
String file, Bool binary)
(BRAM_DUAL_PORT#(addr, data))
provisos(Bits#(addr, addr_sz), Bits#(data, data_sz));
```
```
mkSyncBRAMCore2BELoadDual port, dual clock, byte-enabledBRAM with initial contents in
file.
```
```
module mkSyncBRAMCore2BELoad#(Integer memSize,
Bool hasOutputRegister,
Clock clkA, Reset rstNA,
Clock clkB, Reset rstNB,
String file, Bool binary)
(BRAM_DUAL_PORT_BE#(addr, data, n))
provisos(Bits#(addr, addr_sz),
Bits#(data, data_sz),
Div#(data_sz, n, chunk_sz),
Mul#(chunk_sz, n, data_sz) );
```
Verilog Modules

BRAMmodules correspond to the following Verilog modules, which are found in the Bluespec Verilog
library,$BLUESPECDIR/Verilog/.


Reference Guide Bluespec SystemVerilog

```
BSV Module Name Verilog Module Names
```
```
mkBRAMCore1 BRAM1.v
mkBRAMCore1Load BRAM1Load.v
mkBRAMCore1BE BRAM1BE.v
mkBRAMCore1BELoad BRAM1BELoad.v
mkBRAMCore2 BRAM2.v
mkSyncBRAMCore2
mkBRAMCore2BE BRAM2BE.v
mkSyncBRAMCore2BE
mkBRAMCore2Load BRAM2Load.v
mkSyncBRAMCore2Load
mkBRAMCore2BELoad BRAM2BELoad.v
mkSyncBRAMCore2BELoad
```
### C.2 FIFOs

#### C.2.1 FIFO Overview

The AzureIP Foundation library contains multiple FIFO packages. All library FIFO packages are
supplied as compiled code. The FIFOs in theBRAMFIFO,SpecialFIFO, andAlignedFIFOspackages
are also provided as BSV source code to facilate customization.

```
Package Name Description BSV Source Section
provided
FIFO Defines the FIFO interface and module constructors. FI-
FOs provided have implicit full and empty signals. In-
cludes pipeline FIFO (mkLFIFO).
```
##### C.2.2

```
FIFOF Defines the FIFOF interface and module constructors. FI-
FOs provided have explicit full and empty signals. Includes
pipeline FIFOF (mkLFIFOF).
```
##### C.2.2

```
FIFOLevel Enhanced FIFO interfaces and modules which include
methods to indicate the level or current number of items
stored in the FIFO. Single and dual clock versions are pro-
vided.
```
##### C.2.3

```
BRAMFIFO FIFOs which utilize the Xilinx Block RAMs.
```
##### √

##### C.2.4

```
SpecialFIFOs Additional pipeline and bypass FIFOs
```
##### √

##### C.2.5

```
AlignedFIFOs Parameterized FIFO module for creating synchronizing FI-
FOs between clock domains with aligned edges.
```
##### √

##### C.2.6

```
Gearbox FIFOs which change the frequency and data width of data
across clock domains with aligned edges. The overall data
rate stays the samme.
```
##### √

##### C.2.7

```
Clocks Generalized FIFOs to synchronize data being sent across
clock domains
```
##### C.9.7

#### C.2.2 FIFO and FIFOF packages

Packages


Bluespec SystemVerilog Reference Guide

import FIFO :: * ;
import FIFOF :: * ;

Description

The FIFO package defines theFIFOinterface and four module constructors. TheFIFOpackage is
for FIFOs with implicit full and empty signals.

TheFIFOFpackage defines FIFOs with explicit full and empty signals. The standard version ofFIFOF
has FIFOs with the enq, deq and first methods guarded by the appropriate (notFull or notEmpty)
implicit conditions for safety and improved scheduling. Unguarded (UG) versions of FIFOFare
available for the rare cases when implicit conditions are not desired. Guarded (G) versions ofFIFOF
are available which allow more control over implicit conditions. With the guarded versions the user
can specify whether the enqueue or dequeue side is guarded.

Type classes

FShow TheFIFOFtype belongs to theFShowtype class. AFIFOFcan be turned into aFmttype.
TheFmtvalue returned depends on the values of thenotEmptyandnotFullmethods.

```
FShow values for FIFOF
notEmpty notFull Fmt Object Example
True True <first> 3
True False <first> FULL 2 FULL
False True EMPTY EMPTY
False False EMPTY EMPTY
Note: <first>is the value of the first entry with the fshow function applied
```
Interfaces and methods

The four common methods,enq,deq,firstandclearare provided by both theFIFOandFIFOF
interfaces.

```
FIFO methods
Method Argument
Name Type Description Name Description
enq Action adds an entry to theFIFO x1 variable to be added to theFIFO
must be of typeelementtype
deq Action removes first entry from
theFIFO
first elementtype returns first entry the entry returned is of ele-
menttype
clear Action clears all entries from the
FIFO
```
interface FIFO #(type element_type);
method Action enq(element_type x1);
method Action deq();
method element_type first();
method Action clear();
endinterface: FIFO

FIFOFprovides two additional methods,notFullandnotEmpty.


Reference Guide Bluespec SystemVerilog

```
Additional FIFOF Methods
Name Type Description
notFull Bool returns a True value if there is space, you can enqueue an
entry into the FIFO
notEmpty Bool returns a True value if there are elements the FIFO, you
can dequeue from the FIFO
```
interface FIFOF #(type element_type);
method Action enq(element_type x1);
method Action deq();
method element_type first();
method Bool notFull();
method Bool notEmpty();
method Action clear();
endinterface: FIFOF

TheFIFOandFIFOFinterfaces belong to theToGetandToPuttypeclasses. You can use thetoGet
andtoPutfunctions to convertFIFOandFIFOFinterfaces toGetandPutinterfaces (Section C.7.1).

Modules

TheFIFOandFIFOFinterface types are provided by the module constructors: mkFIFO,mkFIFO1,
mkSizedFIFO,mkDepthParamFIFO, andmkLFIFO. EachFIFOis safe with implicit conditions; they do
not allow anenqwhen theFIFOis full or adeqorfirstwhen theFIFOis empty.

Most FIFOs do not allow simultaneous enqueue and dequeue operations when the FIFO is full or
empty. The exceptions are pipeline and bypass FIFOs. A pipeline FIFO (provided asmkLFIFO
in this package), allows simultaneous enqueue and dequeue operations when full. A bypass FIFO
allows simultaneous enqueue and dequeue operations when empty. Additional pipeline and bypass
FIFOs are provided in theSpecialFIFOspackage (Section C.2.5). The FIFOs in theSpecialFIFOs
package are provided as both compiled code and BSV source code, so they are customizable.

```
Allowed Simultaneous enq and deq
by FIFO type
FIFO Condition
FIFO type empty not empty full
not full
mkFIFO
```
##### √

```
mkFIFOF
mkFIFO1 NA
mkFIFOF1
mkLFIFO
```
##### √ √

```
mkLFIFOF
mkLFIFO1 NA
```
##### √

```
mkLFIFOF1
Modules provided in SpecialFIFOs package C.2.5
mkPipelineFIFO NA
```
##### √

```
mkPipelineFIFOF
mkBypassFIFO
```
##### √

##### NA

```
mkBypassFIFOF
mkSizedBypassFIFOF
```
##### √ √

```
mkBypassFIFOLevel
```
##### √ √

For creating aFIFOFinterface (providing explicitnotFullandnotEmptymethods) use the"F"
version of the module, for example usemkFIFOFinstead ofmkFIFO.


Bluespec SystemVerilog Reference Guide

```
Module Name BSV Module Declaration
For all modules,width_anymay be 0
FIFOorFIFOFof depth 2.
mkFIFO
mkFIFOF
```
```
module mkFIFO#(FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
```
FIFOorFIFOFof depth 1
mkFIFO1
mkFIFOF1
```
```
module mkFIFO1#(FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
```
FIFOorFIFOFof given depth n
mkSizedFIFO
mkSizedFIFOF
```
```
module mkSizedFIFO#(Integer n)(FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
```
FIFOorFIFOFof given depth n where n is a Verilog parameter or computed from
compile-time constants and Verilog parameters.
mkDepthParamFIFO
mkDepthParamFIFOF
```
```
module mkDepthParamFIFO#(UInt#(32) n)(FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
Unguarded (UG) versions ofFIFOFare available for the rare cases when implicit conditions are not
desired. When using an unguarded FIFO, the implicit conditions for correct FIFO operations are
NOT considered during rule and method processing, making it possible to enqueue when full and to
dequeue when empty. These modules provide theFIFOFinterface.

```
Unguarded FIFOF of depth 2
mkUGFIFOF
module mkUGFIFOF#(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
UnguardedFIFOFof depth 1
mkUGFIFOF1
module mkUGFIFO1#(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
UnguardedFIFOFof given depth n
mkUGSizedFIFOF
module mkUGSizedFIFOF#(Integer n)(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```

Reference Guide Bluespec SystemVerilog

```
UnguardedFIFOof given depth n where n is a Verilog parameter or computed from
compile-time constants and Verilog parameters.
mkUGDepthParamFIFOF
module mkUGDepthParamFIFOF#(UInt#(32) n)
(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
The guarded (G) versions of each of theFIFOFs allow you to specify which implicit condition you want
to guard. These modules takes two Boolean parameters;ugenqandugdeq. Setting either parameter
TRUEindicates the relevant methods (enqforugenq,firstanddeqforugdeq) are unguarded. If
both areTRUEtheFIFOFbehaves the same as an unguardedFIFOF. If both areFALSEthe behavior
is the same as a regularFIFOF.

```
GuardedFIFOFof depth 2.
mkGFIFOF module mkGFIFOF#(Bool ugenq, Bool ugdeq)(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
GuardedFIFOFof depth 1
mkGFIFOF1 module mkGFIFOF1#(Bool ugenq, Bool ugdeq)(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
GuardedFIFOFof given depth n
mkGSizedFIFOF module mkGSizedFIFOF#(Bool ugenq, Bool ugdeq, Integer n)
(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
GuardedFIFOFof given depth n where n is a Verilog parameter or computed from
compile-time constants and Verilog parameters.
mkGDepthParamFIFOF module mkGDepthParamFIFOF#(Bool ugenq, Bool ugdeq, UInt#(32) n)
(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
TheLFIFOs (pipeline FIFOs) allowenqanddeqin the same clock cycle when the FIFO is full.
Additional BSV versions of the pipeline FIFO and also bypass FIFOs (allowing simultaneousenq
anddeqwhen the FIFO is empty) are provided in theSpecialFIFOspackage (Section C.2.5). Both
unguarded and guarded versions of theLFIFOare provided in theFIFOFpackage.


Bluespec SystemVerilog Reference Guide

```
PipelineFIFOof depth 1.deqandenqcan be simultaneously applied in the same clock
cycle when theFIFOis full.
mkLFIFO
mkLFIFOF
mkUGLFIFOF
```
```
module mkLFIFO#(FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
```
Guarded pipelineFIFOFof depth 1.deqandenqcan be simultaneously applied in the same
clock cycle when theFIFOFis full.
```
```
mkGLFIFOF module mkGLFIFOF#(Bool ugenq, Bool ugdeq)(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
Functions

The FIFO package provides a functionfifofToFifoto convert an interface of typeFIFOFto an
interface of typeFIFO.

```
Converts a FIFOF interface to a FIFO interface.
```
```
fifofToFifo function FIFO#(a) fifofToFifo (FIFOF#(a) f);
```
Example using the FIFO package

This example creates 2 input FIFOs and moves data from the input FIFOs to the output FIFOs.

```
import FIFO::*;
```
```
typedef Bit#(24) DataT;
```
```
// define a single interface into our example block
interface BlockIFC;
method Action push1 (DataT a);
method Action push2 (DataT a);
method ActionValue#(DataT) get();
endinterface
```
```
module mkBlock1( BlockIFC );
Integer fifo_depth = 16;
```
```
// create the first inbound FIFO instance
FIFO#(DataT) inbound1 <- mkSizedFIFO(fifo_depth);
```
```
// create the second inbound FIFO instance
FIFO#(DataT) inbound2 <- mkSizedFIFO(fifo_depth);
```
```
// create the outbound instance
FIFO#(DataT) outbound <- mkSizedFIFO(fifo_depth);
```
```
// rule for enqueue of outbound from inbound1
```

Reference Guide Bluespec SystemVerilog

```
// implicit conditions ensure correct behavior
rule enq1 (True);
DataT in_data = inbound1.first;
DataT out_data = in_data;
outbound.enq(out_data);
inbound1.deq;
endrule: enq1
```
```
// rule for enqueue of outbound from inbound2
// implicit conditions ensure correct behavior
rule enq2 (True);
DataT in_data = inbound2.first;
DataT out_data = in_data;
outbound.enq(out_data);
inbound2.deq;
endrule: enq2
```
```
//Add an entry to the inbound1 FIFO
method Action push1 (DataT a);
inbound1.enq(a);
endmethod
```
```
//Add an entry to the inbound2 FIFO
method Action push2 (DataT a);
inbound2.enq(a);
endmethod
```
```
//Remove first value from outbound and return it
method ActionValue#(DataT) get();
outbound.deq();
return outbound.first();
endmethod
endmodule
```
Scheduling Annotations

Scheduling constraints describe how methods interact within the schedule. For example, aclearto
a given FIFO must be sequenced after (SA) anenqto the same FIFO. That is, when bothenqand
clearexecute in the same cycle, the resulting FIFO state is empty. For correct rule behavior the
rule executingenqmust be scheduled before the rule callingclear.

The table below lists the scheduling annotations for the FIFO modulesmkFIFO,mkSizedFIFO, and
mkFIFO1.

```
Scheduling Annotations
mkFIFO, mkSizedFIFO, mkFIFO1
enq first deq clear
enq C CF CF SB
first CF CF SB SB
deq CF SA C SB
clear SA SA SA SBR
```
The table below lists the scheduling annotations for the pipeline FIFO module,mkLFIFO. The pipeline
FIFO has a few more restrictions since there is a combinational path between thedeqside and the
enqside, thus restrictingdeqcalls beforeenq.


Bluespec SystemVerilog Reference Guide

```
Scheduling Annotations
mkLFIFO
enq first deq clear
enq C SA SAR SB
first SB CF SB SB
deq SBR SA C SB
clear SA SA SA SBR
```
TheFIFOFmodules add thenotFullandnotEmptymethods. These methods have SB annotations
with the Action methods that change FIFO state. These SB annotations model the atomic behavior
of a FIFO, that is whenenq,deq, orclearare called the state of notFullandnotEmptyare
changed. This is no different than the annotations onmkReg(which isreadSBwrite), where
actions are atomic and the execution module is one rule fires at a time. This does differ from a pure
hardware module of a FIFO or register where the state does not change until the clock edge.

```
Scheduling Annotations
mkFIFOF, mkSizedFIFOF, mkFIFOF1
enq notFull first deq notEmpty clear
enq C SA CF CF SA SB
notFull SB CF CF SB CF SB
first CF CF CF SB CF SB
deq CF SA SA C SA SB
notEmpty SB CF CF SB CF SB
clear SA SA SA SA SA SBR
```
Verilog Modules

FIFOandFIFOFmodules correspond to the following Verilog modules, which are found in the Blue-
spec Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Names Comments
```
```
mkFIFO
mkFIFOF
mkUGFIFOF
mkGFIFOF
```
```
FIFO2.v FIFO20.v
```
```
mkFIFO1
mkFIFOF1
mkUGFIFOF1
mkGFIFOF1
```
```
FIFO1.v FIFO10.v
```
```
mkSizedFIFO
mkSizedFIFOF
mkUGSizedFIFOF
mkGSizedFIFOF
```
```
SizedFIFO.v
FIFO1.v
FIFO2.v
```
```
SizedFIFO0.v
FIFO10.v
FIFO20.v
```
```
If the depth of the FIFO = 1,
then FIFO1.vandFIFO10.v
are used, if the depth = 2,
then FIFO2.vandFIFO20.v
are used.
```

Reference Guide Bluespec SystemVerilog

```
mkDepthParamFIFOF
mkUGDepthParamFIFOF
mkGDepthParamFIFOF
```
```
SizedFIFO.v SizedFIFO0.v
```
```
mkLFIFO
mkLFIFOF
mkUGLFIFOF
mkGLFIFOF
```
```
FIFOL1.v FIFOL10.v
```
#### C.2.3 FIFOLevel

Package

import FIFOLevel :: * ;

Description

The BSVFIFOLevellibrary provides enhanced FIFO interfaces and modules which include methods
to indicate the level or the current number of items stored in the FIFO. Both single clock and dual
clock (separate clocks for the enqueue and dequeue sides) versions are included in this package.

Interfaces and methods

TheFIFOLevelIfcinterface defines methods to compare the current level toIntegerconstants for
a single clock. TheSyncFIFOLevelIfcdefines the same methods for dual clocks; thus it provides
methods for both the source (enqueue) and destination (dequeue) clock domains. Instead of methods
to compare the levels, theFIFOCountIfcandSyncFIFOCountIfcdefine methods to return counts
of the FIFO contents, for single clocks and dual clocks respectively.

```
Interface Name Parameter
name
```
```
Parameter Description Requirements of modules
implementing the ifc
FIFOLevelIfc elementtype type of the elements stored
in theFIFO
```
```
must be inBitsclass
```
```
fifoDepth the depth of theFIFO must benumerictype and
> 2
FIFOCountIfc elementtype type of the elements stored
in theFIFO
```
```
must be inBitsclass
```
```
fifoDepth the depth of theFIFO must benumerictype and
> 2
SyncFIFOLevelIfc elementtype type of the elements stored
in theFIFO
```
```
must be inBitsclass
```
```
fifoDepth the depth of theFIFO must benumerictype and
must be a power of 2 and
>=2
SyncFIFOCountIfc elementtype type of the elements stored
in theFIFO
```
```
must be inBitsclass
```
```
fifoDepth the depth of theFIFO must benumerictype and
must be a power of 2 and
>=2
```
In addition to common FIFO methods, theFIFOLevelIfcinterface defines methods to compare the
current level toIntegerconstants. See Section C.2.2 for details onenq,deq,first,clear,notFull,


Bluespec SystemVerilog Reference Guide

andnotEmpty. Note thatFIFOLevelIfcinterface has a type parameter for thefifoDepth. This
numeric type parameter is needed, since the width of the counter is dependent on the FIFO depth.
ThefifoDepthparameter must be>2.

```
FIFOLevelIfc
Method Argument
Name Type Description Name Description
isLessThan Bool Returns True if the depth
of theFIFOis less than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant
```
```
isGreaterThan Bool ReturnsTrueif the depth of
theFIFOis greater than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant
```
interface FIFOLevelIfc#( type element_type, numeric type fifoDepth ) ;
method Action enq( element_type x1 );
method Action deq();
method element_type first();
method Action clear();

```
method Bool notFull ;
method Bool notEmpty ;
```
method Bool isLessThan ( Integer c1 ) ;
method Bool isGreaterThan( Integer c1 ) ;
endinterface

In addition to common FIFO methods, theFIFOCountIfcinterface defines a method to return the
current number of elements as an bit-vector. See Section C.2.2 for details onenq,deq,first,
clear,notFull, andnotEmpty. Note that theFIFOCountIfcinterface has a type parameter for the
fifoDepth. This numeric type parameter is needed, since the width of the counter is dependent on
the FIFO depth. ThefifoDepthparameter must be>2.

```
FIFOCountIfc
Method
Name Type Description
count UInt#(TLog#(TAdd#(fifoDepth,1))) Returns the number of items in theFIFO.
```
interface FIFOCountIfc#( type element_type, numeric type fifoDepth) ;
method Action enq ( element_type sendData ) ;
method Action deq () ;
method element_type first () ;

```
method Bool notFull ;
method Bool notEmpty ;
```
```
method UInt#(TLog#(TAdd#(fifoDepth,1))) count;
```
method Action clear;
endinterface


Reference Guide Bluespec SystemVerilog

The interfacesSyncFIFOLevelIfcandSyncFIFOCountIfcare dual clock versions of theFIFOLevelIfc
andFIFOCountIfc. Methods are provided for both source and destination clock domains. The fol-
lowing table describes the dual clocknotFullandnotEmptymethods, as well as the dual clock
clearmethods, which are common to both interfaces. Note that theSyncFIFOLevelIfcand
SyncFIFOCountIfcinterfaces each have a type parameter forfifoDepth. This numeric type pa-
rameter is needed, since the width of the counter is dependent on the FIFO depth. ThefifoDepth
parameter must be a power of 2 and>= 2.

```
Common Dual Clock Methods
Name Type Description
sNotFull Bool ReturnsTrueif theFIFOappears as not full from the
source side clock.
sNotEmpty Bool ReturnsTrueif theFIFOappears as not empty from the
source side clock.
dNotFull Bool ReturnsTrueif theFIFOappears as not full from the des-
tination side clock.
dNotEmpty Bool ReturnsTrueif theFIFOappears as not empty from the
destination side clock.
sClear Action Clears the FIFO from the source side.
dClear Action Clears the FIFO from the destination side.
```
In addition to common FIFO methods (Section C.2.2) and the common dual clock methods above,
theSyncFIFOLevelIfcinterface defines methods to compare the current level toIntegerconstants.
Methods are provided for both the source (enqueue side) and destination (dequeue side) clock do-
mains.

```
SyncFIFOLevelIfc Methods
Method Argument
Name Type Description Name Description
sIsLessThan Bool Returns True if the depth of
the FIFO, as appears on the
source side clock, is less than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant
```
```
sIsGreaterThan Bool ReturnsTrueif the depth of the
FIFO, as it appears on the source
side clock, is greater than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant.
```
```
dIsLessThan Bool ReturnsTrueif the depth of the
FIFO, as it appears on the desti-
nation side clock, is less than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant
```
```
dIsGreaterThan Bool ReturnsTrueif the depth of the
FIFO, as appears on the destina-
tion side clock, is greater than the
Integerconstant,c1.
```
```
c1 anIntegercompile-
time constant.
```
interface SyncFIFOLevelIfc#( type element_type, numeric type fifoDepth ) ;
method Action enq ( element_type sendData ) ;
method Action deq () ;
method element_type first () ;

```
method Bool sNotFull ;
method Bool sNotEmpty ;
```

Bluespec SystemVerilog Reference Guide

```
method Bool dNotFull ;
method Bool dNotEmpty ;
```
```
method Bool sIsLessThan ( Integer c1 ) ;
method Bool sIsGreaterThan( Integer c1 ) ;
method Bool dIsLessThan ( Integer c1 ) ;
method Bool dIsGreaterThan( Integer c1 ) ;
```
method Action sClear;
method Action dClear;
endinterface

In addition to common FIFO methods (Section C.2.2) and the common dual clock methods above,
theSyncFIFOCountIfcinterface defines methods to return the current number of elements. Methods
are provided for both the source (enqueue side) and destination (dequeue side) clock domains.

```
SyncFIFOCountIfc
Method
Name Type Description
sCount UInt#(TLog#(TAdd#(fifoDepth,1))) Returns the number of items in theFIFO
from the source side.
dCount UInt#(TLog#(TAdd#(fifoDepth,1))) Returns the number of items in theFIFO
from the destination side.
```
interface SyncFIFOCountIfc#( type element_type, numeric type fifoDepth) ;
method Action enq ( element_type sendData ) ;
method Action deq () ;
method element_type first () ;

```
method Bool sNotFull ;
method Bool sNotEmpty ;
method Bool dNotFull ;
method Bool dNotEmpty ;
```
```
method UInt#(TLog#(TAdd#(fifoDepth,1))) sCount;
method UInt#(TLog#(TAdd#(fifoDepth,1))) dCount;
```
method Action sClear;
method Action dClear;
endinterface

TheFIFOLevelIFC,SyncFIFOLevelIfc,FIFOCountIfc, andSyncFIFOCountIfcinterfaces belong
to theToGetandToPuttypeclasses. You can use thetoGetandtoPutfunctions to convert these
interfaces toGetandPutinterfaces (Section C.7.1).

Modules

The modulemkFIFOLevelprovides theFIFOLevelIfcinterface. Note that the implementation allows
any number ofisLessThanandisGreaterThanmethod calls. Each call with a unique argument
adds an additional comparator to the design.

There is also available a guarded (G) version ofFIFOLevelwhich takes three Boolean parameters;
ugenq,ugdeq, andugcount. Setting any of the parameters toTRUEindicates the method (enqfor
ugenq,deqforugdeq, andisLessThan,isGreaterThanforugcount) is unguarded. If all three are
FALSEthe behavior is the same as a regularFIFOLevel.


Reference Guide Bluespec SystemVerilog

```
Module Name BSV Module Declaration
mkFIFOLevel
module mkFIFOLevel (
FIFOLevelIfc#(element_type, fifoDepth) )
provisos( Bits#(element_type, width_element )
Log#(TAdd#(fifoDepth,1),cntSize) ) ;
```
```
Comment:width_elementmay be 0
```
```
Module Name BSV Module Declaration
mkGFIFOLevel
module mkGFIFOLevel#(Bool ugenq, Bool ugdeq, Bool ugcount)
( FIFOLevelIfc#(element_type, fifoDepth) )
provisos( Bits#(element_type, width_element ),
Log#(TAdd#(fifoDepth,1),cntSize));
```
```
Comment:width_elementmay be 0
```
The modulemkFIFOCountprovides the interfaceFIFOCountIfc. There is also available a guarded (G)
version ofFIFOCountwhich takes three Boolean parameters;ugenq,ugdeq, andugcount. Setting
any of the parameters toTRUEindicates the method (enqforugenq,deqforugdeq, andcountfor
ugcount) is unguarded. If all three areFALSEthe behavior is the same as a regularFIFOCount.

```
Module Name BSV Module Declaration
mkFIFOCount
module mkFIFOCount(
FIFOCountIfc#(element_type, fifoDepth) ifc )
provisos (Bits#(element_type, width_element));
```
```
Comment:width_elementmay be 0
```
```
Module Name BSV Module Declaration
mkGFIFOCount
module mkGFIFOCount#(Bool ugenq, Bool ugdeq, Bool ugcount)
( FIFOCountIfc#(element_type, fifoDepth) ifc )
provisos (Bits#(element_type, width_element));
```
```
Comment:width_elementmay be 0
```
The modulesmkSyncFIFOLevelandmkSyncFIFOCountare dual clock FIFOs, where enqueue and
dequeue methods are in separate clocks domains,sClkInanddClkInrespectively. Because of the
synchronization latency, the flag indicators will not necessarily be identical between the source and
the destination clocks. Note however, that thesNotFullanddNotEmptyflags always give proper
(pessimistic) indications for the safe use ofenqanddeqmethods; these are automatically included
as implicit condition in theenqanddeq(andfirst) methods.

The modulemkSyncFIFOLevelprovides theSyncFIFOLevelIfcinterface.


Bluespec SystemVerilog Reference Guide

```
Module Name BSV Module Declaration
mkSyncFIFOLevel
module mkSyncFIFOLevel(
Clock sClkIn, Reset sRstIn,
Clock dClkIn,
SyncFIFOLevelIfc#(element_type, fifoDepth) ifc )
provisos( Bits#(element_type, width_element),
Log#(TAdd#(fifoDepth,1),cntSize));
```
```
Comment:width_elementmay be 0
```
```
Figure 4: SyncFIFOCount
```
The modulemkSyncFIFOCount, as shown in Figure 4 provides theSyncFIFOCountIfcinterface.
Because of the synchronization latency, the count reports may be different between the source and
the destination clocks. Note however, that thesCountanddCountreports give pessimistic values
with the appropriate side. That is, the countsCount(on the enqueue clock) will report the exact
count of items in the FIFO or a larger count. The larger number is due to the synchronization
delay in observing the dequeue action. Likewise, thedCount(on the dequeue clock) returns the
exact count or a smaller count. The maximum disparity betweensCountanddCountdepends on
the difference in clock periods between the source and destination clocks.

The module providessClearanddClearmethods, both of which cause the contents of the FIFO
to be removed. Since the clears must be synchronized and acknowledged from one domain to the
other, there is a non-trivial delay before the FIFO recovers from the clear and can accept additional
enqueues or dequeues (depending on which side is cleared). The calling of either method immediately
disables other activity in the calling domain. That is, callingsClearin cyclencauses the enqueue
to become unready in the next cycle,n+1. Likewise, callingdClearin cyclencauses the dequeue to
become unready in the next cycle,n+1.


Reference Guide Bluespec SystemVerilog

After thesClearmethod is called, the FIFO appears empty on the dequeue side after threedClock
edges. ThreesClockedges later, the FIFO returns to a state where new items can be enqueued. The
latency is due to the full handshaking synchronization required to send the clear signal todClock
and receive the acknowledgement back.

For thedClearmethod call, the enqueue side is cleared in threesClkInedges and items can be
enqueued at the fourth edge. All items enqueued at or before the clear are removed from the FIFO.

Note that there is a ready signal associated with bothsClearanddClearmethods to ensure that
the clear is properly sent between the clock domains. Also,sRstInmust be synchronized with the
sClkIn.

```
Module Name BSV Module Declaration
mkSyncFIFOCount
module mkSyncFIFOCount(
Clock sClkIn, Reset sRstIn,
Clock dClkIn,
SyncFIFOCountIfc#(element_type, fifoDepth) ifc )
provisos( Bits#(element_type, width_element));
```
```
Comment:width_elementmay be 0
```
Example

The following example shows the use ofSyncFIFOLevelas a way to collect data into a FIFO, and
then send it out in a burst mode. The portion of the design shown, waits until the FIFO is almost
full, and then sets a register,burstOutwhich indicates that the FIFO should dequeue. When the
FIFO is almost empty, the flag is cleared, and FIFO fills again.

...
// Define a fifo of Int(#23) with 128 entries
SyncFIFOLevelIfc#(Int#(23),128) fifo <- mkSyncFIFOLevel(sclk, rst, dclk ) ;

```
// Define some constants
let sFifoAlmostFull = fifo.sIsGreaterThan( 120 ) ;
let dFifoAlmostFull = fifo.dIsGreaterThan( 120 ) ;
let dFifoAlmostEmpty = fifo.dIsLessThan( 12 ) ;
```
```
// a register to indicate a burst mode
Reg#(Bool) burstOut <- mkReg( False, clocked_by (dclk)) ;
```
...
// Set and clear the burst mode depending on fifo status
rule timeToDeque( dFifoAlmostFull &&! burstOut ) ;
    burstOut <= True ;
endrule

```
rule moveData ( burstOut ) ;
let dataToSend = fifo.first ;
fifo.deq ;
...
burstOut <= !dFifoAlmostEmpty;
```
```
endrule
```

Bluespec SystemVerilog Reference Guide

Scheduling Annotations

Scheduling constraints describe how methods interact within the schedule. The annotations for
mkFIFOLevelandmkSyncFIFOLevelare the same, except that methods in different domains (source
and destination) are always conflict free.

```
Scheduling Annotations
mkFIFOLevel,mkSyncFIFOLevel
enq first deq clear notFull notEmpty isLessThan isGreaterThan
enq C CF CF SB SA SA SA SA
first CF CF SB SB CF CF CF CF
deq CF SA C SB SA SA SA SA
clear SA SA SA SBR SA SA SA SA
notFull SB CF SB SB CF CF CF CF
notEmpty SB CF SB SB CF CF CF CF
isLessThan SB CF SB SB CF CF CF CF
isGreaterThan SB CF SB SB CF CF CF CF
```
The annotations formkFIFOCountandmkSyncFIFOCountare the same, except that methods in
different domains (source and destination) are always conflict free.

```
Scheduling Annotations
mkFIFOCount,mkSyncFIFOCount
enq first deq clear notFull notEmpty count
enq C CF CF SB SA SA SA
first CF CF SB SB CF CF CF
deq CF SA C SB SA SA SA
clear SA SA SA SBR SA SA SA
notFull SB CF SB SB CF CF CF
notEmpty SB CF SB SB CF CF CF
count SB CF SB SB CF CF CF
```
Verilog Modules

The modules described in this section correspond to the following Verilog modules, which are found
in the Bluespec Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Names
```
```
mkFIFOLevel
mkFIFOCount
```
```
SizedFIFO.v SizedFIFO0.v
```
```
mkSyncFIFOLevel
mkSyncFIFOCount
```
```
SyncFIFOLevel.v
```
#### C.2.4 BRAMFIFO

Package


Reference Guide Bluespec SystemVerilog

import BRAMFIFO :: * ;

Description

TheBRAMFIFOpackage provides FIFO interfaces and are built around a BRAM memory. The BRAM
is provided in theBRAMCorepackage described in Section C.1.6.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces

TheBRAMFIFOpackage providesFIFOF,FIFO, andSyncFIFOIfcinterfaces, as defined in theFIFOF,
FIFO, (both in Section C.2.2) andClocks(Section C.9.7) packages.

Modules

```
mkSizedBRAMFIFOF Provides aFIFOFinterface of a given depth,n.
```
```
module mkSizedBRAMFIFOF#(Integer n) (FIFOF#(element_type))
provisos (Bits(element_type, width_any),
Add#(1,z,width_any));
```
```
mkSizedBRAMFIFO Provides aFIFOinterface of a given depth,n.
```
```
module mkSizedBRAMFIFO#(Integer n)(FIFO#(element_type))
provisos(Bits#(t, width_element),
Add#(1, z, width_element) );
```
```
mkSyncBRAMFIFO Provides aSyncFIFOIfcinterface to send data across clock domains.
Theenqmethod is in the sourcesClkIndomain, while thedeqand
firstmethods are in the destinationdClkIndomain. The input and
output clocks, along with the input and output resets, are explicitly
provided. The default clock and reset are ignored.
```
```
module mkSyncBRAMFIFO#(Integer depth,
Clock sClkIn, Reset sRstIn,
Clock dClkIn, Reset dRstIn)
(SyncFIFOIfc#(element_type))
provisos(Bits#(element_type, width_element),
Add#(1, z, width_element));
```
```
mkSyncBRAMFIFOToCC Provides aSyncFIFOIfcinterface to send data from a second clock
domain into the current clock domain. The output clock and reset are
the current clock and reset.
```
```
module mkSyncBRAMFIFOToCC#(Integer depth,
Clock sClkIn, Reset sRstIn)
(SyncFIFOIfc#(element_type))
provisos(Bits#(element_type, width_element),
Add#(1, z, width_element));
```

Bluespec SystemVerilog Reference Guide

```
mkSyncBRAMFIFOFromCC Provides aSyncFIFOIfcinterface to send data from the current clock
domain into a second clock domain. The input clock and reset are the
current clock and reset.
```
```
module mkSyncBRAMFIFOFromCC#(Integer depth,
Clock dClkIn, Reset dRstIn)
(SyncFIFOIfc#(element_type))
provisos(Bits#(element_type, width_element),
Add#(1, z, width_element));
```
#### C.2.5 SpecialFIFOs

Package

import SpecialFIFOs :: * ;

Description

The SpecialFIFOs package contains various FIFOs provided as BSV source code, allowing users to
easily modify them to their own specifications. Included in the SpecialFIFOs package are pipeline
and bypass FIFOs. The pipeline FIFOs are equivalent to themkLFIFO(Section C.2.2); they allow
simultaneous enqueue and dequeue operations in the same clock cycle whenfull. The bypass FIFOs
allow simultaneous enqueue and dequeue in the same clock cycle whenempty. FIFOF versions,
with explicit full and empty signals, are provided for both pipeline and bypass FIFOs. The package
also includes theDFIFOF, a FIFOF with unguarded dequeue and first methods (thus they have no
implicit conditions).

```
FIFOs in Special FIFOs package
Module name Interface Description
mkPipelineFIFO FIFO 1 element pipeline FIFO; canenqanddeqsimultane-
ously when full.
mkPipelineFIFOF FIFOF 1 element pipeline FIFO with explicit full and empty
signals.
mkBypassFIFO FIFO 1 element bypass FIFO; canenqanddeqsimultane-
ously when empty.
mkBypassFIFOF FIFOF 1 element bypass FIFO with explicit full and empty
signals.
mkSizedBypassFIFOF FIFOF Bypass FIFO of given depth, with explicit full and
empty signals.
mkBypassFIFOLevel FIFOLevelIfc Same as aFIFOLevel(Section C.2.3), but canenq
anddeqwhen empty.
mkDFIFOF FIFOF A FIFOF with unguardeddeqandfirstmethods
where the first method returns specified default
value when the FIFO is empty.
```

Reference Guide Bluespec SystemVerilog

```
Allowed Simultaneous enq and deq
by FIFO type
FIFO Condition
FIFO type empty not empty full
not full
mkPipelineFIFO NA
```
##### √

```
mkPipelineFIFOF
mkBypassFIFO
```
##### √

##### NA

```
mkBypassFIFOF
mkSizedBypassFIFOF
```
##### √ √

```
mkBypassFIFOLevel
```
##### √ √

```
mkDFIFOF
```
##### √ √ √

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and methods

The modules defined in theSpecialFIFOspackage provide theFIFO,FIFOF, andFIFOLevelIfc
interfaces, as shown in the table above. These interfaces are described in Section C.2.2 (FIFO
package) and Section C.2.3 (FIFOLevel package).

Modules

```
Module Name BSV Module Declaration
1-element pipeline FIFO; canenqanddeqsimultaneously when full.
mkPipelineFIFO module mkPipelineFIFO (FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```
```
1-element pipeline FIFOF; canenqanddeqsimultaneously when full.
Has explicit full and empty signals.
mkPipelineFIFOF module mkPipelineFIFOF (FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
1-element bypass FIFO; canenqanddeqsimultaneously when empty.
mkBypassFIFO module mkBypassFIFO (FIFO#(element_type))
provisos (Bits#(element_type, width_any));
```

Bluespec SystemVerilog Reference Guide

```
1-element bypass FIFOF; canenqanddeqsimultaneously when empty.
Has explicit full and empty signals.
mkBypassFIFOF module mkBypassFIFOF (FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
Bypass FIFOF of given depthfifoDepthwith explicit full and empty signals.
mkSizedBypassFIFOF module mkSizedBypassFIFOF#(Integer fifoDepth)
(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
```
Bypass FIFOLevel of given depthfifoDepth
```
```
mkBypassFIFOLevel module mkBypassFIFOLevel(FIFOLevelIfc#(element_type,
fifoDepth))
provisos( Bits#(element_type, width_any),
Log#(TAdd#(fifoDepth,1), cntSize));
```
```
A FIFOF with unguardeddeqandfirstmethods (thus they have no implicit conditions).
Thefirstmethod returns a specified default value when the FIFO is empty
```
```
mkDFIFOF module mkDFIFOF#(element_type default_value)
(FIFOF#(element_type))
provisos (Bits#(element_type, width_any));
```
#### C.2.6 AlignedFIFOs

Package

import AlignedFIFOs :: * ;

Description

The AlignedFIFOs package contains a parameterized FIFO module intended for creating synchro-
nizing FIFOs between clock domains with aligned edges for both types of clock domain crossings:

- slow-to-fast crossing - every edge in the source domain implies the existence of a simultaneous
    edge in the destination domain
- fast-to-slow crossing - every edge in the destination domain implies the existence of a simulta-
    neous edge in the source domain


Reference Guide Bluespec SystemVerilog

The FIFO is parameterized on the type of store used to hold the FIFO data, which is itself param-
eterized on the index type, value type, and read latency. Modules to construct stores based on a
single register, a vector of registers and a BRAM are provided, and the user can supply their own
store implementation as well.

The FIFO allows the user to control whether or not outputs are held stable during the full slow
clock cycle or allowed to transition mid-cycle. Holding the outputs stable is the safest option but it
slightly increases the minimum latency through the FIFO.

A primary design goal of this FIFO is to provide an efficient and flexible family of synchronizing
FIFOs between aligned clock domains which are written in BSV and are fully compatible with
Bluesim. These FIFOs (particularly ones using vectors of registers) may not be the best choice for
ASIC synthesis due to the muxing to select the head value in thefirstmethod.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and methods

Store Interface TheAlignedFIFOis parameterized on the type of store used to hold the FIFO
data. The three types of stores provided in theAlignedFIFOpackage (single-element, vector-of-
registers, and BRAM) all return aStoreinterface.

TheStoreinterface has aprefetchmethod which is used by some modules (themkBRAMStorein
this package). If a prefetch is used, thereadmethod returns the value at the previously fetched
index; the value ofidxshould be ignored. If a prefetch is not used, thereadmethod index value
determines the returned value.

```
Store Interface Methods
Name Type Description
write Action Writes the value at indexidx.
prefetch Action Initiates a prefetch of the value at indexidx.
read a Returns the value of typea. If prefetch is not used, returns
the value at indexidx. When prefetch is used, returns the
value at the previously fetched index; the value ofidx
should be ignored.
```
interface Store#(type i, type a, numeric type n);
method Action write(i idx, a value);
method Action prefetch(i idx);
method a read(i idx);
endinterface: Store

AlignedFIFO Interface TheAlignedFIFOinterface provides methods for both source (enqueue)
and destination (dequeue) clock domains.


Bluespec SystemVerilog Reference Guide

```
AlignedFIFO Interface Methods
Name Type Description
enq Action Adds an entry to the FIFO from the source clock domain.
first a Returns the first entry from the FIFO in the destination
clock domain.
deq Action Removes the first entry from the FIFO in the destination
clock domain.
dNotFull Bool ReturnsTrueif the FIFO appears not full from the desti-
nation clock domain.
dNotEmpty Bool ReturnsTrueif the FIFO appears not empty from the
destination clock domain.
sNotFull Bool ReturnsTrueif the FIFO appears not full from the source
clock domain.
sNotEmpty Bool ReturnsTrueif the FIFO appears not empty from the
source clock domain.
dClear Action Clears the FIFO from the destination side.
sClear Action Clears the FIFO from the source side.
```
interface AlignedFIFO#(type a);
method Action enq(a x);
method a first();
method Action deq();
method Bool dNotFull();
method Bool dNotEmpty();
method Bool sNotFull();
method Bool sNotEmpty();
method Action dClear();
method Action sClear();
endinterface: AlignedFIFO

Modules

TheAlignedFIFOmodule is parameterized on the type of store used to hold the FIFO data. The
AlignedFIFOspackage contains modules to construct stores based on a single register (mkRegStore),
a vector of registers (mkRegVectorStore), and a BRAM (mkBRAMStore). Users can supply their own
store implementation as well.

ThemkRegStoreinstantiates a single-element store. The module returns aStoreinterface and does
not use a prefetch.

```
Module Name BSV Module Declaration
Implementation of a single-element store
mkRegStore module mkRegStore(Clock sClock, Clock dClock,
Store#(UInt#(0),a,0) ifc)
provisos(Bits#(a,a_sz) );
```
ThemkRegVectorStoremodule instantiates a vector-of-registers store. The module returns aStore
interface and does not use a prefetch.


Reference Guide Bluespec SystemVerilog

```
Implementation of a vector-of-registers store
mkRegVectorStore module mkRegVectorStore(Clock sClock, Clock dClock,
Store#(UInt#(w),a,0) ifc)
provisos( Bits#(a,a_sz) );
```
ThemkBRAMStore2W1Rmodule returns aStoreinterface and uses a prefetch. This model assumes
the read clock is a 2x divided version of the write clock.

```
A BRAM-based store where the read clock is a 2x divided version of the write clock.
mkBRAMStore2W1R module mkBRAMStore2W1R(Clock sClock, Reset sReset,
Clock dClock, Reset dReset,
Store#(i,a,1) ifc)
provisos( Bits#(a,a_sz), Bits#(i,w), Eq#(i) );
```
ThemkBRAMStore1W2Rmodule returns aStoreinterface and uses a prefetch. This model assumes
the write clock is a 2x divided version of the read clock.

```
A BRAM-based store where the write clock is a 2x divided version of read clock.
```
```
mkBRAMStore1W2R module mkBRAMStore1W2R(Clock sClock, Reset sReset,
Clock dClock, Reset dReset,
Store#(i,a,1) ifc)
provisos( Bits#(a,a_sz), Bits#(i,w), Eq#(i) );
```
ThemkAlignedFIFOmodule makes a synchronizing FIFO for aligned clocks, based on the given
backing store (determined by the type of store instantiated). The store is assumed to have 2wslots
addressed from 0 to 2w−1. The store will be written in the source clock domain and read in the
destination clock domain.

Theenqanddeqmethods will only be callable when theallow_enqandallow_deqinputs are high.
For a slow-to-fast crossing use:

allow_enq = constant True
allow_deq = pre-edge signal

For a fast-to-slow crossing, use:

allow_enq = pre-edge signal
allow_deq = constant True

The pre-edge signal isTruewhen the slow clock will rise in the next clock cycle. TheClockNextRdy
from theClockDividerIfc(Section C.9.3) can be used as the pre-edge signal.

These settings ensure that the outputs in the slow clock domain are stable for the entire cycle.
Setting both inputs to constant True reduces the minimum latency through the FIFO, but allows
outputs in the slow domain to transition mid-cycle. This is less safe and can interact badly with the
$displaysin a Verilog simulation.

It is not advisable to call bothdClearandsClearsimultaneously.


Bluespec SystemVerilog Reference Guide

```
Implementation of an aligned FIFO
mkAlignedFIFO (* no_default_clock, no_default_reset *)
module mkAlignedFIFO( Clock sClock
, Reset sReset
, Clock dClock
, Reset dReset
, Store#(i,a,n) store
, Bool allow_enq
, Bool allow_deq
, AlignedFIFO#(a) ifc
)
provisos( Bits#(a,sz_a), Bits#(i,w),
Eq#(i), Arith#(i) );
```
#### C.2.7 Gearbox

Package

import Gearbox :: *

Description

This package defines FIFO-like converters that convert N-wide data to and from 1-wide data at
N-times the frequency. These converters change the frequency and the data width, while the overall
data rate stays the same. The data width on the fast side is always 1, while the data width on the
slow side is N. The converters are intended to be used between clock domains with aligned edges for
both types of clock domain crossings (fast to slow and slow to fast). For example:

300 MHz at 8-bits converted to 100 MHz at 24-bits (fast to slow)
100 MHz at 24-bits converted to 300 MHz at 8-bits (slow to fast)

In both of these examples, the data typeaisBit#(8)and N=3.

These modules are written in pure BSV using a style that utilzies onlymkNullCrossingRegto cross
registered values between clock domains. Restricting the form of clock crossings is important to
ensure that the module preserves atomic semantics and also that it is compatible with both Verilog
and Bluesim backends.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and methods

TheGearboxinterface provides the following methods:enq,deq,first,notFullandnotEmpty.


Reference Guide Bluespec SystemVerilog

```
Gearbox Interface
Method
Name
```
```
Type Description
```
```
enq Action Adds an entry to the converter of typeVector#(in, a),
whereais the datatype. If the input is the fast domain then
in= 1, if the input is the slow domain,in= N.
deq Action Removes the first entry from the converter.
first Vector#(out, a) Returns the first entry from the converter. If the output
domain is the fast side,out= 1, if the output domain is the
slow side,out= N.
notFull Bool Returns a True value if there is space to enqueue an entry
into the FIFO.
notEmpty Bool Returns a True value if there is are elements in the FIFO
and you can dequeue from the FIFO.
```
interface Gearbox#(numeric type in, numeric type out, type a);
method Action enq(Vector#(in, a) din);
method Action deq();
method Vector#(out, a) first();
method Bool notFull();
method Bool notEmpty();
endinterface

Modules

The package provides two modules:mkNto1Gearboxfor slow to fast domain crossings, andmk1toNGearbox
to for fast to slow domain crossings. These are intended for use between clock domains with aligned
edges for both types of clock domain crossings.

Note: for both modules the resets in the source and destination domains (sResetanddReset)
should be asserted together, otherwise only half the unit will be in reset.

With themkNto1Gearboxmodule, 2xN elements of data storage are provided, grouped into 2 blocks
of N elements each. Each block is writable in the source (slow) domain and readable in the destination
(fast) domain.

```
mkNto1Gearbox Moves data from a slow domain to a fast domain, changing the data width
from a larger width to a smaller width. The data rate stays the same.
The width of the output is 1, the width of the input is N.
module mkNto1Gearbox(Clock sClock, Reset sReset,
Clock dClock, Reset dReset,
Gearbox#(in, out, a) ifc)
provisos(Bits#(a, a_sz), Add#(out, 0, 1),
Add#(out, z, in) );
```
With themk1toNGearboxmodule, 2xN elements of data storage are provided, grouped into 2 blocks
of N elements each. Each block is writable in the source (fast) domain and readable in the destination
(slow) domain.


Bluespec SystemVerilog Reference Guide

```
mk1toNGearbox Moves data from a fast domain to a slow domain, changing the data width
from a smaller width to a larger width. The data rate stays the same.
The width of the input is 1, the width of the output is N.
```
```
module mk1toNGearbox(Clock sClock, Reset sReset,
Clock dClock, Reset dReset,
Gearbox#(in, out, a) ifc)
provisos(Bits#(a, a_sz), Add#(in, 0, 1),
Add#(in, z, out), Mul#(2, out, elements),
Add#(1, w, elements), Add#(out, x, elements) );
```
#### C.2.8 MIMO

Package

import MIMO :: *

Description

This package defines a Multiple-In Multiple-Out (MIMO), an enhanced FIFO that allows the de-
signer to specify the number of objects enqueued and dequeued in one cycle. There are different
implementations of the MIMO available for synthesis: BRAM, Register, and Vector.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

TheLUInttype is aUIntdefined as the log of then.

typedef UInt#(TLog#(TAdd#(n, 1))) LUInt#(numeric type n);

TheMimoConfigurationtype defines whether theMIMOis guarded or unguarded, and whether it
is based on a BRAM. There is an instance in theDefaultValuetype class. The defaultMIMOis
guarded and not based on a BRAM.

typedef struct {
Bool unguarded;
Bool bram_based;
} MIMOConfiguration deriving (Eq);

instance DefaultValue#(MIMOConfiguration);
defaultValue = MIMOConfiguration {
unguarded: False,
bram_based: False
};
endinstance

Interfaces and methods

TheMIMOinterface is polymorphic and takes 4 parameters:max_in,max_out,size, andt.


Reference Guide Bluespec SystemVerilog

```
MIMO Interace Parameters
Name Description
max_in Maximum number of objects enqueued in one cycle. Must be numeric.
max_out Maximum number of objects dequeued in one cycle. Must be numeric.
size Total size of internal storage. Must be numeric.
t Data type of the stored objects
```
TheMIMOinterface provides the following methods:enq,first,deq,enqReady,enqReadyN,deqReady,
deqReadyN,count, andclear.

```
MIMO methods
Method Argument
Name Type Description
enq Action adds an entry to theMIMO LUInt#(max_in)
Vector#(max_in, t) data)
first Vector#(max_out, t) Returns a Vector containing
max_outitems of typet.
deq Action Removes the firstcountentries LUint#(max_out)
count
enqReady Bool Returns a True value if there is
space to enqueue an entry
enqReadyN Bool Returns a True value if there is
space to enqueuecountentries
```
```
LUint#(max_in)
```
```
deqReady Bool Returns a True value if there is
an element to dequeue
enqReadyN Bool Returns a True value if there is
arecountelements to dequeue
count LUint#(size) Returns the log of the number of
elements in theMIMO
clear Action Clears theMIMO
```
interface MIMO#(numeric type max_in, numeric type max_out, numeric type size, type t);
method Action enq(LUInt#(max_in) count, Vector#(max_in, t) data);
method Vector#(max_out, t) first;
method Action deq(LUInt#(max_out) count);
method Bool enqReady;
method Bool enqReadyN(LUInt#(max_in) count);
method Bool deqReady;
method Bool deqReadyN(LUInt#(max_out) count);
method LUInt#(size) count;
method Action clear;
endinterface

Modules

The package provides modules to synthesize different implementations of the MIMO: the basic
MIMO (mkMIMO), BRAM-based (mkMIMOBRAM), register-based (mkMIMOReg), and a Vector of registers
(mkMIMOV).

All implementations must meet the following provisos:

- The object must have bit representation


Bluespec SystemVerilog Reference Guide

- The object must have at least 2 elements of storage.
- The maximum number of objects enqueued (max_in) must be less than or equal to the total
    bits of storage (size)
- The maximum number of objects dequeued (max_out) must be less than or equal to the total
    bits of storage (size)

```
mkMIMO The basic implementation of MIMO. Object must be at least 1 bit in size.
module mkMIMO#(MIMOConfiguration cfg)(MIMO#(max_in, max_out, size, t));
```
```
mkMIMOBRAM Implementation of BRAM-based MIMO. Object must be at least 1 byte in size.
```
```
module mkMIMOBram#(MIMOConfiguration cfg)(MIMO#(max_in, max_out, size, t));
```
```
mkMIMOReg Implementation of register-based MIMO.
```
```
module mkMIMOReg#(MIMOConfiguration cfg)(MIMO#(max_in, max_out, size, t));
```
```
mkMIMOV Implementation of Vector-based MIMO. The ojbect must have a default value defined.
```
```
module mkMIMOV(MIMO#(max_in, max_out, size, t));
```
### C.3 Aggregation: Vectors

Package

import Vector :: * ;

Description

TheVectorpackage defines an abstract data type which is a container of a specific length, holding
elements of one type. Functions which create and operate on this type are also defined within this
package. Because it is abstract, there are no constructors available for this type (likeConsandNil
for theListtype).
typedef struct Vector#(type numeric vsize, type element_type);

Here, the type variableelement_typerepresents the type of the contents of the elements while the
numeric type variablevsizerepresents the length of the vector.

If the elements are in theBitsclass, then the vector is as well. Thus a vector of these elements can
be stored into Registers or FIFOs; for example a Register holding a vector of typeint. Note that a
vector can also store abstract types, such as a vector ofRulesor a vector ofReginterfaces. These
are useful during static elaboration although they have no hardware implementation.

Typeclasses


Reference Guide Bluespec SystemVerilog

```
Type Classes forVector
Bits Eq Literal Arith Ord Bounded Bitwise Bit Bit FShow
Reduction Extend
Vector
```
##### √ √ √ √

Bits A vector can be turned into bits if the individual elements can be turned into bits. When
packed and unpacked, the zeroth element of the vector is stored in the least significant bits. The
size of the resulting bits is given bytsize=vsize∗SizeOf#(elementtype) which is specified in the
provisos.

instance Bits #( Vector#(vsize, element_type), tsize)
provisos (Bits#(element_type, sizea),
Mul#(vsize, sizea, tsize));

Vectors are zero-indexed; the first element of a vectorv, isv[0]. When vectors are packed, they are
packed in order from the LSB to the MSB.

Example. Vector#(5, Bit#(7)) v1;

From the type, you can see that this will back into a 35-bit vector (5 elements, each with 7 bits).

##### MSB

```
34 bit positions 0
v1[4] v1[3] v1[2] v1[1] v1[0]
```
##### LSB

Example. A vector with a structure:

typedef struct { Bool a, UInt#(5) b} Newstruct deriving (Bits);
Vector#(3, NewStruct) v2;

The structure,Newstructpacks into 6 bits. Thereforev2will pack into an 18-bit vector. And its
structure would look as follows:

##### MSB

##### 17 16 - 12 11 10 - 6 5 0

```
v2[2].a v2[2].b v2[1].a v2[1].b v2[0].a v2[0].b
v2[2] v2[1] v2[0]
```
##### LSB

Eq Vectors can be compared for equality if the elements can. That is, the operators==and!=are
defined.

Bounded Vectors are bounded if the elements are.

FShow TheFShowclass provides thefshowfunction which can be applied to aVectorand returns
an associatedFmtobject showing:

<V elem1 elem2 ...>

where theelemnare the elements of the vector withfshowapplied to each element value.


Bluespec SystemVerilog Reference Guide

#### C.3.1 Creating and Generating Vectors

The following functions are used to create new vectors, with and without defined elements. There
are no Bluespec SystemVerilog constructors available for this abstract type (and hence no pattern-
matching is available for this type) but the following functions may be used to construct values of
theVectortype.

```
newVector Generate a vector with undefined elements, typically used when vectors are de-
clared.
```
```
function Vector#(vsize, element_type) newVector();
```
```
genVector Generate a vector containing integers 0 through n-1, vector[0] will have value 0.
```
```
function Vector#(vsize, Integer) genVector();
```
```
replicate Generate a vector of elements by replicating the given argument (c).
```
```
function Vector#(vsize, element_type) replicate(element_type c);
```
```
genWith Generate a vector of elements by applying the given function to 0 through n-1.
The argument to the function is another function which has one argument of type
Integerand returns anelement_type.
```
```
function Vector#(vsize, element_type)
genWith(function element_type func(Integer x1));
```
```
cons Adds an element to a vector creating a vector one element larger. The new element
will be at the 0th position. This function can lead to large compile times, so it
can be an inefficient way to create and populate a vector. Instead, the designer
should build a vector, then set each element to a value.
```
```
function Vector#(vsize1, element_type)
cons (element_type elem, Vector#(vsize, element_type) vect)
provisos (Add#(1, vsize, vsize1));
```
```
nil Defines a vector of size zero.
```
```
function Vector#(0, element_type) nil;
```

Reference Guide Bluespec SystemVerilog

```
append Append two vectors containing elements of the same type, returning the com-
bined vector. The resulting vectorresultwill contain all the elements ofvecta
followed by all the elements ofvectb.result[0]=vecta[0],result[vsize-1]
=vectb[v1size-1].
```
```
function Vector#( vsize, element_type )
append( Vector#(v0size,element_type) vecta,
Vector#(v1size,element_type) vectb)
provisos (Add#(v0size, v1size, vsize)); //vsize = v0size + v1size
```
```
concat Append (concatenate) many vectors, that is a vector of vectors into one vector.
concat(xss)[0]will bexss[0][0], providedmandnare non-zero.
```
```
function Vector#(mvsize,element_type)
concat(Vector#(m,Vector#(n,element_type)) xss)
provisos (Mul#(m,n,mvsize));
```
Examples - Creating and Generating Vectors

Create a new vector,my_vector, of 5 elements of datatytpeInt#(32), with elements which are
undefined.

```
Vector #(5, Int#(32)) my_vector;
```
Create a new vector,my_vector, of 5 elements of datatytpeIntegerwith elements 0, 1, 2, 3 and 4.

```
Vector #(5, Integer) my_vector = genVector;
// my_vector is a 5 element vector {0,1,2,3,4}
```
Create a vector, myvector, of five 1’s.

```
Vector #(5,Int #(32)) my_vector = replicate (1);
// my_vector is a 5 element vector {1,1,1,1,1}
```
Create a vector,my_vector, by applying the given functionadd2to 0 through n-1.

```
function Integer add2 (Integer a);
Integer c = a + 2;
return(c);
endfunction
```
```
Vector #(5,Integer) my_vector = genWith(add2);
```
```
// a is the index of the vector, 0 to n-1
// my_vector = {2,3,4,5,6,}
```
Add an element tomy_vector, creating a bigger vectormy_vector1.

```
Vector#(3, Integer) my_vector = genVector();
// my_vector = {0, 1, 2}
```
```
let my_vector1 = cons(4, my_vector);
// my_vector1 = {4, 0, 1, 2}
```

Bluespec SystemVerilog Reference Guide

Append vectors,my_vectorandmy_vector1, resulting in a vectormy_vector2.

```
Vector#(3, Integer) my_vector = genVector();
// my_vector = {0, 1, 2}
```
```
Vector#(3, Integer) my_vector1 = genWith(add2);
// my_vector1 = {2, 3, 4}
```
```
let my_vector2 = append(my_vector, my_vector1);
// my_vector2 = {0, 1, 2, 2, 3, 4}
```
#### C.3.2 Extracting Elements and Sub-Vectors

These functions are used to select elements or vectors from existing vectors, while retaining the input
vector.

```
[i] The square-bracket notation is available to extract an element from a vector or
update an element within it. Extracts or updates the ith element, where the
first element is [0]. Index i must be of an acceptable index type (e.g. Integer,
Bit#(n),Int#(n)orUInt#(n)). The square-bracket notation for vectors can also
be used with register writes.
```
```
anyVector[i];
anyVector[i] = newValue;
```
```
select The select function is another form of the subscript notation ([i]), mainly provided
for backwards-compatibility. The select function is also useful as an argument to
higher-order functions. The subscript notation is generally recommended because
it will report a more useful position for any selection errors.
```
```
function element_type
select(Vector#(vsize,element_type) vect, idx_type index);
```
```
update Update an element in a vector returning a new vector with one element
changed/updated. This function does not change the given vector. This is an-
other form of the subscript notation (see above), mainly provided for backwards
compatibility. The update function may also be useful as an argument to a higher-
order function. The subscript notation is generally recommended because it will
report a more useful position for any update errors.
```
```
function Vector#(vsize, element_type)
update(Vector#(vsize, element_type) vectIn,
idx_type index,
element_type newElem);
```

Reference Guide Bluespec SystemVerilog

```
head Extract the zeroth (head) element of a vector. The vector must have at least one
element.
```
```
function element_type
head (Vector#(vsize, element_type) vect)
provisos(Add#(1,xxx,vxize)); // vsize >= 1
```
```
last Extract the highest (tail) element of a vector. The vector must have at least one
element.
```
```
function element_type
last (Vector#(vsize, element_type) vect)
provisos(Add#(1,xxx,vxize)); // vsize >= 1
```
```
tail Remove the head element of a vector leaving its tail in a smaller vector.
```
```
function Vector#(vsize,element_type)
tail (Vector#(vsize1, element_type) xs)
provisos (Add#(1, vsize, vsize1));
```
```
init Remove the last element of a vector leaving its initial part in a smaller vector.
```
```
function Vector#(vsize,element_type)
init (Vector#(vsize1, element_type) xs)
provisos (Add#(1, vsize, vsize1));
```
```
take Take a number of elements from a vector starting from index 0. The number of
elements to take is indicated by the type of the context where this is called, and
is not specified as an argument to the function.
```
```
function Vector#(vxize2,element_type)
take (Vector#(vsize,element_type) vect)
provisos (Add#(vsize2,xxx,vsize)); // vsize2 <= vsize.
```

Bluespec SystemVerilog Reference Guide

```
drop
takeTail
```
```
Drop a number of elements from the vector starting at the 0th position. The
elements in the result vector will be in the same order as the input vector.
```
```
function Vector#(vxize2,element_type)
drop (Vector#(vsize,element_type) vect)
provisos (Add#(vsize2,xxx,vsize)); // vsize2 <= vsize.
```
```
function Vector#(vxize2,element_type)
takeTail (Vector#(vsize,element_type) vect)
provisos (Add#(vsize2,xxx,vsize)); // vsize2 <= vsize.
```
```
takeAt Take a number of elements starting atstartPos.startPosmust be a compile-
time constant. If thestartPosplus the output vector size extend beyond the end
of the input vector, an error will be returned.
```
```
function Vector#(vsize2,element_type)
takeAt (Integer startPos, Vector#(vsize,element_type) vect)
provisos (Add#(vsize2,xxx,vsize)); // vsize2 <= vsize
```
Examples - Extracting Elements and Sub-Vectors

Extract the element from a vector,my_vector, at the position of index.

```
// my_vector is a vector of elements {6,7,8,9,10,11}
// index = 3
// select or [ ] will generate a MUX
```
```
newvalue = select (my_vector, index);
newvalue = myvalue[index];
// newvalue = 9
```
Update the element of a vector,my_vector, at the position of index.

```
// my_vector is a vector of elements {6,7,8,9,10,11}
// index = 3
```
```
my_vector = update (my_vector, index, 0);
my_vector[index] = 0;
// my_vector = {6,7,8,0,10,11}
```
Extract the zeroth element of the vectormy_vector.

```
// my_vector is a vector of elements {6,7,8,9,10,11}
```
```
newvalue = head(my_vector);
// newvalue = 6
```
Extract the last element of the vectormy_vector.

```
// my_vector is a vector of elements {6,7,8,9,10,11}
```
```
newvalue = last(my_vector);
// newvalue = 11
```

Reference Guide Bluespec SystemVerilog

Create a vector,my_vector2, of size 4 by removing the head (zeroth) element of the vectormy_vector1.

```
// my_vector1 is a vector with 5 elements {0,1,2,3,4}
```
```
Vector #(4, Int#(32)) my_vector2 = tail (my_vector1);
// my_vector2 is a vector of 4 elements {1,2,3,4}
```
Create a vector,my_vector2, of size 4 by removing the tail (last) element of the vectormy_vector1.

```
// my_vector1 is a vector with 5 elements {0,1,2,3,4}
```
```
Vector #(4, Int#(32)) my_vector2 = init (my_vector1);
// my_vector2 is a vector of 4 elements {0,1,2,3}
```
Create a 2 element vector,my_vector2, by taking the first two elements of the vectormy_vector1.

```
// my_vector1 is vector with 5 elements {0,1,2,3,4}
```
```
Vector #(2, Int#(4)) my_vector2 = take (my_vector1);
// my_vector2 is a 2 element vector {0,1}
```
Create a 3 element vector,my_vector2, by taking the last 3 elements of vector,my_vector1. using
takeTail

```
// my_vector1 is Vector with 5 elements {0,1,2,3,4}
```
```
Vector #(3,Int #(4)) my_vector2 = takeTail (my_vector1);
// my_vector2 is a 3 element vector {2,3,4}
```
Create a 3 element vector,my_vector2, by taking the 1st - 3rd elements of vector,my_vector1.
usingtakeAt
// my_vector1 is Vector with 5 elements {0,1,2,3,4}

```
Vector #(3,Int #(4)) my_vector2 = takeAt (1, my_vector1);
// my_vector2 is a 3 element vector {1,2,3}
```
#### C.3.3 Vector to Vector Functions

The following functions generate a new vector by changing the position of elements within the vector.

```
rotate Move the zeroth element to the highest and shift each element lower by one. For
example, the element at indexnmoves to indexn-1.
```
```
function Vector#(vsize,element_type)
rotate (Vector#(vsize,element_type) vect);
```
```
rotateR Move last element to the zeroth element and shift each element up by one. For
example, the element at indexnmoves to indexn+1.
```
```
function Vector#(vsize,element_type)
rotateR (Vector#(vsize,element_type) vect);
```

Bluespec SystemVerilog Reference Guide

```
rotateBy Shift each elementnplaces. The lastnelements are moved to the beginning, the
element at index 0 moves to indexn, index 1 to indexn+1, etc.
```
```
function Vector#(vsize, element_type)
rotateBy (Vector#(vsize,element_type) vect, UInt#(log(v)) n)
provisos (Log#(vsize, logv);
```
```
shiftInAt0 Shift a new element into the vector at index 0, bumping the index of all other
element up by one. The highest element is dropped.
```
```
function Vector#(vsize,element_type)
shiftInAt0 (Vector#(vsize,element_type) vect,
element_type newElement);
```
```
shiftInAtN Shift a new element into the vector at index n, bumping the index of all other
elements down by one. The 0th element is dropped.
```
```
function Vector#(vsize,element_type)
shiftInAtN (Vector#(vsize,element_type) vect,
element_type newElement);
```
```
shiftOutFrom0 Shifts outamountnumber of elements from the vector starting at index 0,
bumping the index of all remaining elements down byamount. The shifted
elements are replaced with the valuedefault. This function is similar to a
>>bit operation. amt_typemust be of an acceptable index type (Integer,
Bit#(n),Int#(n)orUInt#(n)).
```
```
function Vector#(vsize,element_type)
shiftOutFrom0 (element_type default,
Vector#(vsize,element_type) vect,
amt_type amount);
```
```
shiftOutFromN Shifts outamount number of elements from the vector starting at index
vsize-1bumping the index of remaining elements up byamount. The shifted
elements are replaced with the valuedefault. This function is similar to a
<<bit operation. amt_typemust be of an acceptable index type (Integer,
Bit#(n),Int#(n)orUInt#(n)).
```
```
function Vector#(vsize,element_type)
shiftOutFromN (element_type default,
Vector#(vsize,element_type) vect,
amt_type amount);
```

Reference Guide Bluespec SystemVerilog

```
reverse Reverse element order
```
```
function Vector#(vsize,element_type)
reverse(Vector#(vsize,element_type) vect);
```
```
transpose Matrix transposition of a vector of vectors.
```
```
function Vector#(m,Vector#(n,element_type))
transpose ( Vector#(n,Vector#(m,element_type)) matrix );
```
```
transposeLN Matrix transposition of a vector of Lists.
```
```
function Vector#(vsize, List#(element_type))
transposeLN( List#(Vector#(vsize, element_type)) lvs );
```
Examples - Vector to Vector Functions

Create a vector by moving the last element to the first, then shifting each element to the right.

```
// my_vector1 is a vector of elements with values {1,2,3,4,5}
```
```
my_vector2 = rotateR (my_vector1);
// my_vector2 is a vector of elements with values {5,1,2,3,4}
```
Create a vector which is the input vector rotated by 2 places.

```
// my_vector1 is a vector of elements {1,2,3,4,5}
```
```
my_vector2 = rotateBy {my_vector1, 2};
// my_vector2 = {4,5,1,2,3}
```
Create a vector which shifts out 3 elements starting from 0, replacing them with the value F

```
// my_vector1 is a vector of elements {5,4,3,2,1,0}
```
```
my_vector2 = shiftOutFrom0 (F, my_vector1, 3);
// my_vector2 is a vector of elements {F,F,F,5,4,3}
```
Create a vector which shifts out 3 elements starting from n-1, replacing them with the value F

```
// my_vector1 is a vector of elements {5,4,3,2,1,0}
```
```
my_vector2 = shiftOutFromN (F, my_vector1, 3);
// my_vector2 is a vector of elements {2,1,0,F,F,F}
```
Create a vector which is the reverse of the input vector.

```
// my_vector1 is a vector of elements {1,2,3,4,5}
```
```
my_vector2 = reverse (my_vector1);
// my_vector2 is a vector of elements {5,4,3,2,1}
```

Bluespec SystemVerilog Reference Guide

Use transpose to create a new vector.

```
// my_vector1 is a Vector#(3, Vector#(5, Int#(8)))
// the result, my_vector2, is a Vector #(5,Vector#(3,Int #(8)))
```
```
// my_vector1 has the values:
// {{0,1,2,3,4},{5,6,7,8,9},{10,11,12,13,14}}
```
```
my_vector2 = transpose(my_vector1);
// my_vector2 has the values:
// {{0,5,10},{1,6,11},{2,7,12},{3,8,13},{4,9,14}}
```
#### C.3.4 Tests on Vectors

The following functions are used to test vectors. The first set of functions are Boolean functions,
i.e. they returnTrueorFalsevalues.

```
elem Check if a value is an element of a vector.
```
```
function Bool elem (element_type x,
Vector#(vsize,element_type) vect )
provisos (Eq#(element_type));
```
```
any Test if a predicate holds for any element of a vector.
```
```
function Bool any(function Bool pred(element_type x1),
Vector#(vsize,element_type) vect );
```
```
all Test if a predicate holds for all elements of a vector.
```
```
function Bool all(function Bool pred(element_type x1),
Vector#(vsize,element_type) vect );
```
```
or Combine all elements in a vector of Booleans with a logical or. Returns True if
any elements in the Vector are True.
```
```
function Bool or (Vector#(vsize, Bool) vect );
```
```
and Combine all elements in a vector of Booleans with a logical and. Returns True if
all elements in the Vector are True.
```
```
function Bool and (Vector#(vsize, Bool) vect );
```
The following two functions return the number of elements in the vector which match a condition.


Reference Guide Bluespec SystemVerilog

```
countElem Returns the number of elements in the vector which are equal to a given value.
The return value is in the range of 0 to vsize.
```
```
function UInt#(logv1) countElem (element_type x,
Vector#(vsize, element_type) vect)
provisos (Eq#(element_type), Add#(vsize, 1, vsize1),
Log#(vsize1, logv1));
```
```
countIf Returns the number of elements in the vector which satisfy a given predicate
function. The return value is in the range of 0 to vsize.
```
```
function UInt#(logv1) countIf (function Bool pred(element_type x1)
Vector#(vsize, element_type) vect)
provisos (Add#(vsize, 1, vsize1), Log#(vsize1, logv1));
```
```
find Returns the first element that satisfies the predicate orNothingif there is none.
```
```
function Maybe#(element_type)
find (function Bool pred(element_type),
Vector#(vsize, element_type) vect);
```
The following two functions return the index of an element.

```
findElem Returns the index of the first element in the vector which equals a given value.
Returns anInvalidif not found orValidwith a value of 0 to vsize-1 if found.
```
```
function Maybe#(UInt#(logv)) findElem (element_type x,
Vector#(vsize, element_type) vect)
provisos (Eq#(element_type), Add#(xx1, 1, vsize),
Log#(vsize, logv));
```
```
findIndex Returns the index of the first element in the vector which satisfies a given predicate.
Returns anInvalidif not found orValidwith a value of 0 to vsize-1 if found.
```
```
function Maybe#(UInt#(logv)) findIndex
(function Bool pred(element_type x1)
Vector#(vsize, element_type) vect)
provisos (Add#(xx1,1,vsize), Log#(vsize, logv));
```
Examples -Tests on Vectors

Test that all elements of the vectormy_vector1are positive integers.

```
function Bool isPositive (Int #(32) a);
return (a > 0)
```

Bluespec SystemVerilog Reference Guide

```
endfunction
```
```
// function isPositive checks that "a" is a positive integer
// if my_vector1 has n elements, n instances of the predicate
// function isPositive will be generated.
```
```
if (all(isPositive, my_vector1))
$display ("Vector contains all negative values");
```
Test if any elements in the vector are positive integers.

```
// function isPositive checks that "a" is a positive integer
// if my_vector1 has n elements, n instances of the predicate
// function isPositive will be generated.
```
```
if (any(isPositive, my_vector1))
$display ("Vector contains some negative values");
```
Check if the integer 5 is inmy_vector.

```
// if my_vector contains n elements, elem will generate n copies
// of the eq test
if (elem(5,my_vector))
$display ("Vector contains the integer 5");
```
Count the number of elements which match the integer provided.

```
// my_vector1 is a vector of {1,2,1,4,3}
x = countElem ( 1, my_vector1);
// x = 2
y = countElem (4, my_vector1);
// y = 1
```
Find the index of an element which equals a predicate.

```
let f = findIndex ( beIsGreaterThan( 3 ) , my_vector );
if ( f matches tagged Valid .indx )
begin
printBE ( my_vector[indx] ) ;
$display ("Found data > 3 at index %d ", indx ) ;
else
begin
$display ( "Did not find data > 3" ) ;
end
```
#### C.3.5 Bit-Vector Functions

The following functions operate on bit-vectors.

```
rotateBitsBy Shift each bit to a higher index bynplaces. The lastnbits are moved to the
begininng and the bit at index(0)moves to index(n).
```
```
function Bit#(n) rotateBitsBy (Bit#(n) bvect, UInt#(logn) n)
provisos (Log#(n,logn), Add#(1,xxx,n));
```

### C.3.9 ZipWith Functions Bluespec SystemVerilog Reference Guide

```
countOnesAlt Returns the number of elements equal to 1 in a bit-vector. (This function
differs slightly from the Prelude version of countOnes and has fewer provisos.)
```
```
function UInt#(logn1) countOnesAlt (Bit#(n) bvect)
provisos (Add#(1,n,n1), Log#(n1,logn1));
```
#### C.3.6 Functions on Vectors of Registers

```
readVReg Returns the values from reading a vector of registers (interfaces).
```
```
function Vector#(n,a) readVReg ( Vector#(n, Reg#(a)) vrin) ;
```
```
writeVReg Returns an Action which is the write of all registers invrwith the data from
vdin.
```
```
function Action writeVReg ( Vector#(n, Reg#(a)) vr,
Vector#(n,a) vdin) ;
```
#### C.3.7 Combining Vectors with Zip

The family ofzipfunctions takes two or more vectors and combines them into one vector ofTuples.
Several variations are provided for different resultingTuples, as well as support for mis-matched
vector sizes.

```
zip Combine two vectors into a vector of Tuples.
```
```
function Vector#(vsize,Tuple2 #(a_type, b_type))
zip( Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb);
```
```
zip3 Combine three vectors into a vector of Tuple3.
```
```
function Vector#(vsize,Tuple3 #(a_type, b_type, c_type))
zip3( Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb,
Vector#(vsize, c_type) vectc);
```

Bluespec SystemVerilog Reference Guide

```
zip4 Combine four vectors into a vector of Tuple4.
```
```
function Vector#(vsize,Tuple4 #(a_type, b_type, c_type, d_type))
zip4( Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb,
Vector#(vsize, c_type) vectc,
Vector#(vsize, d_type) vectd);
```
```
zipAny Combine two vectors into one vector of pairs (2-tuples); result is as long as the
smaller vector.
```
```
function Vector#(vsize,Tuple2 #(a_type, b_type))
zipAny(Vector#(m,a_type) vect1,
Vector#(n,b_type) vect2);
provisos (Max#(m,vsize,m), Max#(n, vsize, n));
```
```
unzip Separate a vector of pairs (i.e. aTuple2#(a,b))into a pair of two vectors.
```
```
function Tuple2#(Vector#(vsize,a_type), Vector#(vsize, b_type))
unzip(Vector#(vsize,Tuple2 #(a_type, b_type)) vectab);
```
Examples - Combining Vectors with Zip

Combine two vectors into a vector of Tuples.
// my_vector1 is a vector of elements {0,1,2,3,4}
// my_vector2 is a vector of elements {5,6,7,8,9}

```
my_vector3 = zip(my_vector1, my_vector2);
// my_vector3 is a vector of Tuples {(0,5),(1,6),(2,7),(3,8),(4,9)}
```
Separate a vector of pairs into a Tuple of two vectors.
// my_vector3 is a vector of pairs {(0,5),(1,6),(2,7),(3,8),(4,9)}

```
Tuple2#(Vector #(5,Int #(5)),Vector #(5,Int #(5))) my_vector4 =
unzip(my_vector3);
// my_vector4 is ({0,1,2,3,4},{5,6,7,8,9})
```
#### C.3.8 Mapping Functions over Vectors

A function can be applied to all elements of a vector, using high-order functions such asmap. These
functions take as an argument a function, which is applied to the elements of the vector.

```
map Map a function over a vector, returning a new vector of results.
```
```
function Vector#(vsize,b_type)
map (function b_type func(a_type x),
Vector#(vsize, a_type) vect);
```

Reference Guide Bluespec SystemVerilog

Example - Mapping Functions over Vectors

Consider the following code example which applies theextendfunction to each element ofavector
into a new vector,resultvector.
Vector#(13,Bit#(5)) avector;
Vector#(13,Bit#(10)) resultvector;
...
resultvector = map( extend, avector ) ;

This is equivalent to saying:

```
for (Integer i=0; i<13; i=i+1)
resultvector[i] = extend(avector[i]);
```
Map a negate function over a Vector
// my_vector1 is a vector of 5 elements {0,1,2,3,4}
// negate is a function which makes each element negative

```
Vector #(5,Int #(32)) my_vector2 = map (negate, my_vector1);
```
```
// my_vector2 is a vector of 5 elements {0,-1,-2,-3,-4}
```
C.3.9 ZipWith Functions

The zipWith functions combine two or more vectors with a function and generate a new vector.
These functions combine features of map and zip functions.

```
zipWith Combine two vectors with a function.
```
```
function Vector#(vsize,c_type)
zipWith (function c_type func(a_type x, b_type y),
Vector#(vsize,a_type) vecta,
Vector#(vsize,b_type) vectb );
```
```
zipWithAny Combine two vectors with a function; result is as long as the smaller vector.
```
```
function Vector#(vsize,c_type)
zipWithAny (function c_type func(a_type x, b_type y),
Vector#(m,a_type) vecta,
Vector#(n,b_type) vectb )
provisos (Max#(n, vsize, n), Max#(m, vsize, m));
```
```
zipWith3 Combine three vectors with a function.
```
```
function Vector#(vsize,d_type)
zipWith3(function d_type func(a_type x, b_type y, c_type z),
Vector#(vsize,a_type) vecta,
Vector#(vsize,b_type) vectb,
Vector#(vsize,c_type) vectc );
```

Bluespec SystemVerilog Reference Guide

```
zipWithAny3 Combine three vectors with a function; result is as long as the smallest vector.
```
```
function Vector#(vsize,c_type)
zipWithAny3(function d_type func(a_type x, b_type y, c_type z),
Vector#(m,a_type) vecta,
Vector#(n,b_type) vectb,
Vector#(o,c_type) vectc )
provisos (Max#(n, vsize, n), Max#(m, vsize, m), Max#(o, vsize, o));
```
Examples - ZipWith

Create a vector by applying a function over the elements of 3 vectors.
// the function add3 adds 3 values
function Int#(n) add3 (Int #(n) a,Int #(n) b,Int #(n) c);
Int#(n) d = a + b +c ;
return d;
endfunction

```
// Create the vector my_vector4 by adding the ith element of each of
// 3 vectors (my_vector1, my_vector2, my_vector3) to generate the ith
// element of my_vector4.
```
```
// my_vector1 = {0,1,2,3,4}
// my_vector2 = {5,6,7,8,9}
// my_vector3 = {10,11,12,13,14}
```
```
Vector #(5,Int #(8)) my_vector4 = zipWith3(add3, my_vector1, my_vector2, my_vector3);
// creates 5 instances of the add3 function in hardware.
// my_vector4 = {15,18,21,24,27}
```
```
// This is equivalent to saying:
for (Integer i=0; i<5; i=i+1)
my_vector4[i] = my_vector1[i] + my_vector2[i] + my_vector3[i];
```
### C.3.10 Fold Functions

Thefoldfamily of functions reduces a vector to a single result by applying a function over all its
elements. That is, given a vector ofelement_type,V 0 ,V 1 ,V 2 ,...,Vn− 1 , a seed of typeb_type, and
a functionfunc, the reduction forfoldris given by

```
func(V 0 ,func(V 1 ,...,func(Vn− 2 ,func(Vn− 1 ,seed))));
```
Note thatfoldrstart processing from the highest index position to the lowest, whilefoldlstarts
from the lowest index (zero), i.e.foldlis:

```
func(...(func(func(seed,V 0 ),V 1 ),...)Vn− 1 )
```
```
foldr Reduce a vector by applying a function over all its elements. Start processing
from the highest index to the lowest.
```
```
function b_type foldr(function b_type func(a_type x, b_type y),
b_type seed, Vector#(vsize,a_type) vect);
```

Reference Guide Bluespec SystemVerilog

```
foldl Reduce a vector by applying a function over all its elements. Start processing
from the lowest index (zero).
```
```
function b_type foldl (function b_type func(b_type y, a_type x),
b_type seed, Vector#(vsize,a_type) vect);
```
The functionsfoldr1andfoldl1use the first element as the seed. This means they only work on
vectors of at least one element. Since the result type will be the same as the element type, there is
nob_typeas there is in thefoldrandfoldlfunctions.

```
foldr1 foldrfunction for a non-zero sized vector, using elementVn− 1 as a seed. Vector
must have at least 1 element. If there is only one element, it is returned.
```
```
function element_type foldr1(
function element_type func(element_type x, element_type y),
Vector#(vsize,element_type) vect)
provisos (Add#(1, xxx, vsize));
```
```
foldl1 foldlfunction for a non-zero sized vector, using elementV 0 as a seed. Vector must
have at least 1 element. If there is only one element, it is returned.
```
```
function element_type foldl1 (
function element_type func(element_type y, element_type x),
Vector#(vsize,element_type) vect)
provisos (Add#(1, xxx, vsize));
```
Thefoldfunction also operates over a non-empty vector, but processing is accomplished in a binary
tree-like structure. Hence the depth or delay through the resulting function will beO(log 2 (vsize)
rather thanO(vsize).

```
fold Reduce a vector by applying a function over all its elements, using a binary tree-
like structure. The function returns the same type as the arguments.
```
```
function element_type fold (
function element_type func(element_type y, element_type x),
Vector#(vsize,element_type) vect )
provisos (Add#(1, xxx, vsize));
```

Bluespec SystemVerilog Reference Guide

```
mapPairs Map a function over a vector consuming two elements at a time. Any straggling
element is processed by the second function.
```
```
function Vector#(vsize2,b_type)
mapPairs (
function b_type func1(a_type x, a_type y),
function b_type func2(a_type x),
Vector#(vsize,a_type) vect )
provisos (Div#(vsize, 2, vsize2));
```
```
joinActions Join a number of actions together.joinActionsis used for static elaboration
only, no hardware is generated.
```
```
function Action joinActions (Vector#(vsize,Action) vactions);
```
```
joinRules Join a number of rules together.joinRulesis used for static elaboration only,
no hardware is generated.
```
```
function Rules joinRules (Vector#(vsize,Rules) vrules);
```
Example - Folds

Use fold to find the sum of the elements in a vector.

```
// my_vector1 is a vector of five integers {1,2,3,4,5}
// \+ is a function which returns the sum of the elements
// make sure you leave a space after the \+ and before the ,
```
```
// This will build an adder tree, instantiating 4 adders, with a maximum
// depth or delay of 3. If foldr1 or foldl1 were used, it would
// still instantiate 4 adders, but the delay would be 4.
```
```
my_sum = fold (\+ , my_vector1));
// my_sum = 15
```
Use fold to find the element with the maximum value.

```
// my_vector1 is a vector of five integers {2,45,5,8,32}
```
```
my_max = fold (max, my_vector1);
// my_max = 45
```
Create a new vector usingmapPairs. The functionsumis applied to each pair of elements (the first
and second, the third and fourth, etc.). If there is an uneven number of elements, the functionpass
is applied to the remaining element.

```
// sum is defined as c = a+b
function Int#(4) sum (Int #(4) a,Int #(4) b);
Int#(4) c = a + b;
return(c);
```

Reference Guide Bluespec SystemVerilog

```
endfunction
```
```
// pass is defined as a
function Int#(4) pass (Int #(4) a);
return(a);
endfunction
```
```
// my_vector1 has the elements {0,1,2,3,4}
```
```
my_vector2 = mapPairs(sum,pass,my_vector1);
// my_vector2 has the elements {1,5,4}
// my_vector2[0] = 0 + 1
// my_vector2[1] = 2 + 3
// my_vector2[2] = 4
```
### C.3.11 Scan Functions

Thescanfamily of functions applies a function over a vector, creating a new vector result. The
scanfunction is similar tofold, but the intermediate results are saved and returned in a vector,
instead of returning just the last result. The result of ascanfunction is a vector. That is, given a
vector ofelement_type,V 0 ,V 1 ,...,Vn− 1 , an initial valueinitbof typeb_type, and a functionfunc,
application of thescanrfunctions creates a new vectorW, where

```
Wn = init;
Wn− 1 = func(Vn− 1 ,Wn);
Wn− 2 = func(Vn− 2 ,Wn− 1 );
...
W 1 = func(V 1 ,W 2 );
W 0 = func(V 0 ,W 1 );
```
```
scanr Apply a function over a vector, creating a new vector result. Processes elements
from the highest index position to the lowest, and fill the resulting vector in the
same way. The result vector is 1 element longer than the input vector.
```
```
function Vector#(vsize1,b_type)
scanr(function b_type func(a_type x1, b_type x2),
b_type initb,
Vector#(vsize,a_type) vect)
provisos (Add#(1, vsize, vsize1));
```
```
sscanr Apply a function over a vector, creating a new vector result. The elements are pro-
cessed from the highest index position to the lowest. TheWnelement is dropped
from the result. Input and output vectors are the same size.
```
```
function Vector#(vsize,b_type)
sscanr(function b_type func(a_type x1, b_type x2),
b_type initb,
Vector#(vsize,a_type) vect );
```

Bluespec SystemVerilog Reference Guide

Thescanlfunction creates the resulting vector in a similar way asscanrexcept that the processing
happens from the zeroth element up to thenthelement.

```
W 0 = init;
W 1 = func(W 0 ,V 0 );
W 2 = func(W 1 ,V 1 );
...
Wn− 1 = func(Wn− 2 ,Vn− 2 );
Wn = func(Wn− 1 ,Vn− 1 );
```
Thesscanlfunction drops the first result,init, shifting the result index by one.

```
scanl Apply a function over a vector, creating a new vector result. Processes elements
from the zeroth element up to thenthelement. The result vector is 1 element
longer than the input vector.
```
```
function Vector#(vsize1,a_type)
scanl(function a_type func(a_type x1, b_type x2),
a_type q,
Vector#(vsize, b_type) vect)
provisos (Add#(1, vsize, vsize1));
```
```
sscanl Apply a function over a vector, creating a new vector result. Processes elements
from the zeroth element up to thenthelement. The first result,init, is dropped,
shifting the result index up by one. Input and output vectors are the same size.
```
```
function Vector#(vsize,a_type)
sscanl(function a_type func(a_type x1, b_type x2),
a_type q,
Vector#(vsize, b_type) vect );
```
```
mapAccumL Map a function, but pass an accumulator from head to tail.
```
```
function Tuple2 #(a_type, Vector#(vsize,c_type))
mapAccumL (function Tuple2 #(a_type, c_type)
func(a_type x, b_type y), a_type x0,
Vector#(vsize,b_type) vect );
```
```
mapAccumR Map a function, but pass an accumulator from tail to head.
```
```
function Tuple2 #(a_type, Vector#(vsize,c_type))
mapAccumR(function Tuple2 #(a_type, c_type)
func(a_type x, b_type y), a_type x0,
Vector#(vsize,b_type) vect );
```

Reference Guide Bluespec SystemVerilog

Examples - Scan

Create a vector of factorials.

```
// \* is a function which returns the result of a multiplied by b
function Bit #(16) \* (Bit #(16) b, Bit #(8) a);
return (extend (a) * b);
endfunction
```
```
// Create a vector of factorials by multiplying each input list element
// by the previous product (the output list element), to generate
// the next product. The seed is a Bit#(16) with a value of 1.
// The elements are processed from the zeroth element up to the $n^{th}$ element.
```
```
// my_vector1 = {1,2,3,4,5,6,7}
Vector#(8,Bit #(16)) my_vector2 = scanl (\*, 16’d1, my_vector1);
// 7 multipliers are generated
```
```
// my_vector2 = {1,1,2,6,24,120,720,5040}
// foldr with the same arguments would return just 5040.
```
### C.3.12 Monadic Operations

Within Bluespec, there are some functions which can only be invoked in certain contexts. Two
common examples are:ActionValue, and module instantiation. ActionValues can only be invoked
within anActioncontext, such as a rule block or an Action method, and can be considered as two
parts - the action and the value. Module instantiation can similarly be considered, modules can only
be instantiated in the module context, while the two parts are the module instantiation (the action
performed) and the interface (the result returned). These situations are consideredmonadic.

When a monadic function is to be applied over a vector using map-like functions such asmap,
zipWith, orreplicate, the monadic versions of these functions must be used. Moreover, the context
requirements of the applied function must hold. The common application for these functions is in
the generation (or instantiation) of vectors of hardware components.

```
mapM Takes a monadic function and a vector, and applies the function to all vector
elements returning the vector of corresponding results.
```
```
function m#(Vector#(vsize, b_type))
mapM ( function m#(b_type) func(a_type x),
Vector#(vsize, a_type) vecta )
provisos (Monad#(m));
```
```
mapM_ Takes a monadic function and a vector, applies the function to all vector elements,
and throws away the resulting vector leaving the action in its context.
```
```
function m#(void) mapM_(function m#(b_type) func(a_type x),
Vector#(vsize, a_type) vect)
provisos (Monad#(m));
```

Bluespec SystemVerilog Reference Guide

```
zipWithM Take a monadic function (which takes two arguments) and two vectors; the func-
tion applied to the corresponding element from each vector would return an action
and result. Perform all those actions and return the vector of corresponding re-
sults.
```
```
function m#(Vector#(vsize, c_type))
zipWithM( function m#(c_type) func(a_type x, b_type y),
Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb )
provisos (Monad#(m));
```
```
zipWithM_ Take a monadic function (which takes two arguments) and two vectors; the func-
tion is applied to the corresponding element from each vector. This is the same as
zipWithMbut the resulting vector is thrown away leaving the action in its context.
```
```
function m#(void)
zipWithM_(function m#(c_type) func(a_type x, b_type y),
Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb )
provisos (Monad#(m));
```
```
zipWith3M Same aszipWithMbut combines three vectors with a function. The function is
applied to the corresponding element from each vector and returns an action and
the vector of corresponding results.
```
```
function m#(Vector#(vsize, c_type))
zipWith3M( function m#(d_type)
func(a_type x, b_type y, c_type z),
Vector#(vsize, a_type) vecta,
Vector#(vsize, b_type) vectb,
Vector#(vsize, c_type) vectc )
provisos (Monad#(m));
```
```
genWithM Generate a vector of elements by applying the given monadic function to 0 through
n-1.
```
```
function m#(Vector#(vsize, element_type))
genWithM(function m#(element_type) func(Integer x))
provisos (Monad#(m));
```

Reference Guide Bluespec SystemVerilog

```
replicateM Generate a vector of elements by using the given monadic value repeatedly.
```
```
function m#(Vector#(vsize, element_type))
replicateM( m#(element_type) c)
provisos (Monad#(m));
```
Examples - Creating a Vector of Registers

The following example shows some common uses of theVectortype. We first create a vector of
registers, and show how to populate this vector. We then continue with some examples of accessing
and updating the registers within the vector, as well as alternate ways to do the same.

```
// First define a variable to hold the register interfaces.
// Notice the variable is really a vector of Interfaces of type Reg,
// not a vector of modules.
Vector#(10,Reg#(DataT)) vectRegs ;
```
```
// Now we want to populate the vector, by filling it with Reg type
// interfaces, via the mkReg module.
// Notice that the replicateM function is used instead of the
// replicate function since mkReg function is creating a module.
vectRegs <- replicateM( mkReg( 0 ) ) ;
```
```
// ...
```
```
// A rule showing a read and write of one register within the
// vector.
// The readReg function is required since the selection of an
// element from vectRegs returns a Reg#(DType) interface, not the
// value of the register. The readReg functions converts from a
// Reg#(DataT) type to a DataT type.
rule zerothElement ( readReg( vectRegs[0] ) > 20 ) ;
// set 0 element to 0
// The parentheses are required in this context to give
// precedence to the selection over the write operation.
(vectRegs[0]) <= 0 ;
```
```
// Set the 1st element to 5
// An alternate syntax
vectRegs[1]._write( 5 ) ;
endrule
```
```
rule lastElement ( readReg( vectRegs[9] ) > 200 ) ;
// Set the 9th element to -10000
(vectRegs[9]) <= -10000 ;
endrule
```
```
// These rules defined above can execute simultaneously, since
// they touch independent registers
```
```
// Here is an example of dynamic selection, first we define a
// register to be used as the selector.
Reg#(UInt#(4)) selector <- mkReg(0) ;
```

Bluespec SystemVerilog Reference Guide

```
// Now define another Reg variable which is selected from the
// vectReg variable. Note that no register is created here, just
// an alias is defined.
Reg#(DataT) thisReg = select(vectRegs, selector ) ;
```
```
//The above statement is equivalent to:
//Reg#(DataT) thisReg = vectRegs[selector] ;
```
```
// If the selected register is greater than 20’h7_0000, then its
// value is reset to zero. Note that the vector update function is
// not required since we are changing the contents of a register
// not the vector vectReg.
rule reduceReg( thisReg > 20’h7_0000 ) ;
thisReg <= 0 ;
selector <= ( selector < 9 )? selector + 1 : 0 ;
endrule
```
```
// As an alternative, we can define N rules which each check the
// value of one register and update accordingly. This is done by
// generating each rule inside an elaboration-time for-loop.
```
```
Integer i; // a compile time variable
for ( i = 0 ; i < 10 ; i = i + 1 ) begin
rule checkValue( readReg( vectRegs[i] ) > 20’h7_0000 ) ;
(vectRegs[i]) <= 0 ;
endrule
end
```
### C.3.13 Converting to and from Vectors

There are functions which convert between Vectors and other types.

```
toList Convert a Vector to a List.
```
```
function List#(element_type)
toList (Vector#(vsize, element_type) vect);
```
```
toVector Convert a List to a Vector.
```
```
function Vector#(vsize, element_type)
toVector ( List#(element_type) lst);
```
```
arrayToVector Convert an array to a Vector.
```
```
function Vector#(vsize, element_type)
arrayToVector ( element_type[ ] arr);
```

```
Reference Guide Bluespec SystemVerilog
```
```
vectorToArray Convert a Vector to an array.
```
```
function element_type[ ]
vectorToArray (Vector#(vsize, element_type) vect);
```
```
toChunks Convert a value to a Vector of chunks, possibly padding the final chunk. The
input type and size as well as the chunk type and size are determined from
their types.
```
```
function Vector#(n_chunk, chunk_type) toChunks(type_x x)
provisos( Bits#(chunk_type, chunk_sz), Bits#(type_x, x_sz)
, Div#(x_sz, chunk_sz, n_chunk) );
```
```
Example - Converting to and from Vectors
```
```
Convert the vectormy_vectorto a list namedmy_list.
```
```
Vector#(5,Int#(13)) my_vector;
List#(Int#(13)) my_list = toList(my_vector);
```
### C.3.14 ListN

```
Package name
```
```
import ListN :: * ;
```
```
Description
```
```
ListN is an alternative implementation of Vector which is preferred for sequential list processing
functions, such as head, tail, map, fold, etc. All Vector functions are available, by substituting ListN
for Vector. See the Vector documentation (C.3) for details. If the implementation requires random
access to items in the list, the Vector construct is recommended. Using ListN where Vectors is
recommended (and visa-versa) can lead to very long static elaboration times.
```
```
TheListNpackage defines an abstract data type which is a ListN of a specific length. Functions
which create and operate on this type are also defined within this package. Because it is abstract,
there are no constructors available for this type (likeConsandNilfor theListtype).
```
```
struct ListN#(vsize,a_type)
···abstract···
```
Here, the type variable “a_type” represents the type of the contents of the listN while type variable
“vsize” represents the length of the ListN.

## C.4 Aggregation: Lists

```
Package
```
```
import List :: * ;
```

Bluespec SystemVerilog Reference Guide

Description

TheListpackage defines a data type and functions which create and operate on this data type. Lists
are similar to Vectors, but are used when the number of items on the list may vary at compile-time
or need not be strictly enforced by the type system. All elements of a list must be of the same type.
The list type is defined as a tagged union as follows.

typedef union tagged {
void Nil;
struct {
a head;
List #(a) tail;
} Cons;
} List #(type a);

A list is taggedNilif it has no elements, otherwise it is taggedCons.Consis a structure of a single
element and the rest of the list.

Lists are most often used during static elaboration (compile-time) to manipulate collections of ob-
jects. SinceList#(element_type)is not in theBitstypeclass, lists cannot be stored in registers
or other dynamic elements. However, one can have a list of registers or variables corresponding to
hardware functions.

Data classes

FShow TheFShowclass provides the functionfshowwhich can be applied to aListand returns
an associatedFmtobject showing:

<List elem1 elem2 ...>

where theelemnare the elements of the list withfshowapplied to each element value.

### C.4.1 Creating and Generating Lists

```
cons Adds an element to a list. The new element will be at the 0th position.
```
```
function List#(element_type)
cons (element_type x, List#(element_type) xs);
```
```
upto Create a list of Integers counting up over a range of numbers, from m to n. If m
>n, an empty list (Nil) will be returned.
```
```
List#(Integer) upto(Integer m, Integer n);
```
```
replicate Generate a list of n elements by replicating the given argument,elem.
```
```
function List#(element_type)
replicate(Integer n, element_type elem);
```

Reference Guide Bluespec SystemVerilog

```
append Append two lists, returning the combined list. The elements of both lists must be
the same datatype,element_type. The combined list will contain all the elements
ofxsfollowed in order by all the elements ofys.
```
```
function List#(element_type)
append(List#(element_type) xs, List#(element_type) ys);
```
```
concat Append (concatenate) many lists, that is a list of lists, into one list.
```
```
function List# (element_type)
concat (List#(List#(element_type)) xss);
```
Examples - Creating and Generating Lists

Create a new list,my_list, of elements of datatytpeInt#(32)which are undefined

```
List #(Int#(32)) my_list;
```
Create a list,my_list, of five 1’s

```
List #(Int #(32)) my_list = replicate (5,32’d1);
```
```
//my_list = {1,1,1,1,1}
```
Create a new list using theuptofunction

```
List #(Integer) my_list2 = upto (1, 5);
```
```
//my_list2 = {1,2,3,4,5}
```
### C.4.2 Extracting Elements and Sub-Lists

```
[i] The square-bracket notation is available to extract an element from a list or update
an element within it. Extracts or updates the ith element, where the first element
is [0]. Index i must be of an acceptable index type (e.g. Integer,Bit#(n),
Int#(n)orUInt#(n)). The square-bracket notation for lists can also be used
with register writes.
```
```
anyList[i];
anyList[i] = newValue;
```
```
select The select function is another form of the subscript notation ([i]), mainly provided
for backwards-compatibility. The select function is also useful as an argument to
higher-order functions. The subscript notation is generally recommended because
it will report a more useful position for any selection errors.
```
```
function element_type
select(List#(element_type) alist, idx_type index);
```

Bluespec SystemVerilog Reference Guide

```
update Update an element in a list returning a new list with one element
changed/updated. This function does not change the given list. This is another
form of the subscript notation (see above), mainly provided for backwards compat-
ibility. The update function may also be useful as an argument to a higher-order
function. The subscript notation is generally recommended because it will report
a more useful position for any update errors.
```
```
function List#(element_type)
update(List#(element_type) alist,
idx_type index,
element_type newElem);
```
```
oneHotSelect Select a list element with a Boolean list. The Boolean list should have exactly
one element that isTrue, otherwise the result is undefined. The returned
element is the one in the corresponding position to theTrueelement in the
Boolean list.
```
```
function element_type
oneHotSelect (List#(Bool) bool_list,
List#(element_type) alist);
```
```
head Extract the first element of a list. The input list must have at least 1 element, or
an error will be returned.
```
```
function element_type head (List#(element_type) listIn);
```
```
last Extract the last element of a list. The input list must have at least 1 element, or
an error will be returned.
```
```
function element_type last (List#(element_type) alist);
```
```
tail Remove the head element of a list leaving the remaining elements in a smaller list.
The input list must have at least 1 element, or an error will be returned.
```
```
function List#(element_type) tail (List#(element_type) alist);
```
```
init Remove the last element of a list the remaining elements in a smaller list. The
input list must have at least one element, or an error will be returned.
```
```
function List#(element_type) init (List#(element_type) alist);
```

Reference Guide Bluespec SystemVerilog

```
take Take a number of elements from a list starting from index 0. The number to take
is specified by the argumentn. If the argument is greater than the number of
elements on the list, the function stops taking at the end of the list and returns
the entire input list.
```
```
function List#(element_type)
take (Integer n, List#(element_type) alist);
```
```
drop Drop a number of elements from a list starting from index 0. The number to drop
is specified by the argumentn. If the argument is greater than the number of
elements on the list, the entire input list is dropped, returning an empty list.
```
```
function List#(element_type)
drop (Integer n, List#(element_type) alist);
```
```
filter Create a new list from a given list where the new list has only the elements which
satisfy the predicate function.
```
```
function List#(element_type)
filter (function Bool pred(element_type),
List#(element_type) alist);
```
```
find Return the first element that satisfies the predicate orNothingif there is none.
```
```
function Maybe#(element_type)
find (function Bool pred(element_type),
List#(element_type) alist);
```
```
lookup Returns the value in an association list orNothingif there is no matching value.
```
```
function Maybe#(b_type)
lookup (a_type key, List#(Tuple2#(a_type, b_type)) alist)
provisos(Eq#(a_type));
```
```
takeWhile Returns the first set of elements of a list which satisfy the predicate function.
```
```
function List#(element_type)
takeWhile (function Bool pred(element_type x),
List#(element_type) alist);
```

Bluespec SystemVerilog Reference Guide

```
takeWhileRev Returns the last set of elements on a list which satisfy the predicate function.
```
```
function List#(element_type)
takeWhileRev (function Bool pred(element_type x),
List#(element_type) alist);
```
```
dropWhile Removes the first set of elements on a list which satisfy the predicate function,
returning a list with the remaining elements.
```
```
function List#(element_type)
dropWhile (function Bool pred(element_type x),
List#(element_type) alist);
```
```
dropWhileRev Removes the last set of elements on a list which satisfy the predicate function,
returning a list with the remaining elements.
```
```
function List#(element_type)
dropWhileRev (function Bool pred(element_type x),
List#(element_type) alist);
```
Examples - Extracting Elements and Sub-Lists

Extract the element from a list,my_list, at the position ofindex.

```
//my_list = {1,2,3,4,5}, index = 3
```
```
newvalue = select (my_list, index);
```
```
//newvalue = 4
```
Extract the zeroth element of the listmy_list.

```
//my_list = {1,2,3,4,5}
```
```
newvalue = head(my_list);
```
```
//newvalue = 1
```
Create a list,my_list2, of size 4 by removing the head (zeroth) element of the listmy_list1.

```
//my_list1 is a list with 5 elements, {0,1,2,3,4}
```
```
List #(Int #(32)) my_list2 = tail (my_list1);
List #(Int #(32)) my_list3 = tail(tail(tail(tail(tail(my_list1);
```
```
//my_list2 = {1,2,3,4}
//my_list3 = Nil
```
Create a 2 element list,my_list2, by taking the first two elements of the listmy_list1.


Reference Guide Bluespec SystemVerilog

```
//my_list1 is list with 5 elements, {0,1,2,3,4}
List #(Int #(4)) my_list2 = take (2,my_list1);
```
```
//my_list2 = {0,1}
```
The number of elements specified to take intakecan be greater than the number of elements on
the list, in which case the entire input list will be returned.

```
//my_list1 is list with 5 elements, {0,1,2,3,4}
List #(Int #(4)) my_list2 = take (7,my_list1);
```
```
//my_list2 = {0,1,2,3,4}
```
Select an element based on a boolean list.
//my_list1 is a list of unsigned integers, {1,2,3,4,5}
//my_list2 is a list of Booleans, only one value in my_list2 can be True.
//my_list2 = {False, False, True, False,False, False, False}.

```
result = oneHotSelect (my_list2, my_list1));
```
```
//result = 3
```
Create a list by removing the initial segment of a list that meets a predicate.

```
//the predicate function is a < 2
```
```
function Bool lessthan2 (Int #(4) a);
return (a < 2);
endfunction
```
```
//my_list1 = {0,1,2,0,1,7,8}
```
```
List #(Int #(4)) my_result = (dropWhile(lessthan2, my_list1));
```
```
//my_result = {2,0,1,7,8}
```
### C.4.3 List to List Functions

```
rotate Move the first element to the last and shift each element to the next higher index.
```
```
function List#(element_type) rotate (List#(element_type) alist);
```
```
rotateR Move last element to the beginning and shift each element to the next lower index.
```
```
function List#(element_type) rotateR (List#(element_type) alist);
```
```
reverse Reverse element order
```
```
function List#(element_type) reverse(List#(element_type) alist);
```

Bluespec SystemVerilog Reference Guide

```
transpose Matrix transposition of a list of lists.
```
```
function List#(List#(element_type))
transpose ( List#(List#(element_type)) matrix );
```
```
sort Uses the ordering defined for theelement_typedata type to return a list in ascending
order. The typeelement_typemust be in theOrdtype class.
```
```
function List#(element_type) sort(List#(element_type) alist)
provisos(Ord#(element_type)) ;
```
```
sortBy Generalizes thesortfunction to use an arbitrary ordering function defined by the
comparison functioncomparefin place of theOrdinstance forelement_type.
```
```
function List#(element_type)
sortBy(function Ordering comparef(element_type x, element_type y),
List#(element_type) alist);
```
```
group Returns a list of the contiguous subsequences of equal elements (according to theEq
instance forelement_type) found in its input list. Every element in the input list
will appear in exactly one sublist of the result. Every sublist will be a non-empty list
of equal elements. For any list,x,concat(group(x)) == x.
```
```
function List#(List#(element_type)) group (List#(element_type) alist)
provisos(Eq#(element_type)) ;
```
```
groupBy Generalizes thegroupfunction to use an arbitrary equivalence relation defined by
the comparison functioneqfin place of the Eq instance forelement_type.
```
```
function List#(List#(element_type))
groupBy(function Bool eqf(element_type x, element_type y),
List#(element_type) alist);
```
Examples - List to List Functions

Create a list by moving the last element to the first, then shifting each element to the right.

```
//my_list1 is a List of elements with values {1,2,3,4,5}
```
```
my_list2 = rotateR (my_list1);
```
```
//my_list2 is a List of elements with values {5,1,2,3,4}
```
Create a list which is the reverse of the input List


Reference Guide Bluespec SystemVerilog

```
//my_list1 is a List of elements {1,2,3,4,5}
```
```
my_list2 = reverse (my_list1);
```
```
//my_list2 is a List of elements {5,4,3,2,1}
```
Use transpose to create a new list
//my_list1 has the values:
//{{0,1,2,3,4},{5,6,7,8,9},{10,11,12,13,14}}

```
my_list2 = transpose(my_list1);
```
```
//my_list2 has the values:
//{{0,5,10},{1,6,11},{2,7,12},{3,8,13},{4,9,14}}
```
Use sort to create a new list
//my_list1 has the values: {3,2,5,4,1}

```
my_list2 = sort(my_list1);
```
```
//my_list2 has the values: {1,2,3,4,5}
```
Use group to create a list of lists
//my_list1 is a list of elements {Mississippi}

```
my_list2 = group(my_list1);
```
```
//my_list2 is a list of lists:
{{M},{i},{ss},{i},{ss},{i},{pp},{i}}
```
### C.4.4 Tests on Lists

##### ==

##### !=

```
Lists can be compared for equality if the elements in the list can be compared.
```
```
instance Eq #( List#(element_type) )
provisos( Eq#( element_type ) ) ;
```
```
elem Check if a value is an element in a list.
```
```
function Bool elem (element_type x, List#(element_type) alist )
proviso (Eq#(element_type));
```
```
length Determine the length of a list. Can be done at elaboration time only.
```
```
function Integer length (List#(element_type) alist );
```

Bluespec SystemVerilog Reference Guide

```
any Test if a predicate holds for any element of a list.
```
```
function Bool any(function Bool pred(element_type x1),
List#(element_type) alist );
```
```
all Test if a predicate holds for all elements of a list.
```
```
function Bool all(function Bool pred(element_type x1),
List#(element_type) alist );
```
```
or Combine all elements in a Boolean list with a logical or. Returns True if any
elements in the list are True.
```
```
function Bool or (List# (Bool) bool_list);
```
```
and Combine all elements in a Boolean list with a logical and. Returns True if all
elements in the list are true.
```
```
function Bool and (List# (Bool) bool_list);
```
Examples - Tests on Lists

Test that all elements of the listmy_list1are positive integers
function Bool isPositive (Int #(32) a);
return (a > 0)
endfunction

```
// function isPositive checks that "a" is a positive integer
// if my_list1 has n elements, n instances of the predicate
// function isPositive will be generated.
```
```
if (all(isPositive, my_list1))
$display ("List contains all negative values");
```
Test if any elements in the list are positive integers.
// function isPositive checks that "a" is a positive integer
// if my_list1 has n elements, n instances of the predicate
// function isPositive will be generated.

```
if (any(pos, my_list1))
$display ("List contains some negative values");
```
Check if the integer 5 is inmy_list
// if my_list contains n elements, elem will generate n copies
// of the eqt Test
if (elem(5,my_list))
$display ("List contains the integer 5");


Reference Guide Bluespec SystemVerilog

### C.4.5 Combining Lists with Zip Functions

The family ofzipfunctions takes two or more lists and combines them into one list ofTuples.
Several variations are provided for different resultingTuples. All variants can handle input lists of
different sizes. The resulting lists will be the size of the smallest list.

```
zip Combine two lists into a list of Tuples.
```
```
function List#(Tuple2 #(a_type, b_type))
zip( List#(a_type) lista,
List#(b_type) listb);
```
```
zip3 Combine 3 lists into a list of Tuple3.
```
```
function List#(Tuple3 #(a_type, b_type, c_type))
zip3( List#(a_type) lista,
List#(b_type) listb,
List#(c_type) listc);
```
```
zip4 Combine 4 lists into a list of Tuple4.
```
```
function List#(Tuple4 #(a_type, b_type, c_type, d_type))
zip4( List#(a_type) lista,
List#(b_type) listb,
List#(c_type) listc,
List#(d_type) listd);
```
```
unzip Separate a list of pairs (i.e. aTuple2#(a,b))into a pair of two lists.
```
```
function Tuple2#(List#(a_type), List#(b_type))
unzip(List#(Tuple2 #(a_type, b_type)) listab);
```
Examples - Combining Lists with Zip

Combine two lists into a list of Tuples
//my_list1 is a list of elements {0,1,2,3,4,5,6,7}
//my_list2 is a list of elements {True,False,True,True,False}

```
my_list3 = zip(my_list1, my_list2);
```
```
//my_list3 is a list of Tuples {(0,True),(1,False),(2,True),(3,True),(4,False)}
```
Separate a list of pairs into a Tuple of two lists
//my_list is a list of pairs {(0,5),(1,6),(2,7),(3,8),(4,9)}

```
Tuple2#(List#(Int#(5)),List#(Int#(5))) my_list2 = unzip(my_list);
```
```
//my_list2 is ({0,1,2,3,4},{5,6,7,8,9})
```

Bluespec SystemVerilog Reference Guide

### C.4.6 Mapping Functions over Lists

A function can be applied to all elements of a list, using high-order functions such asmap. These
functions take as an argument a function, which is applied to the elements of the list.

```
map Map a function over a list, returning a new list of results.
```
```
function List#(b_type) map (function b_type func(a_type),
List#(a_type) alist);
```
Example - Mapping Functions over Lists

Consider the following code example which applies theextendfunction to each element ofalist
creating a new list,resultlist.

```
List#(Bit#(5)) alist;
List#(Bit#(10)) resultlist;
```
```
resultlist = map( extend, alist ) ;
```
This is equivalent to saying:

```
for (Integer i=0; i<13; i=i+1)
resultlist[i] = extend(alist[i]);
```
Map a negate function over a list

```
//my_list1 is a list of 5 elements {0,1,2,3,4}
//negate is a function which makes each element negative
```
```
List #(Int #(32)) my_list2 = map (negate, my_list1);
```
```
//my_list2 is a list of 5 elements {0,-1,-2,-3,-4}
```
### C.4.7 ZipWith Functions

The zipWith functions combine two or more lists with a function and generate a new list. These
functions combine features of map and zip functions.

```
zipWith Combine two lists with a function. The lists do not have to have the same number
of elements.
```
```
function List#(c_type)
zipWith (function c_type func(a_type x, b_type y),
List#(a_type) listx,
List#(b_type) listy );
```

Reference Guide Bluespec SystemVerilog

```
zipWith3 Combine three lists with a function. The lists do not have to have the same
number of elements.
```
```
function List#(d_type)
zipWith3(function d_type func(a_type x, b_type y, c_type z),
List#(a_type) listx,
List#(b_type) listy,
List#(c_type) listz );
```
```
zipWith4 Combine four lists with a function. The lists do not have to have the same number
of elements.
```
```
function List#(e_type) zipWith4
(function e_type func(a_type x, b_type y, c_type z, d_type w),
List#(a_type) listx,
List#(b_type) listy,
List#(c_type) listz
List#(d_type) listw );
```
Examples - ZipWith

Create a list by applying a function over the elements of 3 lists.
//the function add3 adds 3 values
function Int#(8) add3 (Int #(8) a,Int #(8) b,Int #(8) c);
Int#(8) d = a + b +c ;
return(d);
endfunction

```
//Create the list my_list4 by adding the ith element of each of
//3 lists (my_list1, my_list2, my_list3) to generate the ith
//element of my_list4.
```
```
//my_list1 = {0,1,2,3,4}
//my_list2 = {5,6,7,8,9}
//my_list3 = {10,11,12,13,14}
```
```
List #(Int #(8)) my_list4 = zipWith3(add3, my_list1, my_list2, my_list3);
```
```
//my_list4 = {15,18,21,24,27}
```
```
// This is equivalent to saying:
for (Integer i=0; i<5; i=i+1)
my_list4[i] = my_list1[i] + my_list2[i] + my_list3[i];
```
### C.4.8 Fold Functions

Thefoldfamily of functions reduces a list to a single result by applying a function over all its
elements. That is, given a list ofelement_type,L 0 ,L 1 ,L 2 ,...,Ln− 1 , a seed of typeb_type, and a
functionfunc, the reduction forfoldris given by

```
func(L 0 ,func(L 1 ,...,func(Ln− 2 ,func(Ln− 1 ,seed))));
```

Bluespec SystemVerilog Reference Guide

Note thatfoldrstart processing from the highest index position to the lowest, whilefoldl starts
from the lowest index (zero), i.e.,

```
func(...(func(func(seed,L 0 ),L 1 ),...)Ln− 1 )
```
```
foldr Reduce a list by applying a function over all its elements. Start processing from
the highest index to the lowest.
```
```
function b_type foldr(b_type function func(a_type x, b_type y),
b_type seed,
List#(a_type) alist);
```
```
foldl Reduce a list by applying a function over all its elements. Start processing from
the lowest index (zero).
```
```
function b_type foldl (b_type function func(b_type y, a_type x),
b_type seed,
List#(a_type) alist);
```
The functionsfoldr1andfoldl1use the first element as the seed. This means they only work on
lists of at least one element. Since the result type will be the same as the element type, there is no
b_typeas there is in thefoldrandfoldlfunctions.

```
foldr1 foldrfunction for a non-zero sized list. Uses elementLn− 1 as the seed. List must
have at least 1 element.
```
```
function element_type foldr1
(element_type function func(element_type x, element_type y),
List#(element_type) alist);
```
```
foldl1 foldlfunction for a non-zero sized list. Uses elementL 0 as the seed. List must
have at least 1 element.
```
```
function element_type foldl1
(element_type function func(element_type y, element_type x),
List#(element_type) alist);
```
Thefoldfunction also operates over a non-empty list, but processing is accomplished in a binary
tree-like structure. Hence the depth or delay through the resulting function will beO(log 2 (lsize)
rather thanO(lsize).


Reference Guide Bluespec SystemVerilog

```
fold Reduce a list by applying a function over all its elements, using a binary tree-like
structure. The function returns the same type as the arguments.
```
```
function element_type fold
(element_type function func(element_type y, element_type x),
List#(element_type) alist );
```
```
joinActions Join a number of actions together.
```
```
function Action joinActions (List#(Action) list_actions);
```
```
joinRules Join a number of rules together.
```
```
function Rules joinRules (List#(Rules) list_rules);
```
```
mapPairs Map a function over a list consuming two elements at a time. Any straggling
element is processed by the second function.
```
```
function List#(b_type)
mapPairs (
function b_type func1(a_type x, a_type y),
function b_type func2(a_type x),
List#(a_type) alist );
```
Example - Folds

```
// my_list1 is a list of five integers {1,2,3,4,5}
// \+ is a function which returns the sum of the elements
```
```
my_sum = foldr (\+ , 0, my_list1));
```
```
// my_sum = 15
```
Use fold to find the element with the maximum value

```
// my_list1 is a list of five integers {2,45,5,8,32}
```
```
my_max = fold (max, my_list1);
```
```
// my_max = 45
```
Create a new list usingmapPairs. The functionsumis applied to each pair of elements (the first
and second, the third and fourth, etc.). If there is an uneven number of elements, the functionpass
is applied to the remaining element.

```
//sum is defined as c = a+b
function Int#(4) sum (Int #(4) a,Int #(4) b);
```

Bluespec SystemVerilog Reference Guide

```
Int#(4) c = a + b;
return(c);
endfunction
```
```
//pass is defined as a
function Int#(4) pass (Int #(4) a);
return(a);
endfunction
```
```
//my_list1 has the elements {0,1,2,3,4}
```
```
my_list2 = mapPairs(sum,pass,my_list1);
```
```
//my_list2 has the elements {1,5,4}
//my_list2[0] = 0 + 1
//my_list2[1] = 2 + 3
//my_list2[3] = 4
```
### C.4.9 Scan Functions

Thescanfamily of functions applies a function over a list, creating a new List result. Thescan
function is similar tofold, but the intermediate results are saved and returned in a list, instead
of returning just the last result. The result of ascanfunction is a list. That is, given a list
ofelement_type,L 0 ,L 1 ,...,Ln− 1 , an initial valueinitbof typeb_type, and a functionfunc,
application of thescanrfunctions creates a new listW, where

```
Wn = init;
Wn− 1 = func(Ln− 1 ,Wn);
Wn− 2 = func(Ln− 2 ,Wn− 1 );
...
W 1 = func(L 1 ,W 2 );
W 0 = func(L 0 ,W 1 );
```
```
scanr Apply a function over a list, creating a new list result. Processes elements from
the highest index position to the lowest, and fills the resulting list in the same
way. The result list is one element longer than the input list.
```
```
function List#(b_type)
scanr(function b_type func(a_type x1, b_type x2),
b_type initb,
List#(a_type) alist);
```

Reference Guide Bluespec SystemVerilog

```
sscanr Apply a function over a list, creating a new list result. The elements are processed
from the highest index position to the lowest. Drops theWnelement from the
result. Input and output lists are the same size.
```
```
function List#(b_type)
sscanr(function b_type func(a_type x1, b_type x2),
b_type initb,
List#(a_type) alist );
```
Thescanlfunction creates the resulting list in a similar way asscanrexcept that the processing
happens from the zeroth element up to the nth element.

```
W 0 = init;
W 1 = func(W 0 ,L 0 );
W 2 = func(W 1 ,L 1 );
...
Wn− 1 = func(Wn− 2 ,Ln− 2 );
Wn = func(Wn− 1 ,Ln− 1 );
```
Thesscanlfunction drops the first result,init, shifting the result index by one.

```
scanl Apply a function over a list, creating a new list result. Processes elements from
the zeroth element up to the nth element. The result list is 1 element longer than
the input list.
```
```
function List#(a_type)
scanl(function a_type func(a_type x1, b_type x2),
a_type inita,
List#(b_type) alist);
```
```
sscanl Apply a function over a list, creating a new list result. Processes elements from
the zeroth element up to the nth element. Drop the first result,init, shifting the
result index by one. The length of the input and output lists are the same.
```
```
function List#(a_type)
sscanl(function a_type func(a_type x1, b_type x2),
a_type inita,
List#(b) alist );
```
```
mapAccumL Map a function, but pass an accumulator from head to tail.
```
```
function Tuple2 #(a_type, List#(c_type))
mapAccumL (function Tuple2 #(a_type, c_type)
func(a_type x, b_type y),a_type x0,
List#(b_type) alist );
```

Bluespec SystemVerilog Reference Guide

```
mapAccumR Map a function, but pass an accumulator from tail to head.
```
```
function Tuple2 #(a_type, List#(c_type))
mapAccumR(function Tuple2 #(a_type, c_type)
func(a_type x, b_type y),a_type x0,
List#(b_type) alist );
```
Examples - Scan

Create a list of factorials

```
//the function my_mult multiplies element a by element b
function Bit #(16) my_mult (Bit #(16) b, Bit #(8) a);
return (extend (a) * b);
endfunction
```
```
// Create a list of factorials by multiplying each input list element
// by the previous product (the output list element), to generate
// the next product. The seed is a Bit#(16) with a value of 1.
// The elements are processed from the zeroth element up to the nth element.
//my_list1 = {1,2,3,4,5,6,7}
```
```
List #(Bit #(16)) my_list2 = scanl (my_mult, 16’d1, my_list1);
```
```
//my_list2 = {1,1,2,6,24,120,720,5040}
```
### C.4.10 Monadic Operations

Within Bluespec, there are some functions which can only be invoked in certain contexts. Two
common examples are:ActionValue, and module instantiation. ActionValues can only be invoked
within anActioncontext, such as a rule block or an Action method, and can be considered as two
parts - the action and the value. Module instantiation can similarly be considered, modules can only
be instantiated in the module context, while the two parts are the module instantiation (the action
performed) and the interface (the result returned). These situations are consideredmonadic.

When a monadic function is to be applied over a list using map-like functions such asmap, zipWith,
orreplicate, the monadic versions of these functions must be used. Moreover, the context require-
ments of the applied function must hold.

```
mapM Takes a monadic function and a list, and applies the function to all list elements
returning the list of corresponding results.
```
```
function m#(List#(b_type))
mapM ( function m#(b_type) func(a_type x),
List#(a_type) alist )
provisos (Monad#(m));
```

Reference Guide Bluespec SystemVerilog

```
mapM_ Takes a monadic function and a list, applies the function to all list elements, and
throws away the resulting list leaving the action in its context.
```
```
function m#(List#(b_type) mapM_(m#(b_type) c_type)
provisos (Monad#(m));
```
```
zipWithM Take a monadic function (which takes two arguments) and two lists; the function
applied to the corresponding element from each list would return an action and
result. Perform all those actions and return the list of corresponding results.
```
```
function m#(List#(c_type))
zipWithM( function m#(c_type) func(a_type x, b_type y),
List#(a_type) alist,
List#(b_type) blist )
provisos (Monad#(m));
```
```
zipWith3M Same aszipWithMbut combines three lists with a function. The function is
applied to the corresponding element from each list and returns an action and the
list of corresponding results.
```
```
function m#(List#(d_type))
zipWith3M( function m#(d_type)
func(a_type x, b_type y, c_type z),
List#(a_type) alist ,
List#(b_type) blist,
List#(c_type) clist )
provisos (Monad#(m));
```
```
replicateM Generate a list of elements by using the given monadic value repeatedly.
```
```
function m#(List#(element_type))
replicateM( Integer n, m#(element_type) c)
provisos (Monad#(m));
```
## C.5 Math

### C.5.1 Real

Package

import Real :: * ;

Description


Bluespec SystemVerilog Reference Guide

TheReallibrary package defines functions to operate on and manipulate real numbers. Real numbers
are numbers with a fractional component. They are also of limited precision. TheRealdata type
is described in section B.2.6.

Constants

The constantpi(π) is defined.

```
pi The value of the constant pi (π).
```
```
Real pi;
```
Trigonometric Functions

The following trigonometric functions are provided: sin,cos,tan,sinh,cosh,tanh,asin,acos,
atan,asinh,acosh,atanh, andatan2.

```
sin Returns the sine ofx.
```
```
function Real sin (Real x);
```
```
cos Returns the cosine ofx.
```
```
function Real cos (Real x);
```
```
tan Returns the tangent ofx.
```
```
function Real tan (Real x);
```
```
sinh Returns the hyperbolic sine ofx.
```
```
function Real sinh (Real x);
```
```
cosh Returns the hyperbolic cosine ofx.
```
```
function Real cosh (Real x);
```

Reference Guide Bluespec SystemVerilog

```
tanh Returns the hyperbolic tangent ofx.
```
```
function Real tanh (Real x);
```
```
asinh Returns the inverse hyperbolic sine ofx.
```
```
function Real asinh (Real x);
```
```
acosh Returns the inverse hyperbolic cosine ofx.
```
```
function Real acosh (Real x);
```
```
atanh Returns the inverse hyperbolic tangent ofx.
```
```
function Real atanh (Real x);
```
```
atan2 Returnsatan(x/y).atan2(1,x)is equivalent toatan(x), but provides more
precision when required by the division ofx/y.
```
```
function Real atan2 (Real y, Real x);
```
Arithmetic Functions

```
pow The element x is raised to the y power. An alias for**.pow(x,y)=x**y=
xy.
```
```
function Real pow (Real x, Real y);
```
```
sqrt Returns the square root ofx. Returns an error ifxis negative.
```
```
function Real sqrt (Real x);
```

Bluespec SystemVerilog Reference Guide

Conversion Functions

The following four functions are used to convert aRealto anInteger.

```
trunc Converts aRealto anIntegerby removing the fractional part ofx, which
can be positive or negative.trunc(1.1) = 1,trunc(-1.1)= -1.
```
```
function Integer trunc (Real x);
```
```
round Converts aRealto anIntegerby rounding to the nearest whole number..5
rounds up in magnitude.round(1.5) = 2,round(-1.5)= -2.
```
```
function Integer round (Real x);
```
```
ceil Converts aRealto anIntegerby rounding to the higher number, regardless
of sign.ceil(1.1) = 2,ceil(-1.1) = -1.
```
```
function Integer ceil (Real x);
```
```
floor Converts aRealto anIntegerby rounding to the lower number, regardless
of sign.floor(1.1) = 1,floor(-1.1) = -2.
```
```
function Integer floor (Real x);
```
There are also two system functions$realtobitsand$bitstoreal, defined in the Prelude (section
B.2.6) which provide conversion to and from IEEE 64-bit vectors (Bit#(64)).

Introspection Functions

```
isInfinite ReturnsTrueif the value ofxis infinite,Falseifxis finite.
```
```
function Bool isInfinite (Real x);
```
```
isNegativeZero ReturnsTrueif the value ofxis negative zero.
```
```
function Bool isNegativeZero (Real x);
```

Reference Guide Bluespec SystemVerilog

```
splitReal Returns a Tuple containing the whole (n) and fractional (f) parts ofxsuch
thatn+f=x. Both values have the same sign asx. The absolute value of
the fractional part is guaranteed to be in the range [0,1).
```
```
function Tuple2#(Integer, Real) splitReal (Real x);
```
```
decodeReal Returns a Tuple3 containing the sign, the fraction, and the exponent of a real
number. The second part (the first Integer) represents the fractional part as
a signed Integer value. This can be converted to anInt#(54)(52 bits, plus
hidden bit, plus the sign bit). The last value is a signed Integer representing
the exponent, which can be be converted to anInt#(11). The real number is
represented exactly as (fractional× 2 exp). TheBoolrepresents the sign and
isTruefor positive and positive zero,Falsefor negative and negative zero.
Since the second value is a signed value, theBoolis redundant except for zero
values.
```
```
function Tuple3#(Bool, Integer, Integer) decodeReal (Real x);
```
```
realToDigits Deconstructs a real number into its digits. The function takes a base and
a real number and returns a list of digits and an exponent (ignoring the
sign). In particular, ifx≥0, andrealToDigits(base,x)returned a list
of digitsd1, d2, ..., dnand an exponente, then:
```
- n≥ 1
- abs(x) = 0.d1d2...dn * (basee)
- 0 ≤di≤base-1

```
function Tuple2#(List#(Integer), Integer)
realToDigits (Integer base, Real r);
```
### C.5.2 OInt

Package

import OInt :: * ;

Description

TheOInt#(n)type is an abstract type that can store a number in the range “0..n-1”. The repre-
sentation of aOInt#(n)takes upnbits, where exactly one bit is a set to one, and the others are
zero, i.e., it is aone-hotdecoded version of the number. The reason to use aOIntnumber is that
theselectoperation is more efficient than for a binary-encoded number; the code generated for
select takes advantage of the fact that only one of the bits may be set at a time.

Types and type classes

Definition ofOInt


Bluespec SystemVerilog Reference Guide

typedef ... OInt #(numeric type n) ... ;

```
Type Classes used byOInt
Bits Eq Literal Arith Ord Bounded Bit Bit Bit
wise Reduction Extend
OInt
```
##### √ √ √ √

Functions

A binary-encoded number can be converted to anOInt.

```
toOInt Converts from a bit-vector in unsigned binary format to anOInt.
An out-of-range number gives an unspecified result.
```
```
function OInt#(n) toOInt(Bit#(k) k)
provisos( Log#(n,k)) ;
```
AnOIntcan be converted to a binary-encoded number.

```
fromOInt Converts anOIntto a bit-vector in unsigned binary format.
```
```
function Bit#(k) fromOInt(OInt#(n) o)
provisos( Log#(n,k)) ;
```
AnOIntcan be used to select an element from a Vector in an efficient way.

```
select The Vectorselectfunction, where the type of the index is an
OInt.
```
```
function a_type select(Vector#(vsize, a_type) vecta,
OInt#(vsize) index)
provisos (Bits#(a_type, sizea));
```
### C.5.3 Complex

Package

import Complex :: * ;

Description

TheComplexpackage provides a representation for complex numbers plus functions to operate on
variables of this type. The basic representation is theComplexstructure, which is polymorphic
on the type of data it holds. For example, one can have complex numbers of typeIntor of type
FixedPoint. AComplexnumber is represented in two part, the real part (rel) and the imaginary
part (img). These fields are accessible though standard structure addressing, i.e.,foo.rel and
foo.imgwherefoois of typeComplex.

typedef struct {
any_t rel ;
any_t img ;
} Complex#(type any_t)
deriving ( Bits, Eq ) ;


Reference Guide Bluespec SystemVerilog

This package is provided as both a compiled library package and as BSV source code to facilitate
customization. The source code file can be found in the$BLUESPECDIR/BSVSource/Mathdirectory.
To customize a package, copy the file into a local directory and then include the local directory in
the path when compiling. This is done by specifying the search path with the-poption as described
in the BSV Users Guide.

Types and type classes

TheComplextype belongs to theArith,Literal,SaturatingArith, andFShowtype classes. Each
type class definition includes functions which are then also defined for the data type. The Prelude
library definitions (Section B) describes which functions are defined for each type class.

```
Type Classes used byComplex
Bits Eq Literal Arith Ord Bounded Bit Bit Bit FShow
wise Reduction Extend
Complex
```
##### √ √ √ √ √

Arith The typeComplexbelongs to theArithtype class, hence the common infix operators (+,
-, *, and /) are defined and can be used to manipulate variables of typeComplex. The remaining
arithmetic operators are not defined for theComplextype. Note however, that some functions
generate more hardware than may be expected. The complex multiplication (*) produces four
multipliers in a combinational function; some other modules could accomplish the same function with
less hardware but with greater latency. The complex division operator (/) produces 6 multipliers,
and a divider and may not always be synthesizable with downstream tools.

instance Arith#( Complex#(any_type) )
provisos( Arith#(any_type) ) ;

Literal TheComplextype is a member of theLiteralclass, which defines a conversion from the
compile-timeIntegertype toComplextype with thefromIntegerfunction. This function converts
the Integer to the real part, and sets the imaginary part to 0.

instance Literal#( Complex#(any_type) )
provisos( Literal#(any_type) );

SaturatingArith TheSaturatingArithclass provides the functionssatPlus,satMinus,boundedPlus,
andboundedMinus. These are modified plus and minus functions which saturate to values defined
by theSaturationModewhen the operations would otherwise overflow or wrap-around. The type
of the complex value (any_type)must be in theSaturatingArithclass.

instance SaturatingArith#(Complex#(any_type))
provisos (SaturatingArith#(any_type));

FShow An instance ofFShowis available providedany_typeis a member ofFShowas well.

instance FShow#(Complex#(any_type))
provisos (FShow#(any_type));
function Fmt fshow (Complex#(any_type) x);
return $format("<C ", fshow(x.rel), ",", fshow(x.img), ">");
endfunction
endinstance

Functions


Bluespec SystemVerilog Reference Guide

```
cmplx A simple constructor function is provided to set the fields.
```
```
function Complex#(a_type) cmplx( a_type realA, a_type imagA ) ;
```
```
cmplxMap Applies a function to each part of the complex structure. This is useful for
operations such asextend,truncate, etc.
```
```
function Complex#(b_type) cmplxMap(
function b_type mapFunc( a_type x),
Complex#(a_type) cin ) ;
```
```
cmplxSwap Exchanges the real and imaginary parts.
```
```
function Complex#(a_type) cmplxSwap( Complex#(a_type) cin ) ;
```
```
cmplxWrite Displays a complex number given a prefix string, an infix string, a postscript
string, and an Action function which writes each part. cmplxWriteis of type
Action and can only be invoked in Action contexts such as Rules and Actions
methods.
```
```
function Action cmplxWrite(String pre,
String infix,
String post,
function Action writeaFunc( a_type x ),
Complex#(a_type) cin );
```
Examples - Complex Numbers

```
// The following utility function is provided for writing data
// in decimal format. An example of its use is show below.
```
```
function Action writeInt( Int#(n) ain ) ;
$write( "%0d", ain ) ;
endfunction
```
```
// Set the fields of the complex number using the constructor function cmplx
Complex#(Int#(6)) complex_value = cmplx(-2,7) ;
```
```
// Display complex_value as ( -2 + 7i ).
// Note that writeInt is passed as an argument to the cmplxWrite function.
cmplxWrite( "( ", " + ", "i)", writeInt, complex_value );
```

Reference Guide Bluespec SystemVerilog

```
// Swap the real and imaginary parts.
swap_value = cmplxSwap( complex_value ) ;
```
```
// Display the swapped values. This will display ( -7 + 2i).
cmplxWrite( "( ", " + ", "i)", writeInt, swap_value );
```
### C.5.4 FixedPoint

Package

import FixedPoint :: * ;

Description

TheFixedPointlibrary package defines a type for representing fixed-point numbers and correspond-
ing functions to operate and manipulate variables of this type.

A fixed-point number represents signed numbers which have a fixed number of binary digits (bits)
before and after the binary point. The type constructor for a fixed-point number takes two numeric
types as argument; the first (isize) defines the number of bits to the left of the binary point (the
integer part), while the second (fsize) defines the number of bits to the right of the binary point,
(the fractional part).

The following data structure defines this type, while some utility functions provide the reading of
the integer and fractional parts.
typedef struct {
Bit#(isize) i;
Bit#(fsize) f;
}
FixedPoint#(numeric type isize, numeric type fsize )
deriving( Eq ) ;

This package is provided as both a compiled library package and as BSV source code to facilitate
customization. The source code file can be found in the$BLUESPECDIR/BSVSource/Mathdirectory.
To customize a package, copy the file into a local directory and then include the local directory in
the path when compiling. This is done by specifying the search path with the-poption as described
in the BSV Users Guide.

Types and type classes

TheFixedPointtype belongs to the following type classes;Bits,Eq,Literal,RealLiteral,Arith,
Ord,Bounded,Bitwise,SaturatingArith, andFShow. Each type class definition includes functions
which are then also defined for the data type. The Prelude library definitions (Section B) describes
which functions are defined for each type class.

```
Type Classes used byFixedPoint
Bits Eq Literal Real Arith Ord Bounded Bit Bit Bit Format
Literal wise Reduce Extend
FixedPoint
```
##### √ √ √ √ √ √ √ √ √

Bits The typeFixedPointbelongs to theBitstype class, which allows conversion from typeBits
to typeFixedPoint.

instance Bits#( FixedPoint#(isize, fsize), bsize )
provisos ( Add#(isize, fsize, bsize) );


Bluespec SystemVerilog Reference Guide

Literal The typeFixedPointbelongs to theLiteraltype class, which allows conversion from
(compile-time) typeIntegerto typeFixedPoint. Note that only the integer part is assigned.
instance Literal#( FixedPoint#(isize, fsize) )
provisos( Add#(isize, fsize, bsize) );

RealLiteral The typeFixedPointbelongs to theRealLiteraltype class, which allows conversion
from typeRealto typeFixedPoint.
instance RealLiteral#( FixedPoint# (isize, fsize) )

Example:
FixedPoint#(4,10) mypi = 3.1415926; //Implied fromReal
FixedPoint#(2,14) cx = fromReal(cos(pi/4));

Arith The typeFixedPointbelongs to theArithtype class, hence the common infix operators
(+, -, *, and/) are defined and can be used to manipulate variables of typeFixedPoint. The
arithmetic operator%is not defined.

instance Arith#( FixedPoint#(isize, fsize) )
provisos( Add#(isize, fsize, bsize) ) ;

For multiplication (*) and quotient (/) , the operation is calculated in full precision and the result
is then rounded and saturated to the resulting size. Both operators use the rounding function
fxptTruncateRoundSat, with modeRnd_Zero,Sat_Bound.

Ord In addition to equality and inequality comparisons,FixedPointvariables can be compared
by the relational operators provided by theOrdtype class. i.e.,<,>,<=, and>=.
instance Ord#( FixedPoint#(isize, fsize) )
provisos( Add#(isize, fsize, bsize) ) ;

Bounded The typeFixedPointbelongs to theBoundedtype class. The range of values,v, rep-
resentable with a signed fixed-point number of typeFixedPoint#(isize, fsize)is +(2isize−^1 −
2 −fsize)≤v≤ − 2 isize−^1. The functionepsilonreturns the smallest representable quantum by
a specific type, 2−fsize. For example, a variablevof typeFixedPoint#(2,3)type can repre-
sent numbers from 1.875 (1^78 ) to− 2 .0 in intervals of^18 = 0.125, i.e. epsilon is 0.125. The type
FixedPoint#(5,0)is equivalent toInt#(5).
instance Bounded#( FixedPoint#(isize, fsize) )
provisos( Add#(isize, fsize, bsize) ) ;

```
epsilon Returns the value ofepsilonwhich is the smallest representable
quantum by a specific type, 2−fsize.
```
```
function FixedPoint#(isize, fsize) epsilon () ;
```
Bitwise Left and right shifts are provided forFixedPointvariables as part of theBitwisetype
class. Note that the shift right (>>) function does an arithmetic shift, thus preserving the sign of
the operand. Note that a right shift of 1 is equivalent to a division by 2, except when the operand is
equal to−epsilon. The functionsmsbandlsbare also provided. The other methods ofBitwise
type class are not provided since they have no operational meaning onFixedPointvariables; the
use of these generates an error message.
instance Bitwise#( FixedPoint#(isize, fsize) )
provisos( Add#(isize, fsize, bsize) );


Reference Guide Bluespec SystemVerilog

SaturatingArith TheSaturatingArithclass provides the functionssatPlus,satMinus,boundedPlus,
andboundedMinus. These are modified plus and minus functions which saturate to values defined
by theSaturationModewhen the operations would otherwise overflow or wrap-around.

instance SaturatingArith#(FixedPoint#(isize, fsize));

FShow TheFShowclass provides the functionfshowwhich can be applied to a type to create an
associatedFmtrepresentation.

instance FShow#(FixedPoint#(i,f));
function Fmt fshow (FixedPoint#(i,f) value);
Int#(i) i_part = fxptGetInt(value);
UInt#(f) f_part = fxptGetFrac(value);
return $format("<FP %b.%b>", i_part, f_part);
endfunction
endinstance

Functions

Utility functions are provided to extract the integer and fractional parts.

```
fxptGetInt Extracts the integer part of theFixedPointnumber.
```
```
function Int#(isize) fxptGetInt ( FixedPoint#(isize, fsize) x );
```
```
fxptGetFrac Extracts the fractional part of theFixedPointnumber.
```
```
function UInt#(fsize) fxptGetFrac ( FixedPoint#(isize, fsize) x );
```
To convert run-timeIntandUIntvalues to typeFixedPoint, the following conversion functions
are provided. Both of these functions invoke the necessary extension of the source operand.

```
fromInt Converts run-timeIntvalues to typeFixedPoint.
```
```
function FixedPoint#(ir,fr) fromInt( Int#(ia) inta )
provisos ( Add#(ia, xxA, ir ) ); // ir >= ia
```
```
fromUInt Converts run-timeUIntvalues to typeFixedPoint.
```
```
function FixedPoint#(ir,fr) fromUInt( UInt#(ia) uinta )
provisos ( Add#(ia, 1, ia1), // ia1 = ia + 1
Add#(ia1,xxB, ir ) ); // ir >= ia1
```

Bluespec SystemVerilog Reference Guide

Non-integer compile time constants may be specified by a rational number which is a ratio of two
integers. For example, one-third may be specified byfromRational(1,3).

```
fromRational Specify aFixedPointwith a rational number which is the ratio of two
integers.
```
```
function FixedPoint#(isize, fsize) fromRational(
Integer numerator, Integer denominator)
provisos ( Add#(isize, fsize, bsize ) ) ;
```
At times, full precision Arithmetic functions may be required, where the operands are not the same
type (sizes), as is required for the infixArithoperators. These functions do not overflow on the
result.

```
fxptAdd Function for full precision addition. The operands do not have to be of the
same type (size) and there is no overflow on the result.
```
```
function FixedPoint#(ri,rf) fxptAdd( FixedPoint#(ai,af) a,
FixedPoint#(bi,bf) b )
provisos (Max#(ai,bi,rim) // ri = 1 + max(ai, bi)
,Add#(1,rim, ri)
,Max#(af,bf,rf)); // rf = max (af, bf)
```
```
fxptSub Function for full precision subtraction where the operands do not have to
be of the same type (size) and there is no overflow on the result.
```
```
function FixedPoint#(ri,rf) fxptSub( FixedPoint#(ai,af) a,
FixedPoint#(bi,bf) b )
provisos (Max#(ai,bi,rim) // ri = 1 + max(ai, bi)
,Add#(1,rim, ri)
,Max#(af,bf,rf)); // rf = max (af, bf)
```
```
fxptMult Function for full precision multiplication, where the result is the sum of the
field sizes of the operands. The operands do not have to be of the same
type (size).
```
```
function FixedPoint#(ri,rf) fxptMult( FixedPoint#(ai,af) x,
FixedPoint#(bi,bf) y )
provisos ( Add#(ai,bi,ri) // ri = ai + bi
,Add#(af,bf,rf) // rf = af + bf
,Add#(ai,af,ab)
,Add#(bi,bf,bb)
,Add#(ab,bb,rb)
,Add#(ri,rf,rb) ) ;
```

Reference Guide Bluespec SystemVerilog

```
fxptQuot Function for full precision division where the operands do not have to be
of the same type (size).
```
```
function FixedPoint#(ri,rf) fxptQuot (FixedPoint#(ai,af) a,
FixedPoint#(bi,bf) b )
provisos (Add#(ai1,bf,ri) // ri = ai + bf + 1
,Add#(ai,1,ai1)
,Add#(af,_xf,rf)); // rf >= af
```
fxptTruncateis a general truncate function which converts variables toFixedPoint#(ai,af)to
typeFixedPoint#(ri,rf), whereai≥riandaf≥rf. This function truncates bits as appropriate
from the most significant integer bits and the least significant fractional bits.

```
fxptTruncate Truncates bits as appropriate from the most significant integer bits and the
least significant fractional bits.
```
```
function FixedPoint#(ri,rf) fxptTruncate(
FixedPoint#(ai,af) a )
provisos( Add#(xxA,ri,ai), // ai >= ri
Add#(xxB,rf,af)); // af >= rf
```
Two saturating fixed-point truncation functions are provided:fxptTruncateSatandfxptTruncateRoundSat.
They both use theSaturationMode, defined in Section B.1.12, to determine the final result.

typedef enum { Sat_Wrap
,Sat_Bound
,Sat_Zero
,Sat_Symmetric
} SaturationMode deriving (Bits, Eq);

```
fxptTruncateSat A saturating fixed point truncation. If the value cannot be represented in
its truncated form, an alternate value,minBoundormaxBound, is selected
based onsmode.
```
```
function FixedPoint#(ri,rf) fxptTruncateSat (
SaturationMode smode, FixedPoint#(ai,af) din)
provisos (Add#(ri,idrop,ai)
,Add#(rf,_f,af) );
```
The functionfxptTruncateRoundSatrounds the saturated value, as determined by the value of
rmodeof typeRoundMode. The rounding only applies to the truncation of the fractional component
of the fixed-point number, though it may cause a wrap or overflow to the integer component which
requires saturation.


Bluespec SystemVerilog Reference Guide

```
fxptTruncateRoundSat A saturating fixed point truncate function which rounds the truncated frac-
tional component as determined by the value ofrmode(RoundMode). If
the final value cannot be represented in its truncated form, theminBound
ormaxBoundvalue is returned.
```
```
function FixedPoint#(ri,rf) fxptTruncateRoundSat
(RoundMode rmode, SaturationMode smode,
FixedPoint#(ai,af) din)
provisos (Add#(ri,idrop,ai)
,Add#(rf,fdrop,af) );
```
typedef enum {
Rnd_Plus_Inf
,Rnd_Zero
,Rnd_Minus_Inf
,Rnd_Inf
,Rnd_Conv
,Rnd_Truncate
,Rnd_Truncate_Zero
} RoundMode deriving (Bits, Eq);

These modes are equivalent to the SystemC values shown in the table below. The rounding mode de-
termines how the value is rounded when the truncated value is equidistant between two representable
values.

```
Rounding Modes
RoundMode SystemC Description Action when truncated value
Equivalent equidistant between values
Rnd_Plus_Inf SCRND Round to plus infinity Always increment
Rnd_Zero SCRNDZERO Round to zero Move towards reduced mag-
nitude (decrement positive
value, increment negative
value)
Rnd_Minus_Inf SCRNDMININF Round to minus infinity Always decrement
Rnd_Inf SCRNDINF Round to infinity Always increase magnitude
Rnd_Conv SCRNDCONV Round to convergence Alternate increment and
decrement based on even and
odd values
Rnd_Truncate SCTRN Truncate, no rounding
Rnd_Truncate_Zero SCTRNZERO Truncate to zero Move towards reduced magni-
tude
```
Consider what happens when you apply the functionfxptTruncateRoundSatto a fixed-point num-
ber. The least significant fractional bits are dropped. If the dropped bits are non-zero, the remaining
fractional component rounds towards the nearest representable value. If the remaining component
is exactly equidistant between two representable values, the rounding mode (rmode) determines
whether the value rounds up or down.

The following table displays the rounding value added to the LSB of the remaining fractional com-
ponent. When the value is equidistant (1/2), the algorithm may be dependent on whether the value
of the variable is positive or negative.


Reference Guide Bluespec SystemVerilog

```
Rounding Value added to LSB of Remaining Fractional Component
RoundMode Value of Truncated Bits
<1/2 1/2 >1/2
Pos Neg
Rnd_Plus_Inf 0 1 1 1
Rnd_Zero 0 0 1 1
Rnd_Minus_Inf 0 0 0 1
Rnd_Inf 0 1 0 1
Rnd_Conv
Remaining LSB = 0 0 0 0 1
Remaining LSB = 1 0 1 1 1
```
The final two modes are truncates and are handled differently. TheRnd_Truncatemode simply drops
the extra bits without changing the remaining number. TheRnd_Truncate_Zeromode decreases
the magnitude of the variable, moving the value closer to 0. If the number is positive, the function
simply drops the extra bits, if negative, 1 is added.

```
RoundMode Sign of Argument Description
Positive Negative
Rnd_Truncate 0 0 Truncate extra bits, no rounding
Rnd_Truncate_Zero 0 1 Add 1 to negative number if trun-
cated bits are non-zero
```
Example: Truncated values by Round type, where argument isFixedPoint#(2,3)type and result
is aFixedPoint#(2,1)type. In this example, we’re rounding to the nearest 1/2, as determined by
RoundMode.

```
Result by RoundMode when SaturationMode = SatWrap
Argument RoundMode
Binary Decimal Plus_Inf Zero Minus_Inf Inf Conv Trunc Trunc_Zero
10.001 -1.875 -2.0 -2.0 -2.0 -2.0 -2.0 -2.0 -1.5
10.110 -1.250 -1.0 -1.0 -1.5 -1.5 -1.0 -1.5 -1.0
11.101 -0.375 -0.5 -0.5 -0.5 -0.5 -0.5 -0.5 0.0
00.011 0.375 0.5 0.5 0.5 0.5 0.5 0.0 0.0
01.001 1.250 1.5 1.0 1.0 1.5 1.0 1.0 1.0
01.111 1.875 -2.0 -2.0 -2.0 -2.0 -2.0 1.5 1.5
```
```
fxptSignExtend A general sign extend function which converts variables of type
FixedPoint#(ai,af)to typeFixedPoint#(ri,rf), whereai≤riand
af ≤rf. The integer part is sign extended, while additional 0 bits are
added to least significant end of the fractional part.
```
```
function FixedPoint#(ri,rf) fxptSignExtend(
FixedPoint#(ai,af) a )
provisos( Add#(xxA,ai,ri), // ri >= ai
Add#(fdiff,af,rf)); // rf >= af
```

Bluespec SystemVerilog Reference Guide

```
fxptZeroExtend A general zero extend function.
```
```
function FixedPoint#(ri,rf) fxptZeroExtend(
FixedPoint#(ai,af) a )
provisos( Add#(xxA,ai,ri), // ri >= ai
Add#(xxB,af,rf)); // rf >= af
```
DisplayingFixedPointvalues in a simple bit notation would result in a difficult to read pattern.
The following write utility function is provided to ease in their display. Note that the use of this
function adds many multipliers and adders into the design which are only used for generating the
output and not the actual circuit.

```
fxptWrite Displays aFixedPointvalue in a decimal format, wherefwidthgive the
number of digits to the right of the decimal point. fwidthmust be in
the inclusive range of 0 to 10. The displayed result is truncated without
rounding.
```
```
function Action fxptWrite( Integer fwidth,
FixedPoint#(isize, fsize) a )
provisos( Add#(i, f, b),
Add#(33,f,ff)); // 33 extra bits for computations.
```
Examples - Fixed Point Numbers

```
// The following code writes "x is 0.5156250"
FixedPoint#(1,6) x = half + epsilon ;
$write( "x is " ) ; fxptWrite( 7, x ) ; $display("" ) ;
```
ARealvalue can automatically be converted to aFixedPointvalue:

```
FixedPoint#(3,10) foo = 2e-3;
```
```
FixedPoint#(2,3) x = 1.625 ;
```
### C.5.5 NumberTypes

Package

import NumberTypes :: * ;

Description

TheNumberTypespackage defines two new number types for use as index types: BuffIndexand
WrapNumber.

ABuffIndex#(sz, ln)is an unsigned integer which wraps around, whereszis the number of bits
in its representation andlnis the size of the buffer it is to index. Oftenszwill beTLog#(ln).
BuffIndexis intended to be used as the index type for buffers of arbitrary size. The values of
BuffIndexare not ordered; you cannot determine which of two values is ahead of the other because
of the wrap-around.


Reference Guide Bluespec SystemVerilog

AWrapNumber#(sz)is an unsigned integer which wraps around, whereszis the number of bits in
its representation. The range is the entire value space (i.e.2sz), but should be used in situations
where at any time all valid values are in at most half of that space. The ordering of values can
be defined taking wrap-around into account, so that the nearer distance apart is used to determine
which value is ahead of the other.

This package is provided as both a compiled library package and as BSV source code to facilitate
customization. The source code file can be found in the$BLUESPECDIR/BSVSource/Mathdirectory.
To customize a package, copy the file into a local directory and then include the local directory in
the path when compiling. This is done by specifying the search path with the-poption as described
in the BSV Users Guide.

Types and type classes

ABuffIndexhas two numeric type parameters: the size in bits of the representation (sz), and the
length of the buffer it is to index (ln).

typedef struct { UInt#(sz) bix; } BuffIndex#(numeric type sz, numeric type ln)
deriving (Bits, Eq);

AWrapNumber#(sz)has a single numeric type parameter,sz, which is the size in bits of the repre-
sentation.

typedef struct { UInt#(sz) wn; } WrapNumber#(numeric type sz)
deriving (Bits, Eq, Arith, Literal, Bounded);

Both types belong to theBits,Eq,Arith, andLiteraltypeclasses. TheWrapNumbertype also
belongs to theOrdtypeclass. Each type class definition includes functions which are then also
defined for the data type. The Prelude library definitions (Section B) describes which functions are
defined for each type class.

```
Type Classes used byBuffIndexandWrapNumber
Bits Eq Literal Arith Ord Bounded Bit Bit Bit
wise Reduction Extend
WrapNumber
```
##### √ √ √ √ √ √

```
BuffIndex
```
##### √ √ √ √

Literal BothBuffIndexandWrapNumberbelong to theLiteraltypeclass, which allows conversion
from (compile-time) typeIntegerto these types.

For theBuffIndextype, thefromIntegerandinLiteralRangefunctions are defined as:

instance Literal#(BuffIndex#(sz,ln));
function fromInteger(i) = BuffIndex {bix: fromInteger(i) };
function inLiteralRange(x,i) = (i>=0 && i < valueof(ln));
endinstance

Arith The type classArithdefines the common infix operators. Addition and subtraction are the
only meaningful arithmetic operations forWrapNumberandBuffIndex.


Bluespec SystemVerilog Reference Guide

Ord WrapNumberbelongs to theOrdtypeclass, so values ofWrapNumbercan be compared by the
relational operators<,>,<=, and>=. Since the ordering ofWrapNumbertypes takes into account
wrap-around, the nearer distance apart is used to determine which value is ahead of the other.

Functions

Utility functions to convert aBuffIndexto aUIntand for adding and subtractingBuffIndexand
UIntvalues are provided.

```
unwrapBI Converts aBuffIndexto aUInt
```
```
function UInt#(sz) unwrapBI(BuffIndex#(sz,ln) x);
```
```
addBIUInt Adds aUIntto aBuffIndex, returning aBuffIndex
```
```
function BuffIndex#(sz,ln) addBIUInt(BuffIndex#(sz,ln) bin,
UInt#(sz) i);
```
```
sbtrctBIUInt Subtracts aUIntfrom aBuffIndex, returning aBuffIndex
```
```
function BuffIndex#(sz,ln) sbtrctBIUInt(BuffIndex#(sz,ln) bin,
UInt#(sz) i);
```
Utility functions to convert between aWrapNumberand aUInt, and a function to add aUIntto a
WrapNumberare provided.

```
wrap Converts aUIntto aWrapNumber
```
```
function WrapNumber#(sz) wrap(UInt#(sz) x) ;
```
```
unwrap Converts aWrapNumberto aUInt
```
```
function UInt#(sz) unwrap (WrapNumber#(sz) x);
```
```
addUInt Adds aUIntto aWrapNumber, returning aWrapNumber
```
```
function WrapNumber#(sz) addUInt(WrapNumber#(sz) wn,
UInt#(sz) i) ;
```
## C.6 FSM

### C.6.1 StmtFSM

Package

import StmtFSM :: * ;


```
Reference Guide Bluespec SystemVerilog
```
```
Description
```
```
TheStmtFSMpackage provides a procedural way of defining finite state machines (FSMs) which are
automatically synthesized.
```
```
First, one uses theStmtsublanguage to compose the actions of an FSM using sequential, parallel,
conditional and looping structures. This sublanguage is within theexpressionsyntactic category,
i.e., a term in the sublanguage is an expression whose value is of typeStmt. This value can be bound
to identifiers, passed as arguments and results of functions, held in static data structures, etc., like
any other value. Finally, the FSM can be instantiated into hardware, multiple times if desired, by
passing theStmtvalue to the module constructormkFSM. The resulting module interface has type
FSM, which has methods to start the FSM and to wait until it completes.
```
```
TheStmtsublanguage
The state machine is automatically constructed from the procedural description given in theStmt
definition. Appropriate state counters are created and rules are generated internally, corresponding
to the transition logic of the state machine. The use of rules for the intermediate state machine
generation ensures that resource conflicts are identified and resolved, and that implicit conditions
are properly checked before the execution of any action.
```
The names of generated rules (which may appear in conflict warnings) have suffixes of the form
“l<nn>c<nn>”, where the<nn>are line or column numbers, referring to the statement which gave
rise to the rule.

```
A term in theStmtsublanguage is an expression, introduced at the outermost level by the keywords
seqorpar. Note that within the sublanguage,if,whileandforstatements are interpreted
as statements in the sublanguage and not as ordinary statements, except when enclosed within
action/endactionkeywords.
```
```
exprPrimary ::= seqFsmStmt|parFsmStmt
```
```
fsmStmt ::= exprFsmStmt
| seqFsmStmt
| parFsmStmt
| ifFsmStmt
| whileFsmStmt
| repeatFsmStmt
| forFsmStmt
| returnFsmStmt
```
```
exprFsmStmt ::= regWrite;
| expression;
```
```
seqFsmStmt ::= seqfsmStmt{ fsmStmt }endseq
```
```
parFsmStmt ::= parfsmStmt{ fsmStmt }endpar
```
```
ifFsmStmt ::= ifexpression fsmStmt
[ elsefsmStmt ]
```
```
whileFsmStmt ::= while (expression)
loopBodyFsmStmt
```
```
forFsmStmt ::= for (fsmStmt;expression;fsmStmt)
loopBodyFsmStmt
```
```
returnFsmStmt ::= return ;
```
```
repeatFsmStmt ::= repeat (expression)
loopBodyFsmStmt
```

Bluespec SystemVerilog Reference Guide

```
loopBodyFsmStmt ::= fsmStmt
| break ;
| continue ;
```
The simplest kind of statement is anexprFsmStmt, which can be a register assignment or, more
generally, any expression of typeAction (including action method calls andaction-endaction
blocks or of typeStmt. Statements of typeActionexecute within exactly one clock cycle, but of
course the scheduling semantics may affect exactly which clock cycle it executes in. For example, if
the actions in a statement interfere with actions in some other rule, the statement may be delayed
by the schedule until there is no interference. In all the descriptions of statements below, the
descriptions of time taken by a construct are minimum times; they could take longer because of
scheduling semantics.

Statements can be composed into sequential, parallel, conditional and loop forms. In the sequential
form (seq-endseq), the contained statements are executed one after the other. Theseqblock
terminates when its last contained statement terminates, and the total time (number of clocks) is
equal to the sum of the individual statement times.

In the parallel form (par-endpar), the contained statements (“threads”) are all executed in parallel.
Statements in each thread may or may not be executed simultaneously with statements in other
threads, depending on scheduling conflicts; if they cannot be executed simultaneously they will be
interleaved, in accordance with normal scheduling. The entireparblock terminates when the last
of its contained threads terminates, and the minimum total time (number of clocks) is equal to the
maximum of the individual thread times.

In the conditional form (if (b)s 1 elses 2 ), the boolean expressionbis first evaluated. If true,
s 1 is executed, otherwises 2 (if present) is executed. The total time taken istcycles, if the chosen
branch takestcycles.

In thewhile (b)sloop form, the boolean expressionbis first evaluated. If true,sis executed, and
the loop is repeated. Each time the condition evaluates true , the loop body is executed, so the total
time isn×tcycles, wherenis the number of times the loop is executed (possibly zero) andtis the
time for the loop body statement.

Thefor (s 1 ;b;s 2 )sBloop form is equivalent to:

```
s 1 ; while (b) seq sB; s 2 endseq
```
i.e., the initializers 1 is executed first. Then, the conditionbis executed and, if true, the loop body
sBis executed followed by the “increment” statements 2. Theb,sB,s 2 sequence is repeated as long
asbevaluates true.

Similarly, therepeat (n) sBloop form is equivalent to:

```
while (repeat_count < n) seq sB; repeat_count <=repeat_count+ 1endseq
```
where the value ofrepeatcountis initialized to 0. During execution, the condition (repeatcount <
n) is executed and, if true, the loop bodysBis executed followed by the “increment” statement
repeatcount <=repeatcount+ 1. The sequence is repeated as long asrepeatcount < nevaluates
true.

In all the loop forms, the loop body statements can contain the keywordscontinueorbreak, with
the usual semantics, i.e.,continueimmediately jumps to the start of the next iteration, whereas
breakjumps out of the loop to the loop sequel.

It is important to note that this use of loops, within aStmtcontext, expresses time-based (temporal)
behavior.

Interfaces and Methods


Reference Guide Bluespec SystemVerilog

Two interfaces are defined with this package,FSMandOnce. TheFSMinterface defines a basic state
machine interface while theOnceinterface encapsulates the notion of an action that should only be
performed once. AStmtvalue can be instatiated into a module that presents an interface of type
FSM.

There is a one clock cycle delay after thestartmethod is asserted before the FSM starts. This
insulates thestartmethod from many of the FSM schedule constraints that change depending on
what computation is included in each specific FSM. Therefore, it is possible that the StmtFSM is
enabled when thestartmethod is called, but not on the next cycle when the FSM actually starts.
In this case, the FSM will stall until the conditions allow it to continue.

```
Interfaces
Name Description
FSM The state machine interface
Once Used when an action should only be performed once
```
- FSMInterface
    TheFSMinterface provides four methods;start,waitTillDone,doneandabort. Once in-
    stantiated, the FSM can be started by calling thestartmethod. One can wait for the FSM
    to stop running by waiting explicitly on the boolean value returned by thedonemethod. The
    donemethod isTruebefore the FSM has run the first time. Alternatively, one can use the
    waitTillDonemethod in any action context (including from within another FSM), which (be-
    cause of an implicit condition) cannot execute until this FSM is done. The user must not use
    waitTillDoneuntil after the FSM has been started because the FSM comes out of a reset as
    done. Theabortmethod immediately exits the execution of the FSM.

```
interface FSM;
method Action start();
method Action waitTillDone();
method Bool done();
method Action abort();
endinterface: FSM
```
```
FSMInterface
Methods
Name Type Description
start Action Begins state machine execution. This can only be called
when the state machine is not executing.
waitTillDone Action Does not do any action, but is only ready when the state
machine is done.
done Bool Asserted when the state machine is done and is ready to
rerun. State machine comes out of reset as done.
abort Action Exits execution of the state machine.
```
- OnceInterface
    TheOnceinterface encapsulates the notion of an action that should only be performed once.
    Thestartmethod performs the action that has been encapuslated in theOncemodule. After
    starthas been calledstartcannot be called again (an implicit condition will enforce this).
    If theclearmethod is called, thestartmethod can be called once again.

```
interface Once;
method Action start();
method Action clear();
method Bool done() ;
endinterface: Once
```

Bluespec SystemVerilog Reference Guide

```
OnceInterface
Methods
Name Type Description
start Action Performs the action that has been encapsulated in the
Oncemodule, but oncestarthas been called it cannot
be called again (an implicit condition will enforce this).
clear Action If theclearmethod is called, thestartmethod can be
called once again.
done Bool Asserted when the state machine is done and is ready to
rerun.
```
Modules

Instantiation is performed by passing aStmtvalue into the module constructormkFSM. The state
machine is automatically constructed from the procedural decription given in the definition described
by state machine of typeStmtnamedseq_stmt. During construction, one or more registers of
appropriate widths are created to track state execution. Uponstartaction, the registers are loaded
and subsequent state changes then decrement the registers.

```
module mkFSM#( Stmt seq_stmt ) ( FSM );
```
ThemkFSMWithPredmodule is likemkFSMabove, except that the module constructor takes an ad-
ditional boolean argument (the predicate). The predicate condition is added to the condition of
each rule generated to create the FSM. This capability is useful when using the FSM in conjuction
with other rules and/or FSMs. It allows the designer to explicitly specify to the compiler the condi-
tions under which the FSM will run. This can be used to eliminate spurious rule conflict warnings
(between rules in the FSM and other rules in the design).

```
module mkFSMWithPred#( Stmt seq_stmt, Bool pred ) ( FSM );
```
ThemkAutoFSMmodule is also likemkFSMabove, except the state machine runs automatically im-
mediately after reset and a$finish(0)is called upon completion. This is useful for test benches.
Thus, it has no interface, that is, it has an empty interface.

```
module mkAutoFSM#( seq_stmt ) ();
```
ThemkOncefunction is used to create aOnceinterface where the action argument has been encap-
sulated and will be performed whenstartis called.

```
module mkOnce#( Action a ) ( Once );
```
The implementation forOnceis a 1 bit state machine (with a state register namedonceReady)
allowing the action argument to occur only one time. The ready bit is initiallyTrueand then
cleared when the action is performed. It might not be performed right away, because of implicit
conditions or scheduling conflicts.


Reference Guide Bluespec SystemVerilog

```
Name BSV Module Declaration Description
mkFSM
module mkFSM#(Stmt seq_stmt)(FSM);
```
```
Instantiate aStmtvalue into a mod-
ule that presents an interface of type
FSM.
mkFSMWithPred
module mkFSMWithPred#(Stmt seq_stmt,
Bool pred)(FSM);
```
```
LikemkFSM, except that the module
constructor takes an additional pred-
icate condition as an argument. The
predicate condition is added to the
condition of each rule generated to
create the FSM.
mkAutoFSM
module mkAutoFSM#(Stmt seq_stmt)();
```
```
Like mkFSM, except that state ma-
chine simulation is automatically
started and a$finish(0)) is called
upon completion.
mkOnce
module mkOnce#( Action a )( Once );
```
```
Used to create aOnceinterface where
the action argument has been encap-
sulated and will be performed when
startis called.
```
Functions

There are two functions,awaitanddelay, provided by theStmtFSMpackage.

Theawaitfunction is used to create an action which can only execute when the condition isTrue.
The action does not do anything.awaitis useful to block the execution of an action until a condition
becomesTrue.

Thedelayfunction is used to executenoActionfor a specified number of cycles. The function is
provided the value of the delay and returns aStmt.

```
Name Function Declaration Description
await
function Action await( Bool cond ) ;
```
```
Creates an Action which does nothing,
but can only execute when the condi-
tion isTrue.
delay
function Stmt delay( a_type value ) ;
```
```
Creates a Stmt which executes
noActionforvaluenumber of cycles.
a_typemust be in the Arith class and
Bits class and<32 bits.
```
Example - Initializing a single-ported SRAM.

Since the SRAM has only a single port, we can write to only one location in each clock. Hence, we
need to express a temporal sequence of writes for all the locations to be initialized.

```
Reg#(int) i <- mkRegU; // instantiate register with interface i
Reg#(int) j <- mkRegU; // instantiate register with interface j
```
```
// Define fsm behavior
Stmt s = seq
for (i <= 0; i < M; i <= i + 1)
for (j <= 0; j < N; j <= j + 1)
sram.write (i, j, i+j);
```

Bluespec SystemVerilog Reference Guide

```
endseq;
```
```
FSM fsm(); // instantiate FSM interface
mkFSM#(s) (fsm); // create fsm with interface fsm and behavior s
```
```
...
```
```
rule initSRAM (start_reset);
fsm.start; // Start the fsm
endrule
```
When thestart_resetsignal is true, the rule kicks off the SRAM initialization. Other rules can
wait onfsm.done, if necessary, for the SRAM initialization to be completed.

In this example, theseq-endseqbrackets are used to enter theStmtsublanguage, and thenfor
representsStmtsequencing (instead of its usual role of static generation). Sinceseq-endseqcontains
only one statement (the loop nest),par-endparbrackets would have worked just as well.

Example - Defining and instantiating a state machine.

import StmtFSM :: *;
import FIFO :: *;

module testSizedFIFO();

```
// Instantiation of DUT
FIFO#(Bit#(16)) dut <- mkSizedFIFO(5);
```
```
// Instantiation of reg’s i and j
Reg#(Bit#(4)) i <- mkRegA(0);
Reg#(Bit#(4)) j <- mkRegA(0);
```
```
// Action description with stmt notation
Stmt driversMonitors =
(seq
// Clear the fifo
dut.clear;
```
```
// Two sequential blocks running in parallel
par
// Enque 2 times the Fifo Depth
for(i <= 1; i <= 10; i <= i + 1)
seq
dut.enq({0,i});
$display(" Enque %d", i);
endseq
```
```
// Wait until the fifo is full and then deque
seq
while (i < 5)
seq
noAction;
endseq
while (i <= 10)
action
```

Reference Guide Bluespec SystemVerilog

```
dut.deq;
$display("Value read %d", dut.first);
endaction
endseq
```
```
endpar
```
```
$finish(0);
endseq);
```
```
// stmt instantiation
FSM test <- mkFSM(driversMonitors);
```
```
// A register to control the start rule
Reg#(Bool) going <- mkReg(False);
```
// This rule kicks off the test FSM, which then runs to completion.
rule start (!going);
going <= True;
test.start;
endrule
endmodule

Example - Defining and instantiating a state machine to control speed changes

import StmtFSM::*;
import Common::*;

interface SC_FSM_ifc;
method Speed xcvrspeed;
method Bool devices_ready;
method Bool out_of_reset;
endinterface

module mkSpeedChangeFSM(Speed new_speed, SC_FSM_ifc ifc);
Speed initial_speed = FS;

```
Reg#(Bool) outofReset_reg <- mkReg(False);
Reg#(Bool) devices_ready_reg <- mkReg(False);
Reg#(Speed) device_xcvr_speed_reg <- mkReg(initial_speed);
```
```
// the following lines define the FSM using the Stmt sublanguage
// the state machine is of type Stmt, with the name speed_change_stmt
Stmt speed_change_stmt =
(seq
action outofReset_reg <= False; devices_ready_reg <= False; endaction
noAction; noAction; // same as: delay(2);
```
```
device_xcvr_speed_reg <= new_speed;
noAction; noAction; // same as: delay(2);
```
```
outofReset_reg <= True;
if (device_xcvr_speed_reg==HS)
seq noAction; noAction; endseq
// or seq delay(2); endseq
```

Bluespec SystemVerilog Reference Guide

```
else
seq noAction; noAction; noAction; noAction; noAction; noAction; endseq
// or seq delay(6); endseq
devices_ready_reg <= True;
endseq);
// end of the state machine definition
```
```
// the statemachine is instantiated using mkFSM
FSM speed_change_fsm <- mkFSM(speed_change_stmt);
```
```
// the rule change_speed starts the state machine
// the rule checks that previous actions of the state machine have completed
rule change_speed ((device_xcvr_speed_reg != new_speed || !outofReset_reg) &&
speed_change_fsm.done);
speed_change_fsm.start;
endrule
```
method xcvrspeed = device_xcvr_speed_reg;
method devices_ready = devices_ready_reg;
method out_of_reset = outofReset_reg;
endmodule

Example - Defining a state machine and using theawaitfunction

```
// This statement defines this brick’s desired behavior as a state machine:
// the subcomponents are to be executed one after the other:
Stmt brickAprog =
seq
// Since the following loop will be executed over many clock
// cycles, its control variable must be kept in a register:
for (i <= 0; i < 0-1; i <= i+1)
// This sequence requests a RAM read, changing the state;
// then it receives the response and resets the state.
seq
action
// This action can only occur if the state is Idle
// the await function will not let the statements
// execute until the condition is met
await(ramState==Idle);
ramState <= DesignReading;
ram.request.put(tagged Read i);
endaction
action
let rs <- ram.response.get();
ramState <= Idle;
obufin.put(truncate(rs));
endaction
endseq
// Wait a little while:
for (i <= 0; i < 200; i <= i+1)
action
endaction
// Set an interrupt:
action
inrpt.set;
```

Reference Guide Bluespec SystemVerilog

```
endaction
endseq
);
// end of the state machine definition
```
```
FSM brickAfsm <- mkFSM#(brickAprog); //instantiate the state machine
```
```
// A register to remember whether the FSM has been started:
Reg#(Bool) notStarted();
mkReg#(True) the_notStarted(notStarted);
```
```
// The rule which starts the FSM, provided it hasn’t been started
// previously and the brick is enabled:
rule start_Afsm (notStarted && enabled);
brickAfsm.start; //start the state machine
notStarted <= False;
endrule
```
Creating FSM Server Modules

Instantiation of an FSM server module is performed in a manner analogous to that of a standard FSM
module constructor (such asmkFSM). WhereasmkFSMtakes aStmtvalue as an argument, howver,
mkFSMServertakes a function as an argument. More specifically, the argument tomkFSMServeris a
function which takes an argument of typeaand returns a value of typeRStmt#(b).

```
module mkFSMServer#(function RStmt#(b) seq_func (a input)) ( FSMServer#(a, b) );
```
TheRStmttype is a polymorphic generalization of theStmttype. A sequence of typeRStmt#(a)
allows valuedreturnstatements (where the return value is of typea). Note that theStmttype is
equivalent toRStmt#(Bit#(0)).

typedef RStmt#(Bit#(0)) Stmt;

ThemkFSMServermodule constructor provides an interface of typeFSMServer#(a, b).

interface FSMServer#(type a, type b);
interface Server#(a, b) server;
method Action abort();
endinterface

TheFSMServerinterface has one subinterface of typeServer#(a, b)(from theClientServer
package) as well as an `Action` method calledabort; Theabortmethod allows the FSM inside the
FSMServermodule to be halted if the client FSM is halted.

AnFSMServermodule is accessed using thecallServerfunction from within an FSM statement
block.callServertakes two arguments. The first is the interface of theFSMServermodule. The
second is the input value being passed to the module.

result <- callServer(serv_ifc, value);

Note the special left arrow notation that is used to pass the server result to a register (or more
generally to any state element with aReginterface). A simple example follows showing the definition
and use of amkFSMServermodule.

Example - Defining and instantiating an FSM Server Module


Bluespec SystemVerilog Reference Guide

```
// State elements to provide inputs and store results
Reg#(Bit#(8)) count <- mkReg(0);
Reg#(Bit#(16)) partial <- mkReg(0);
Reg#(Bit#(16)) result <- mkReg(0);
```
```
// A function which creates a server sequence to scale a Bit#(8)
// input value by and integer scale factor. The scaling is accomplished
// by a sequence of adds.
function RStmt#(Bit#(16)) scaleSeq (Integer scale, Bit#(8) value);
seq
partial <= 0;
repeat (fromInteger(scale))
action
partial <= partial + {0,value};
endaction
return partial;
endseq;
endfunction
```
```
// Instantiate a server module to scale the input value by 3
FSMServer#(Bit#(8), Bit#(16)) scale3_serv <- mkFSMServer(scaleSeq(3));
```
```
// A test sequence to apply the server
let test_seq = seq
result <- callServer(scale3_serv, count);
count <= count + 1;
endseq;
```
```
let test_fsm <- mkFSM(test_seq);
```
```
// A rule to start test_fsm
rule start;
test_fsm.start;
endrule
// finish after 6 input values
rule done (count == 6);
$finish;
endrule
```
## C.7 Connectivity

The packages in this section provide useful components, primarily interfaces, to connect hardware
elements in a design.

The basic interfaces,GetandPutare defined in the packageGetPut. The typeclassConnectable
indicates that two related types can be connected together. The packageClientServerprovides
interfaces usingGetandPutfor modules that have a request-response type of interface. The package
CGetPutdefines a type of theGetandPutinterfaces that is implemented with a credit based FIFO.

### C.7.1 GetPut

Package

import GetPut :: *;


Reference Guide Bluespec SystemVerilog

Description

A common paradigm between two blocks is the get/put mechanism: one sidegetsor retrieves an
item from an interface and the other sideputsor gives an item to an interface. These types of
interfaces are used inTransaction Level Modelingor TLM for short. This pattern is so common in
system design that BSV provides theGetPutlibrary package for this purpose.

TheGetPutpackage provides basic interfaces to implement the TLM paradigm, along with interface
transformer functions and modules to transform to/from FIFO implementations. TheClientServer
package in Section C.7.3 defines more complex interfaces based on theGetandPutinterfaces
to support request-response interfaces. TheGetPutpackage must be imported when using the
ClientServerpackage.

Typeclasses

TheGetPutpackage defines two typeclasses: ToGetandToPut. The types with instances defined
in these typeclasses provide the functionstoGetandtoPut, used to create associatedGetandPut
interfaces from these other types.

ToGetdefines the class to which the functiontoGetcan be applied to create an associatedGet
interface.

typeclass ToGet#(a, b);
function Get#(b) toGet(a ax);
endtypeclass

ToPutdefines the class to which the functiontoPutcan be applied to create an associatedPut
interface.

typeclass ToPut#(a, b);
function Put#(b) toPut(a ax);
endtypeclass

Instances ofToGetandToPutare defined for the following interfaces:

```
Defined Instances forToGetandToPut
Type (Interface) toGet toPut Comments
a D toGetreturns valuea
ActionValue#(a) D toGetperforms the Action and returns the value
function Action fn(a) D toPutcallsActionfunctionfnwith argumenta
Get#(a) D identity function: returnsGet#(a)
Put#(a) D identity function: returnsPut#(a)
Reg#(a) D D toGetreturns_read,toPutcalls_write
RWire#(a) D D toGetreturnswget,toPutcallswset
ReadOnly#(a) D toGetreturns_read
FIFO#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
FIFOF#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
SyncFIFOIfc#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
FIFOLevelIfc#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
SyncFIFOLevelIfc#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
FIFOCountIfc#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
SyncFIFOCountIfc#(a) D D toGetcallsdeqreturnsfirst,toPutcallsenq
```
Example - UsingtoPut


Bluespec SystemVerilog Reference Guide

module mkTop (Put#(UInt#(64)));
Reg#(UInt#(64)) inValue <- mkReg(0);
Reg#(Bool) startit <- mkReg(True);
...
StimIfc stim_gen <- mkStimulusGen;

rule startTb (startit && inValue!=0);
// Get the value
let val = inValue;
stim_gen.start(val);
startit <= False;
endrule
...
return (toPut(asReg(inValue)));
endmodule: mkTop

Interfaces and methods

TheGetinterface defines thegetmethod, similar to adequeue, which retrieves an item from an
interface and removes it at the same time. ThePutinterface defines theputmethod, similar to an
enqueue, which gives an item to an interface. Also provided is theGetSinterface, which defines
separate methods for the dequeue (deq) and retreiving the item (first) from the interface.

You can design your ownGetandPutinterfaces with implicit conditions on theget/putto ensure
that theget/putis not performed when the module is not ready. This would ensure that a rule
containinggetmethod would not fire if the element associated with it is empty and that a rule
containingputmethod would not fire if the element is full.

The following interfaces are defined in theGetPutpackage. They each take a single parameter,
element_typewhich must be in theBitstypeclass.

```
Interfaces defined in GetPut
Interface
Name
```
```
Description Methods Type
```
```
Get Retrieves item from an interface get ActionValue
Put Adds an item to an interface put Action
GetS Retrieves an item from an interface with 2 methods, first Value
separating the return of the value from the dequeue deq Action
GetPut Combination of aGetand aPutin aTuple2 get ActionValue
put Action
```
Get

TheGetinterface is where you retrieve (get) data from an object. TheGetinterface is provides a
single ActionValue method,get, which retrieves an item of data from an interface and removes it
from the object. Agetis similar to adequeue, but it can be associated with any interface. AGet
interface is more abstract than aFIFOinterface; it does not describe the underlying hardware.

```
Get
Method Argument
Name Type Description Name Description
get ActionValue returns an item from an
interface and removes it
from the object
```

Reference Guide Bluespec SystemVerilog

interface Get#(type element_type);
method ActionValue#(element_type) get();
endinterface: Get

Example - adding your ownGetinterface:

module mkMyFifoUpstream (Get#(int));
...
method ActionValue#(int) get();
f.deq;
return f.first;
endmethod

Put

ThePutinterface is where you can give (put) data to an object. ThePutinterface provides a single
Action method,put, which gives an item to an interface. Aputis similar to anenqueue, but it can
be associated with any interface. APutinterface is more abstract than aFIFOinterface; it does not
describe the underlying hardware.

```
Put
Method Argument
Name Type Description Name Description
put Action gives an item to an interface x1 data to be added to the object
must be of typeelement_type
```
interface Put#(type element_type);
method Action put(element_type x1);
endinterface: Put

Example - adding your ownPutinterface:

module mkMyFifoDownstream (Put#(int));
...
method Action put(int x);
F.enq(x);
endmethod

GetS

TheGetSinterface is like aGetinterface, but separates thegetmethod into two methods: afirst
and adeq.

```
GetS
Method Argument
Name Type Description Name Description
first Value returns an item from the
interface
deq Action Removes the item from
the interface
```
interface GetS#(type element_type);
method element_type first();
method Action deq();
endinterface: GetS


Bluespec SystemVerilog Reference Guide

GetPut

The library also defines an interfaceGetPutwhich associatesGetandPutinterfaces into aTuple2.

typedef Tuple2#(Get#(element_type), Put#(element_type)) GetPut#(type element_type);

Type classes

The classConnectable(Section C.7.2) is meant to indicate that two related types can be connected
in some way. It does not specify the nature of the connection.

AGetandPutis an example of connectable items. One object willputan element into the interface
and the other object willgetthe element from the interface.

instance Connectable#(Get#(element_type), Put#(element_type));

Modules

There are three modules provided by theGetPutpackage which provide theGetPutinterface with
a type ofFIFO. These FIFOs useGetandPutinterfaces instead of the usualenqinterfaces. To use
any of these modules theFIFOpackage must be imported. You can also write your own modules
providing aGetPutinterface for other hardware structures.

```
mkGPFIFO Creates a FIFO of depth 2 with aGetPutinterface.
```
```
module mkGPFIFO (GetPut#(element_type))
provisos (Bits#(element_type, width_elem));
```
```
mkGPFIFO1 Creates a FIFO of depth 1 with aGetPutinterface.
```
```
module mkGPFIFO1 (GetPut#(element_type))
provisos (Bits#(element_type, width_elem));
```
```
mkGPSizedFIFO Creates a FIFO of depth n with aGetPutinterface.
```
```
module mkGPSizedFIFO# (Integer n) (GetPut#(element_type))
provisos (Bits#(element_type, width_elem));
```
Functions

There are three functions defined in theGetPutpackage that change aFIFOinterface to aGet,GetS
orPutinterface. Given a FIFO we can use the functionfifoToGetto obtain aGetinterface, which
is a combination ofdeqandfirst. Given a FIFO we can use the functionfifoToPutto obtain a
Putinterface usingenq. The functionstoGetandtoPut(C.7.1) are recommended instead of the
fifoToGetandfifoToPutfunctions. The functionfifoToGetSreturns theGetSmethods as fifo
methods.


Reference Guide Bluespec SystemVerilog

```
fifoToGet Returns aGetinterface. It is recommended that you use the functiontoGet
(C.7.1) instead of this function.
```
```
function Get#(element_type) fifoToGet(FIFO#(element_type) f);
```
```
fifoToGetS Returns aGetSinterface.
```
```
function GetS#(element_type) fifoToGet(FIFO#(element_type) f);
```
```
fifoToPut Returns aPutinterface. It is recommended that you use the functiontoPut
(C.7.1) instead of this function.
```
```
function Put#(element_type) fifoToPut(FIFO#(element_type) f);
```
Example of creating a FIFO with aGetPutinterface

```
import GetPut::*;
import FIFO::*;
```
```
...
module mkMyModule (MyInterface);
GetPut#(StatusInfo) aFifoOfStatusInfoStructures <- mkGPFIFO;
...
endmodule: mkMyModule
```
Example of a protocol monitor

This is an example of how you might write a protocol monitor that watches bus traffic between a
bus and a bus target device

```
import GetPut::*;
import FIFO::*;
```
```
// Watch bus traffic beteween a bus and a bus target
interface ProtocolMonitorIfc;
// These subinterfaces are defined inside the module
interface Put#(Bus_to_Target_Request) bus_to_targ_req_ifc;
interface Put#(Target_to_Bus_Response) targ_to_bus_resp_ifc;
endinterface
...
module mkProtocolMonitor (ProtocolMonitorIfc);
// Input FIFOs that have Put interfaces added a few lines down
FIFO#(Bus_to_Target_Request) bus_to_targ_reqs <- mkFIFO;
FIFO#(Target_To_Bus_Response) targ_to_bus_resps <- mkFIFO;
...
// Define the subinterfaces: attach Put interfaces to the FIFOs, and
// then make those the module interfaces
interface bus_to_targ_req_ifc = fifoToPut (bus_to_targ_reqs);
```

Bluespec SystemVerilog Reference Guide

```
interface targ_to_bus_resp_ifc = fifoToPut (targ_to_bus_resps);
end module: mkProtocolMonitor
```
```
// Top-level module: connect mkProtocolMonitor to the system:
module mkSys (Empty);
ProtocolMonitorIfc pmon <- mkProtocolInterface;
```
```
rule pass_bus_req_to_interface;
let x <- bus.bus_ifc.get; // definition not shown
pmon.but_to_targ_ifc.put (x);
endrule
```
```
endmodule: mkSys
```
### C.7.2 Connectable

Package

import Connectable :: * ;

Description

TheConnectable package contains the definitions for the classConnectable and instances of
Connectables.

Types and Type-Classes

The classConnectableis meant to indicate that two related types can be connected in some way.
It does not specify the nature of the connection. TheConnectablestype class defines the module
mkConnection, which is used to connect the pairs.

typeclass Connectable#(type a, type b);
module mkConnection#(a x1, b x2)(Empty);
endtypeclass

Instances

Get and Put One instance of the typeclass ofConnectableisGetandPut. One object willput
an element into an interface and the other object willgetthe element from the interface.

instance Connectable#(Get#(a), Put#(a));

Tuples If we haveTuple2of connectable items then the pair is also connectable, simply by con-
necting the individual items.

instance Connectable#(Tuple2#(a, c), Tuple2#(b, d))
provisos (Connectable#(a, b), Connectable#(c, d));

The proviso shows that the first component of one tuple connects to the first component of the other
tuple, likewise, the second components connect as well. In the above statement,aconnects toband
cconnects tod. This is used byClientServer(Section C.7.3) to connect theGetof theClientto
thePutof theServerand visa-versa.

This is extensible to all Tuples (Tuple3,Tuple4, etc.). As long as the items are connectable, the
Tuples are connectable.


Reference Guide Bluespec SystemVerilog

Vector TwoVectors are connectable if their elements are connectable.

instance Connectable#(Vector#(n, a), Vector#(n, b))
provisos (Connectable#(a, b));

ListN TwoListNs are connectable if their elements are connectable.

instance Connectable#(ListN#(n, a), ListN#(n, b))
provisos (Connectable#(a, b));

Action, ActionValue AnActionValue method (or function) which produces a value can be
connected to an `Action` method (or function) which takes that value as an argument.

instance Connectable#(ActionValue#(a), function Action f(a x));

instance Connectable#(function Action f(a x), ActionValue#(a));

AValuemethod (or value) can be connected to an `Action` method (or function) which takes that
value as an argument.

instance Connectable#(a, function Action f(a x));

instance Connectable#(function Action f(a x), a);

Inout Inouts are connectable via theConnectabletypeclass. The use ofmkConnectioninstanti-
ates a Verilog moduleInoutConnect. TheInouts must be on the same clock and the same reset.
The clock and reset of theInouts may be different than the clock and reset of the parent module of
themkConnection.

instance Connectable#(Inout#(a, x1), Inout#(a, x2))
provisos (Bit#(a,sa));

### C.7.3 ClientServer

Package

import ClientServer :: * ;

Description

TheClientServerpackage provides two interfaces,ClientandServerwhich can be used to define
modules which have a request-response type of interface. TheGetPutpackage must be imported
when using this package because theGetandPutinterface types are used.

Interfaces and methods

The interfacesClientandServercan be used for modules that have a request-response type of
interface (e.g. a RAM). The server accepts requests and generates responses, the client accepts
responces and generates requests. There are no assumptions about how many (if any) responses a
request generates

```
Interfaces
Interface Name Parameter name Parameter Description Restrictions
Client reqtype type of the client request must be in the Bits class
resptype type of the client response must be in the Bits class
Server reqtype type of the server request must be in the Bits class
resptype type of the server response must be in the Bits class
```

Bluespec SystemVerilog Reference Guide

Client

TheClientinterface provides two subinterfaces,requestandresponse. From aClient, onegets
a request andputsa response.

```
Client SubInterface
Name Type Description
request Get#(req_type) the interface through which the outside world
retrieves (gets) a request
response Put#(resp_type) the interface through which the outside world
returns (puts) a response
```
interface Client#(type req_type, type resp_type);
interface Get#(req_type) request;
interface Put#(resp_type) response;
endinterface: Client

Server

TheServerinterface provides two subinterfaces,requestandresponse. From aServer, oneputs
a request andgetsa response.

```
Server SubInterface
Name Type Description
request Put#(req_type) the interface through which the outside world
returns (puts) a request
response Get#(resp_type) the interface through which the outside world
retrieves (gets) a response
```
interface Server#(type req_type, type resp_type);
interface Put#(req_type) request;
interface Get#(resp_type) response;
endinterface: Server

ClientServer

AClientcan be connected to aServerand vice versa. Therequest(which is aGetinterface)
of the client will connect toresponse(which is aPutinterface) of theServer. By making the
ClientServertuple an instance of theConnectabletypeclass, you can connect theGetof the client
to thePutof the server, and thePutof the client to theGetof the server.

instance Connectable#(Client#(req_type, resp_type), Server#(req_type, resp_type));
instance Connectable#(Server#(req_type, resp_type), Client#(req_type, resp_type));

ThisTuple2can be redefined to be calledClientServer

typedef Tuple2#(Client#(req_type, resp_type), Server#(req_type,resp_type))
ClientServer#(type req_type, type resp_type);

Example Connecting a bus to a target

interface Bus_Ifc;
interface Server#(RQ, RS) to_initor ;
interface Client#(RQ, RS) to_targ;
endinterface


Reference Guide Bluespec SystemVerilog

typedef Server#(RQ, RS) Target_Ifc;
typedef Client#(RQ, RS) Initiator_Ifc;

module mkSys (Empty);
// Instantiate subsystems
Bus_Ifc bus <- mkBus;
Target_Ifc targ <- mkTarget;
Initiator_Ifc initor <- mkInitiator;

```
// Connect bus and targ (to_targ is a Client ifc, targ is a Server ifc)
Empty x <- mkConnection (bus.to_targ, targ);
```
```
// Connect bus and initiator (to_initor is a Server ifc, initor is a Client ifc)
mkConnection (bus.to_initor, initor);
// Since mkConnection returns an interface of type Empty, it does
// not need to be specified (but may be as above)
```
endmodule: mkSys

Functions

TheClientServerpackage includes functions which return aClientinterface or aServerinterface
from separate request and response interfaces taken as arguments. The argument interfaces must
be able to be converted toGetandPutinterfaces, as indicated by theToGetandToPutprovisos.

```
toGPClient Function that returns aClientinterface from two arguments (request and response inter-
faces). The arguments must be able to be converted toGetandPutinterfaces.
```
```
function Client#(req_type, resp_type)
toGPClient(req_ifc_type req_ifc, resp_ifc_type resp_ifc)
provisos (ToGet#(req_ifc_type, req_type), ToPut#(resp_ifc_type, resp_type));
```
```
toGPServer Function that returns aServerinterface from two arguments (request and response inter-
faces). The arguments must be able to be converted toGetandPutinterfaces.
function Server#(req_type, resp_type)
toGPServer(req_ifc_type req_ifc, resp_ifc_type resp_ifc)
provisos (ToPut#(req_ifc_type, req_type), ToGet#(resp_ifc_type,resp_type));
```
### C.7.4 Memory

Package

import Memory :: * ;

Description


Bluespec SystemVerilog Reference Guide

TheMemorypackage provides the memory structuresMemoryRequestandMemoryResponsewhich
can be used to define a Client/Server memory structure.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

AMemoryRequestis a polymorphic structure of a request containing awritebit, a byte enable
(byteen), theaddressand thedatafor a memory request:

typedef struct {
Bool write;
Bit#(TDiv#(d,8)) byteen;
Bit#(a) address;
Bit#(d) data;
} MemoryRequest#(numeric type a, numeric type d) deriving (Bits, Eq);

TheMemoryResponsecontains the data:

typedef struct {
Bit#(d) data;
} MemoryResponse#(numeric type d) deriving (Bits, Eq);

Interfaces and Methods

The interfacesMemoryServerandMemoryClientare defined from theServerandClientinterfaces
defined inClientServerpackage (Section C.7.3) using theMemoryRequestandMemoryResponse
types.

TheMemoryServeraccepts requests and generates responses, theMemoryClientaccepts responces
and generates requests. There are no assumptions about how many (if any) responses a request
generates.

typedef Server#(MemoryRequest#(a,d), MemoryResponse#(d))
MemoryServer#(numeric type a, numeric type d);

typedef Client#(MemoryRequest#(a,d), MemoryResponse#(d))
MemoryClient#(numeric type a, numeric type d);

Default value instances are defined for bothMemoryRequestandMemoryResponse:

instance DefaultValue#(MemoryRequest#(a,d));
defaultValue = MemoryRequest {
write: False,
byteen: ’1,
address: 0,
data: 0
};
endinstance

instance DefaultValue#(MemoryResponse#(d));
defaultValue = MemoryResponse {
data: 0
};
endinstance


Reference Guide Bluespec SystemVerilog

An instance of theTieOffclass (Section C.8.10) is defined forMemoryClient:

instance TieOff#(MemoryClient#(a, d));

Functions

```
updateDataWithMask Replaces the original data with new data. The data must be divisible an 8-bit
multiple of the mask. The mask indicates which bits to replace.
```
```
function Bit#(d) updateDataWithMask(Bit#(d) origdata
, Bit#(d) newdata
, Bit#(d8) mask);
```
### C.7.5 CGetPut

Package

import CGetPut :: * ;

Description

The interfacesCGetandCPutare similar toGetandPut, but the interconnection of them (via
Connectable) is implemented with a credit-based FIFO. This means that theCGetandCPutinter-
faces have completely registered input and outputs, and furthermore that additional register buffers
can be introduced in the connection path without any ill effect (except an increase in latency, of
course).

In the absence of additional register buffers, the round-trip time for communication between the two
interfaces is 4 clock cycles. Call this numberr. The first argument to the type,n, specifies that
transfers will occur for a fractionn/rof clock cycles (note that the used cycles will not necessarily be
evenly spaced).nalso specifies the depth of the buffer used in the receiving interface (the transmitter
side always has only a single buffer). So (in the absence of additional buffers) usen= 4 to allow
full-bandwidth transmission, at the cost of sufficient registers for quadruple buffering at one end;
usen= 1 for minimal use of registers, at the cost of reducing the bandwidth to one quarter; use
intermediate values to select the optimal trade-off if appropriate.

Interfaces and methods

The interface types are abstract to avoid any improper use of the credit signaling protocol.

```
Interfaces
Interface Name Parameter
name
```
```
Parameter Description Restrictions
```
```
CGet n depth of the buffer used in the re-
ceiving interface
```
```
must be a numeric
type
elementtype type of the element must be inBitsclass
being retrieved by theCGet
CPut n depth of the buffer used in the re-
ceiving interface
```
```
must be a numeric
type
elementtype type of the element must be inBitsclass
being added by theCPut
```
- CGet
    interface CGet#(numeric type n, type element_type);
       ...Abstract...


Bluespec SystemVerilog Reference Guide

- CPut
    interface CPut#(numeric type n, type element_type);
       ...Abstract...
- Connectables
    TheCGetandCPutinterfaces are connectable.
    instance Connectable#(CGet#(n, element_type), CPut#(n, element_type));
    instance Connectable#(CPut#(n, element_type), CGet#(n, element_type));
- CClient and CServer
    The same idea may be extended to clients and servers.
    interface CClient#(type n, type req_type, type resp_type);
    interface CServer#(type n, type req_type, type resp_type);

Modules

```
mkCGetPut Create an n depth FIFO with aCGetinterface on the dequeue side and a
Putinterface on the enqueue side.
```
```
module mkCGetPut(Tuple2#(CGet#(n, element_type),
Put#(element_type)))
provisos (Bits#(element_type));
```
```
mkGetCPut Create an n depth FIFO with aGetinterface on the dequeue side and a
CPutinterface on the enqueue side.
```
```
module mkGetCPut(Tuple2#(Get#(element_type),
CPut#(n, element_type)))
provisos (Bits#(element_type));
```
```
mkClientCServer Create aCServerwith amkCGetPutand amkGetCPut. Provides aCServer
interface and a regularClientinterface.
```
```
module mkClientCServer(
Tuple2#(Client#(req_type, resp_type),
CServer#(n, req_type, resp_type)))
provisos (Bits#(req_type),
Bits#(resp_type));
```
```
mkCClientServer Create aCClientwith amkCGetPutand amkGetCPut. Provides aCClient
interface and a regularServerinterface.
```
```
module mkCClientServer(
Tuple2#(CClient#(n, req_type, resp_type),
Server#(req_type, resp_type)))
provisos (Bits#(req_type),
Bits#(resp_type));
```

Reference Guide Bluespec SystemVerilog

### C.7.6 CommitIfc

Package

import CommitIfc :: * ;

Description

TheCommitIfcpackage defines a Commit/Accept protocol and interfaces to implement a combi-
national connection between two modules without adding an AND gate in the connection. The
protocols implemented by FIFO and Get/Put connections add an AND gate between the modules
being connected. This combinational loop in the connection of the interfaces can cause complications
in FPGA applications and in partitioning for FPGAs. Additionally, some synthesis tools require a
connection level without any gates. By using theCommitIfcprotocol the AND gate is moved out
of the connection and into the connecting modules.

TheCommitIfcpackage defines two interfaces,SendCommitandRecvCommit, which model the op-
posit ends of a FIFO. The protocol does not apply an execution order between thedataoutandack
ordatainandacceptmethods. That is, one can signalacceptbefore the data arrives.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces

TheCommitIfcpackage defines two interfaces:SendCommitandRecvCommit.

TheSendCommitinterface declares two methods: a value methoddataoutand an Action method
ack. No execution order is applied between thedataoutand theack; one can signal that the data
is accepted before the data rrives.

```
SendCommitInterface
Name Type Description
dataout atype The data being sent. There is an implicit RDY indicating the data is valid.
ack Action Signal that data has been accepted.
```
interface SendCommit#(type a_type);
method a_type dataout;
(*always_ready*)
method Action ack;
endinterface

TheRecvCommitinterface declares two methods: an Action method with the data,datain, and a
value methodaccept, returning a Bool indicating if the interface can accept data (comparable to a
notFull).

```
RecvCommitInterface
Name Type Description
datain Action Receives, or enqueues, the valuedin, of typea_type.
accept Bool A boolean indicating if the interface can accept data, comparable to a
notFull.
```

Bluespec SystemVerilog Reference Guide

interface RecvCommit#(type a_type);
(*always_ready*)
method Action datain (a_type din);
(*always_ready*)
method Bool accept ;
endinterface

Connectble Instances

TheCommitIfcpackage defines instances of theConnectabletype class for theSendCommitand
RecvCommitinterfaces, defining how the types can be connected. TheConnectabletype class defines
amkConnectionmodule for each set of pairs.

instance Connectable#(SendCommit#(a_type), RecvCommit#(a_type));

instance Connectable#(RecvCommit#(a_type), SendCommit#(a_type));

FIFOF TheSendCommitandRecvCommitinterfaces can be connected toFIFOFinterfaces.

instance Connectable#(SendCommit#(a_type), FIFOF#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(FIFOF#(a_type), SendCommit#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(RecvCommit#(a_type), FIFOF#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(FIFOF#(a_type), RecvCommit#(a_type))
provisos (Bits#(a_type, size_a));

SyncFIFOIfc TheSendCommitandRecvCommitinterfaces can be connected toSyncFIFOIfcin-
terfaces.

instance Connectable#(SendCommit#(a_type), SyncFIFOIfc#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(SyncFIFOIfc#(a_type), SendCommit#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(RecvCommit#(a_type), SyncFIFOIfc#(a_type))
provisos (Bits#(a_type, size_a));

instance Connectable#(SyncFIFOIfc#(a_type), RecvCommit#(a_type))
provisos (Bits#(a_type, size_a));

Typeclasses

TheCommitIfcpackage defines typeclasses for converting to these interfaces from other interface
types. These must use a module since rules and wires are required.

typeclass ToSendCommit#(type a_type , type b_type)
dependencies (a_type determines b_type);
module mkSendCommit#(a_type x) (SendCommit#(b_type));
endtypeclass


Reference Guide Bluespec SystemVerilog

typeclass ToRecvCommit#(type a_type , type b_type)
dependencies (a_type determines b_type);
module mkRecvCommit#(a_type x) (RecvCommit#(b_type));
endtypeclass

Instances

Instances for theToSendCommitandToRecvCommittype classes are defined to convert to convert
fromFIFO,FIFOF,SyncFIFOIfc,GetandPutinterfaces.

##### FIFO

instance ToSendCommit#(FIFO#(a), a);

Note:ToRecvCommit#(FIFO#(a_type), a_type)is not possible, because it would need to have a
notFullsignal.

FIFOF TheFIFOFinstances assume that the fifo has proper implicit conditions.

instance ToSendCommit#(FIFOF#(a_type), a_type);
module mkSendCommit #(FIFOF#(a) f) (SendCommit#(a));

instance ToRecvCommit#(FIFOF#(a_type), a_type)
provisos(Bits#(a,sa));
module mkRecvCommit #(FIFOF#(a) f) (RecvCommit#(a));

SyncFIFOIfc TheSyncFIFOIfcinstances assume that the fifo has proper implicit conditions.

instance ToSendCommit#(SyncFIFOIfc#(a_type), a_type);
module mkSendCommit #(SyncFIFOIfc#(a) f) (SendCommit#(a));

instance ToRecvCommit#(SyncFIFOIfc#(a_type), a_type)
provisos(Bits#(a,sa));
module mkRecvCommit #(SyncFIFOIfc#(a) f) (RecvCommit#(a));

Get and Put These convert fromGetandPutinterfaces but introduce additional latency:

instance ToSendCommit#(Get#(a_type), a_type)
provisos ( Bits#(a,sa));
module mkSendCommit #(Get#(a_type) g) (SendCommit#(a_type));

instance ToRecvCommit#(Put#(a_type), a_type)
provisos(Bits#(a,sa));
module mkRecvCommit #(Put#(a_type) p) (RecvCommit#(a_type));

These add FIFOs, but maintain loopless behavior:

instance Connectable#(SendCommit#(a_type), Put#(a_type))
provisos (ToRecvCommit#(Put#(a_type), a_type));

instance Connectable#(Put#(a_type), SendCommit#(a_type))


Bluespec SystemVerilog Reference Guide

```
provisos (ToRecvCommit#(Put#(a_type), a_type));
```
instance Connectable#(RecvCommit#(a_type), Get#(a_type))
provisos (ToSendCommit#(Get#(a_type), a_type));

instance Connectable#(Get#(a_type), RecvCommit#(a_type))
provisos (ToSendCommit#(Get#(a_type), a_type));

Client/Server Variations

TheSendCommitandRecvCommitinterfaces can be combined intoClientCommitandServerCommit
type interfaces, similar to theClientandServerinterfaces described in Section C.7.3.

AClientprovides two subinterfaces, aGetand aPutTheClientCommitinterface combines a
SendCommitrequest with aRecvCommitresponse.

interface ClientCommit#(type req, type resp);
interface SendCommit#(req) request;
interface RecvCommit#(resp) response;
endinterface

ThemkClientfromClientCommitmodule takes aClientCommitinterface and provides aClient
interface:

```
mkClientFromClientCommit Provides aClientinterface from aClientCommitinterface.
```
```
module mkClientFromClientCommit#(ClientCommit#(req, resp) c)
(Client#(req,resp))
provisos ( Bits#(resp,_x), Bits#(req,_y));
```
AServerinterface provides aPutrequest with aGetresponse. TheServerCommitinterface com-
bines aRecvCommitrequest with aSendCommitresponse.

interface ServerCommit#(type req, type resp);
interface RecvCommit#(req) request;
interface SendCommit#(resp) response;
endinterface

ClientCommitandServerCommitinterfaces are connectable to each other.

instance Connectable#(ClientCommit#(req,resp), ServerCommit#(req,resp));

instance Connectable#( ServerCommit#(req,resp), ClientCommit#(req,resp));

ClientCommitandServerCommitinterfaces are connectable toClientsandServers.

instance Connectable #(ClientCommit#(req,resp), Server#(req,resp))
provisos ( Bits#(resp,_x), Bits#(req,_y));

instance Connectable #(Server#(req,resp), ClientCommit#(req,resp))
provisos ( Bits#(resp,_x), Bits#(req,_y));


Reference Guide Bluespec SystemVerilog

instance Connectable #(ServerCommit#(req,resp), Client#(req,resp))
provisos ( Bits#(resp,_x), Bits#(req,_y));

instance Connectable #( Client#(req,resp), ServerCommit#(req,resp))
provisos ( Bits#(resp,_x), Bits#(req,_y));

TheSendCommitandRecvCommitcan be defined as instances ofToGetandToPut. These functions
introduce a combinational loop between the Commit interface methods.

instance ToGet#(SendCommit#(a_type, a_type);

instance ToPut#(RecvCommit#(a_type, a_type);

## C.8 Utilities

### C.8.1 LFSR

Package

import LFSR :: * ;

Description

TheLFSRpackage implements Linear Feedback Shift Registers (LFSRs). LFSRs can be used to
obtain reasonable pseudo-random numbers for many purposes (though not good enough for cryp-
tography). Theseedmethod must be called first, to prime the algorithm. Then values may be
read using thevaluemethod, and the algorithm stepped on to the next value by thenextmethod.
When a LFSR is created the start value, or seed, is 1.

Interfaces and Methods

TheLFSRpackage provides an interface,LFSR, which contains three methods;seed,value, and
next. To prime the LFSR theseedmethod is called with the parameterseed_value, of datatype
a_type. The value is read with thevaluemethod. Thenextmethod is used to shift the register
on to the next value.

```
LFSRInterface
Method Arguments
Name Type Description Name Description
seed Action Sets the value of the shift register. a_type datatype of the
seed value
seed_value the initial value
value a_type returns the value of the shift register
next Action signals the shift register to shift to
the next value.
```
interface LFSR #(type a_type);
method Action seed(a_type seed_value);
method a_type value();
method Action next();
endinterface: LFSR


Bluespec SystemVerilog Reference Guide

Modules

The modulemkFeedLFSRcreates a LFSR where the polynomial is specified by the mask used for
feedback.

```
mkFeedLFSR Creates a LFSR where the polynomial is specified by the mask (feed) used
for feedback.
module mkFeedLFSR#( Bit#(n) feed )( LFSR#(Bit#(n)) );
```
For example, the polynominalx^7 +x^3 +x^2 +x+1 is defined by the expressionmkFeedLFSR#(8’b1000_1111)

Using the modulemkFeedLFSR, the following maximal length LFSR’s are defined in this package.

```
Module Name feed Module Definition
```
```
mkLFSR_4 4’h9 module mkLFSR_4 (LFSR#(Bit#(4)));
x^3 + 1
```
```
mkLFSR_8 8’h8E module mkLFSR_8 (LFSR#(Bit#(8)));
```
```
mkLFSR_16 16’h8016 module mkLFSR_16 (LFSR#(Bit#(16)));
```
```
mkLFSR_32 32’h80000057 module mkLFSR_32 (LFSR#(Bit#(32)));
```
For example,

mkLFSR_4 = mkFeedLFSR( 4’h9 );

The modulemkLFSR_4instantiates the interfaceLFSRwith the valueBit#(4)to produce a 4 bit
shift register. The module uses the polynomial defined by the mask 4’h9 (x^3 + 1) and the module
mkFeedLFSR.

ThemkRCounterfunction creates a counter with a LFSR interface. This is useful during debugging
when a non-random sequence is desired. This function can be used in place of the other mkLFSR
module constructors, without changing any method calls or behavior.

```
mkRCounter Creates a counter with a LFSR interface.
```
```
module mkRCounter#( Bit#(n) seed ) ( LFSR#(Bit#(n)) );
```
Example - Random Number Generator

import GetPut::*;
import FIFO::*;
import LFSR::*;

// We want 6-bit random numbers, so we will use the 16-bit version of
// LFSR and take the most significant six bits.

// The interface for the random number generator is parameterized on bit


Reference Guide Bluespec SystemVerilog

// length. It is a "get" interface, defined in the GetPut package.

typedef Get#(Bit#(n)) RandI#(type n);

module mkRn_6(RandI#(6));
// First we instantiate the LFSR module
LFSR#(Bit#(16)) lfsr <- mkLFSR_16 ;

```
// Next comes a FIFO for storing the results until needed
FIFO#(Bit#(6)) fi <- mkFIFO ;
```
```
// A boolean flag for ensuring that we first seed the LFSR module
Reg#(Bool) starting <- mkReg(True) ;
```
```
// This rule fires first, and sends a suitable seed to the module.
rule start (starting);
starting <= False;
lfsr.seed(’h11);
endrule: start
```
```
// After that, the following rule runs as often as it can, retrieving
// results from the LFSR module and enqueing them on the FIFO.
rule run (!starting);
fi.enq(lfsr.value[10:5]);
lfsr.next;
endrule: run
```
// The interface for mkRn_6 is a Get interface. We can produce this from a
// FIFO using the fifoToGet function. We therefore don’t need to define any
// new methods explicitly in this module: we can simply return the produced
// Get interface as the "result" of this module instantiation.
return fifoToGet(fi);
endmodule

### C.8.2 Randomizable

Package

import Randomizable :: * ;

Description

The Randomizable package includes interfaces and modules to generate random values of a given
data type.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Typeclasses

TheRandomizablepackage includes theRandomizabletypeclass.


Bluespec SystemVerilog Reference Guide

typeclass Randomizable#(type t);
module mkRandomizer (Randomize#(t));
endtypeclass

Interfaces and Methods

```
Randomize Interface
Name Type Description
cntrl Interface Control interface provided by the module.
next ActionValue Returns the next value of typea.
```
interface Randomize#(type a);
interface Control cntrl;
method ActionValue#(a) next();
endinterface

```
Control Interface
Name Type Description
init Control Action method to initialize the randomizer.
```
interface Control ;
method Action init();
endinterface

Modules

TheRandomizablepackage includes two modules which return random values of typea. The
difference between the two modules is how the min and max values are determined. The module
mkGenericRandomizeruses the min and max values of the type, while the modulemkConstrainedRandomizer
uses arguments to set the min and max values. The typeamust be in theBoundedclass for both
modules.

```
mkGenericRandomizer This module provides aRandomizeinterface, which will return the next ran-
dom value when thenextmethod is invoked. Theminandmaxvalues are
the values defined by the typeawhich must be in theBoundedclass.
```
```
module mkGenericRandomizer (Randomize#(a))
provisos (Bits#(a, sa), Bounded#(a));
```
```
mkConstrainedRandomizer This module provides aRandomizeinterface, which will give the next random
value when thenextmethod is invoked. When instantiated, theminandmax
values are provided as arguments. Typeamust be in theBoundedclass.
```
```
module mkConstrainedRandomizer#(a min, a max) (Randomize#(a))
provisos (Bits#(a, sa), Bounded#(a));
```

Reference Guide Bluespec SystemVerilog

Example

ThemkTLMRandomizermodule, shown below, uses the Randomize package to generate random values
for TLM packets. ThemkConstrainedRandomizermodule is for fields with specific allowed values or
ranges, while themkGenericRandomizermodule is for field where all values of the type are allowed.

module mkTLMRandomizer#(Maybe#(TLMCommand) m_command) (Randomize#(TLMRequest#(‘TLM_TYPES)))
provisos(Bits#(RequestDescriptor#(‘TLM_TYPES), s0),
Bounded#(RequestDescriptor#(‘TLM_TYPES)),
Bits#(RequestData#(‘TLM_TYPES), s1),
Bounded#(RequestData#(‘TLM_TYPES))
);

```
// Use mkGeneric Randomizer - entire range valid
Randomize#(RequestDescriptor#(‘TLM_TYPES)) descriptor_gen <- mkGenericRandomizer;
Randomize#(Bit#(2)) log_wrap_gen <- mkGenericRandomizer;
Randomize#(RequestData#(‘TLM_TYPES)) data_gen <- mkGenericRandomizer;
```
```
// Use mkConstrainedRandomizer to Avoid UNKNOWN
Randomize#(TLMCommand) command_gen <- mkConstrainedRandomizer(READ, WRITE);
Randomize#(TLMBurstMode) burst_mode_gen <- mkConstrainedRandomizer(INCREMENT, WRAP);
```
```
// Use mkConstrainedRandomizer to set legal sizes between 1 and 16
Randomize#(TLMUInt#(‘TLM_TYPES)) burst_length_gen <- mkConstrainedRandomizer(1,16);
```
### C.8.3 Arbiter

Package

import Arbiter :: * ;

Description

The Arbiter package includes interfaces and modules to implement two different arbiters: a fair
arbiter with changing priorities (round robin) and a sticky arbiter, also round robin, but which gives
the current owner priority.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and Methods

The Arbiter package includes three interfaces: a arbiter client interface, an arbiter request interface
and an arbiter interface which is a vector of client interfaces.

ArbiterClientIFC TheArbiterClient_IFCinterface has two methods: an Action method to
make the request and a Boolean value method to indicate the request was granted. The lock method
is unused in this implementation.


#### C.8.5 GrayCounter Reference Guide Bluespec SystemVerilog

interface ArbiterClient_IFC;
method Action request();
method Action lock();
method Bool grant();
endinterface

ArbiterRequestIFC TheArbiterRequest_IFCinterface has two methods: an Action method
to grant the request and a Boolean value method to indicate there is a request. The lock method is
unused in this implementation.

interface ArbiterRequest_IFC;
method Bool request();
method Bool lock();
method Action grant();
endinterface

TheArbiterClient_IFCinterface and theArbiterRequest_IFCinterface are connectable.

instance Connectable#(ArbiterClient_IFC, ArbiterRequest_IFC);

ArbiterIFC TheArbiter_IFChas a subinterface which is a vector ofArbiterClient_IFCin-
terfaces. The number of items in the vector equals the number of clients.

interface Arbiter_IFC#(type count);
interface Vector#(count, ArbiterClient_IFC) clients;
endinterface

Modules

ThemkArbitermodule is a fair arbiter with changing priorities (round robin). ThemkStickyArbiter
gives the current owner priority - they can hold priority as long as they keep requesting it. The
modules all provide aArbiter_IFCinterface.

```
mkArbiter This module is a fair arbiter with changing priorities (round robin). Iffixedis
True, the current client holds the priority, iffixedisFalse, it moves to the next
client. mkArbiterprovides aArbiter_IFCinterface. Initial priority is given to
client 0.
```
```
module mkArbiter#(Bool fixed) (Arbiter_IFC#(count));
```
```
mkStickyArbiter As long as the client currently with the grant continues to assertrequest, it can
hold the grant. It provides aArbiter_IFCinterface.
```
```
module mkStickyArbiter (Arbiter_IFC#(count));
```

Reference Guide Bluespec SystemVerilog

### C.8.4 Cntrs

Package

import Cntrs :: * ;

Description

The Cntrs package provides interfaces and modules to implement typed and untyped up/down
counters.

TheCountinterface and associatedmkCountmodule provides an up/down counter which allows
atomic simultaneous increment and decrement operations. The scheduled order of operations in a
single cycle is:

```
read SB update SB (incr,decr) SB write
```
If there are simultaneousupdate,incr, anddecroperations, the final result will be:

update_val + incr_val - decr_val

Awritesets the new value towrite_valregardless of the other methods called in the cycle.

TheUCountinterface and associatedmkUCountmodule provide an untyped version of an up/down
counter; that is the counter width can be determined at elaboration time rather than type check
time. The value of the counter is represented by anUInt#(n)where the width of the counter is
determined by themaxValue(0≤maxV alue < 232 ) argument. There are no methods to access the
counter value directly; you can only access the value through the comparison operations.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and Methods

The Cntrs package provides two interfaces;Countwhich is a typed interface andUCountwhich is
an untyped interface.

```
Count Interface Methods
Name Type Description
incr Action Increments the counter byincr_val
decr Action Decrements the counter bydecr_val
update Action Sets the value toupdate_val. Final value will
include increment and decrement values.
_write Action Sets the final value towrite_valregardless of
other operations in the cycle.
_read t Returns the value of the counter.
```
interface Count#(type t);
method Action incr (t incr_val);
method Action decr (t decr_val);
method Action update (t update_val);
method Action _write (t write_val);
method t _read;
endinterface


Bluespec SystemVerilog Reference Guide

```
UCount Interface Methods
Name Type Description
incr Action Increments the counter byincr_val
decr Action Decrements the counter bydecr_val
update Action Sets the value toupdate_val. Final value will
include increment and decrement values.
_write Action Sets the final value towrite_valregardless of
other operations in the cycle.
isEqual Bool Returns true ifvalis equal to the value of the
counter.
isLessThan Bool Returns true ifvalis less than the value of the
counter.
isGreaterThan Bool Returns true ifvalis greater than the value of the
counter.
```
interface UCount;
method Action update (Integer update_val);
method Action _write (Integer write_val);
method Action incr (Integer incr_val);
method Action decr (Integer decr_val);
method Bool isEqual (Integer val);
method Bool isLessThan (Integer val);
method Bool isGreaterThan (Integer val);
endinterface

Modules

```
mkCounter Instantiates a Counter where read precedes update precedes an increment or
decrement precedes a write. TheModArithprovisos limits its use to module
2 arithmetic types:UInt,Int, andBit. Widths of size 0 are supported.
```
```
module mkCount #(t resetVal) (Count#(t))
provisos ( Arith#(t)
,ModArith#(t)
,Bits#(t,st));
```
```
mkUCount Instantiates a counter which can count from 0 tomaxValinclusive. maxVal
must be known at compile time. TheinitValueandmaxValuemust be
Integers.
```
```
module mkUCount#(Integer initValue, Integer maxValue) (UCount);
```
Verilog Modules

mkCounterandmkUCountcorresponds to the following Verilog module, which are found in the
Bluespec Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name Defined in File
```
```
mkCounter vCount Counter.v
mkUCount
```

Reference Guide Bluespec SystemVerilog

C.8.5 GrayCounter

Package

import GrayCounter :: * ;

Description

The GrayCounter package provides an interface and a module to implement a gray-coded counter
with methods for both binary and Gray code. This package is designed for use in theBRAMFIFO
module, Section C.2.4. Since BRAMs have registered address inputs, the binary outputs are not
registered. The counter has two domains, source and destination. Binary and Gray code values are
written in the source domain. Both types of values can be read from the source and the destination
domains.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types

The GrayCounter package uses the typeGray, defined in the Gray package, Section C.8.6. The Gray
package is imported by the GrayCounter package.

Interfaces and Methods

The GrayCounter package includes one interface,GrayCounter.

```
GrayCounter Interface Methods
Name Type Description
incr Action Increments the counter by 1
decr Action Decrements the counter by 1
sWriteBin Action Writes a binary value into the counter in the source
domain.
sReadBin Bit#(n) Returns a binary value from the source domain of
the counter. The output is not registered
sWriteGray Action Writes a Gray code value into the counter in the
source domain.
sReadGray Gray#(n) Returns the Gray code value from the source do-
main of the counter. The output is registered.
dReadBin Bit#(n) Returns the binary value from the destination do-
main of the counter. The output is not registered.
dReadGray Gray#(n) Returns the Gray code value from the destination
domain of the counter. The output is registered.
```
interface GrayCounter#(numeric type n);
method Action incr;
method Action decr;
method Action sWriteBin(Bit#(n) value);
method Bit#(n) sReadBin;
method Action sWriteGray(Gray#(n) value);
method Gray#(n) sReadGray;
method Bit#(n) dReadBin;
method Gray#(n) dReadGray;
endinterface: GrayCounter


Bluespec SystemVerilog Reference Guide

Modules

The modulemkGrayCounterinstantiates a Gray code counter with methods for both binary and
Gray code.

```
mkGrayCounter Instantiates a Gray counter with an initial valueinitval.
```
```
module mkGrayCounter#(Gray#(n) initval,
Clock dClk, Reset dRstN)
(GrayCounter#(n))
provisos(Add#(1, msb, n));
```
#### C.8.6 Gray

Package

import Gray :: * ;

Description

TheGraypackage defines a datatype,Grayand functions for working with the Gray type. This type
is used by theGrayCounterpackage.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

The datatypeGrayis a representation for Gray code values. The basic representation is theGray
structure, which is polymorphic on the size of the value.

```
typedef struct {
Bit#(n) code;
} Gray#(numeric type n) deriving (Bits, Eq);
```
TheGraytype belongs to theLiteralandBoundedtype classes. Each type class definition includes
functions which are then also defined for the data type. The Prelude library definitions (Section B)
describes which functions are defined for each type class.

```
Type Classes used byGray
Bits Eq Literal Arith Ord Bounded Bit Bit Bit
wise Reduction Extend
Gray
```
##### √ √ √ √

Literal TheGraytype is a member of theLiteralclass, which defines an encoding from the
compile-timeIntegertype toGraytype with thefromIntegerandgrayEncodefunctions. The
fromIntegerconverts the value to a bit pattern, and then callsgrayEncode.

instance Literal #( Gray#(n) )
provisos(Add#(1, msb, n));


Reference Guide Bluespec SystemVerilog

Bounded TheGraytype is a member of theBoundedclass, which provides the functionsminBound
andmaxBoundto define the minimum and maximum Gray code values.

- minimum:’b0
- maximum:’b10...0

instance Bounded # ( Gray#(n) )
provisos(Add#(1, msb, n));

Functions

```
grayEncode This function takes a binary value of typeBit#(n)and returns aGray
type with the Gray code value.
```
```
function Gray#(n) grayEncode(Bit#(n) value)
provisos(Add#(1, msb, n));
```
```
grayDecode This function takes a Gray code value of sizenand returns the binary
value.
```
```
function Bit#(n) grayDecode(Gray#(n) value)
provisos(Add#(1, msb, n));
```
```
grayIncrDecr This functions takes a Gray code value and a Boolean,decrement. If
decrementis True, the value returned is one less than the input value.
Ifdecrementis False, the value returned is one greater.
```
```
function Gray#(n) grayIncrDecr(Bool decrement,
Gray#(n) value)
provisos(Add#(1, msb, n));
```
```
grayIncr Takes a Gray code value and returns a Gray code value incremented
by 1.
```
```
function Gray#(n) grayIncr(Gray#(n) value)
provisos(Add#(1, msb, n));
```
```
grayDecr Takes a Gray code value a returns a Gray code value decremented by
1.
function Gray#(n) grayDecr(Gray#(n) value)
provisos(Add#(1, msb, n));
```
#### C.8.7 CompletionBuffer

Package


Bluespec SystemVerilog Reference Guide

import CompletionBuffer :: * ;

Description

ACompletionBufferis like a FIFO except that the order of the elements in the buffer is independent
of the order in which the elements are entered. Each element obtains a token, which reserves a slot
in the buffer. Once the element is ready to be entered into the buffer, the token is used to place the
element in the correct position. When removing elements from the buffer, the elements are delivered
in the order specified by the tokens, not in the order that the elements were written.

Completion Buffers are useful when multiple tasks are running, which may complete at different
times, in any order. By using a completion buffer, the order in which the elements are placed in the
buffer can be controlled, independent of the order in which the data becomes available.

Interface and Methods

The CompletionBuffer interface provides three subinterfaces. Thereserveinterface, aGet, allows
the caller to reserve a slot in the buffer by returning a token holding the identity of the slot. When
data is ready to be placed in the buffer, it is added to the buffer using thecompleteinterface of type
Put. This interface takes a pair of values as its argument - the token identifying its slot, and the
data itself. Finally, using thedraininterface, of typeGet, data may be retrieved from the buffer in
the order in which the tokens were originally allocated. Thus the results of quick tasks might have
to wait in the buffer while a lengthy task ahead of them completes.

The type of the elements to be stored iselement_type. The type of the required size of the buffer
is a numeric typen, which is also the type argument for the type for the tokens issued,CBToken.
This allows the type-checking phase of the synthesis to ensure that the tokens are the appropriate
size for the buffer, and that all the buffer’s internal registers are of the correct sizes as well.

```
CompletionBufferInterface
Name Type Description
```
```
reserve Get Used to reserve a slot in the buffer. Returns a token, CBToken #(n),
identifying the slot in the buffer.
```
```
complete Put Enters the element into the buffer. Takes as arguments the slot in the
buffer,CBToken#(n), and the element to be stored in the buffer.
```
```
drain Get Removes an element from the buffer. The elements are returned in the
order the tokens were allocated.
```
interface CompletionBuffer #(numeric type n, type element_type);
interface Get#(CBToken#(n)) reserve;
interface Put#(Tuple2 #(CBToken#(n), element_type)) complete;
interface Get#(element_type) drain;
endinterface: CompletionBuffer

Datatypes

TheCBTokentype is abstract to avoid misuse.

typedef union tagged { ... } CBToken #(numeric type n) ...;

Modules

ThemkCompletionBuffermodule is used to instantiate a completion buffer. It takes no size argu-
ments, as all that information is already contained in the type of the interface it produces.


Reference Guide Bluespec SystemVerilog

```
mkCompletionBuffer Creates a completion buffer.
```
```
module mkCompletionBuffer(CompletionBuffer#(n, element_type))
provisos (Bits#(element_type, sizea))
```
Example- Using a Completion Buffer in a server farm of multipliers

A server farm is a set of identical servers, which can each perform the same task, together with
a controller. The controller allocates incoming tasks to any server which happens to be available
(free), and sends results back to its caller.

The time needed to complete each task depends on the value of the multiplier argument; there is
therefore no guarantee that results will become available in the order the tasks were started. It is
required, however, that the controller return results to its caller in the order the tasks were received.
The controller accordingly must instantiate a special mechanism for this purpose. The appropriate
mechanism is a Completion Buffer.

import List::*;
import FIFO::*;
import GetPut::*;
import CompletionBuffer::*;

typedef Bit#(16) Tin;
typedef Bit#(32) Tout;

// Multiplier interface
interface Mult_IFC;
method Action start (Tin m1, Tin m2);
method ActionValue#(Tout) result();
endinterface

typedef Tuple2#(Tin,Tin) Args;
typedef 8 BuffSize;
typedef CBToken#(BuffSize) Token;

// This is a farm of multipliers, mkM. The module
// definition for the multipliers mkM is not provided here.
// The interface definition, Mult_IFC, is provided.
module mkFarm#( module#(Mult_IFC) mkM ) ( Mult_IFC );

```
// make the buffer twice the size of the farm
Integer n = div(valueof(BuffSize),2);
```
```
// Declare the array of servers and instantiate them:
Mult_IFC mults[n];
for (Integer i=0; i<n; i=i+1)
begin
Mult_IFC s <- mkM;
mults[i] = s;
end
```
```
FIFO#(Args) infifo <- mkFIFO;
```

Bluespec SystemVerilog Reference Guide

```
// instantiate the Completion Buffer, cbuff, storing values of type Tout
// buffer size is Buffsize, data type of values is Tout
CompletionBuffer#(BuffSize,Tout) cbuff <- mkCompletionBuffer;
```
```
// an array of flags telling which servers are available:
Reg#(Bool) free[n];
// an array of tokens for the jobs in progress on the servers:
Reg#(Token) tokens[n];
// this loop instantiates n free registers and n token registers
// as well as the rules to move data into and out of the server farm
for (Integer i=0; i<n; i=i+1)
begin
// Instantiate the elements of the two arrays:
Reg#(Bool) f <- mkReg(True);
free[i] = f;
Reg#(Token) t <- mkRegU;
tokens[i] = t;
```
```
Mult_IFC s = mults[i];
```
```
// The rules for sending tasks to this particular server, and for
// dealing with returned results:
rule start_server (f); // start only if flag says it’s free
// Get a token
CBToken#(BuffSize) new_t <- cbuff.reserve.get;
```
```
Args a = infifo.first;
Tin a1 = tpl_1(a);
Tin a2 = tpl_2(a);
infifo.deq;
```
```
f <= False;
t <= new_t;
s.start(a1,a2);
endrule
```
```
rule end_server (!f);
Tout x <- s.result;
// Put the result x into the buffer, at the slot t
cbuff.complete.put(tuple2(t,x));
f <= True;
endrule
end
```
```
method Action start (m1, m2);
infifo.enq(tuple2(m1,m2));
endmethod
```
// Remove the element from the buffer, returning the result
// The elements will be returned in the order that the tokens were obtained.
method result = cbuff.drain.get;
endmodule


Reference Guide Bluespec SystemVerilog

#### C.8.8 UniqueWrappers

Package

import UniqueWrappers :: * ;

Description

TheUniqueWrapperspackage takes a piece of combinational logic which is to be shared and puts it
into its own protective shell orwrapperto prevent its duplication. This is used in instances where a
separately synthesized module is not possible. It allows the designer to use a piece of logic at several
places in a design without duplicating it at each site.

There are times where it is desired to use a piece of logic at several places in a design, but it is too
bulky or otherwise expensive to duplicate at each site. Often the right thing to do is to make the
piece of logic into a separately synthesized module – then, if this module is instantiated only once,
it will not be duplicated, and the tool will automatically generate the scheduling and multiplexing
logic to share it among the sites which use its methods. Sometimes, however, this is not convenient.
One reason might be that the logic is to be incorporated into a submodule of the design which is
itself polymorphic – this will probably cause difficulties in observing the constraints necessary for a
module which is to be separately synthesized. And if a module isnotseparately synthesized, the
tool will inline its logic freely wherever it is used, and thus duplication will not be prevented as
desired.

This package covers the case where the logic to be shared is combinational and cannot be put
into a separately synthesized module. It may be thought of as surrounding this combinational
function with a protective shell, aunique wrapper, which will prevent its duplication. The module
mkUniqueWrappertakes a one-argument function as a parameter; both the argument typeaand the
result typebmust be representable as bits, that is, they must both be in theBitstypeclass.

Interfaces

TheUniqueWrapperspackage provides an interface,Wrapper, with one actionvalue method,func,
which takes an argument of typeaand produces a method of typeActionValue#(b). If the module
is instantiated only once, the logic implementing its parameter will be instantiated just once; the
module’s method may, however, be used freely at several places.

Although the function supplied as the parameter is purely combinational and does not change state,
the method is of typeActionValue. This is because actionvalue methods haveenablesignals and
these signals are needed to organize the scheduling and multiplexing between the calling sites.

Variants of the interfaceWrapperare also provided for handling functions of two or three arguments;
the interfaces have one and two extra parameters respectively. In each case the result type is the
final parameter, following however many argument type parameters are required.

```
Wrapper Interfaces
```
```
Wrapper This interface has one actionvalue method,func, which takes an argument of type
a_typeand produces an actionvalue of typeActionValue#(b_type).
interface Wrapper#(type a_type, type b_type);
method ActionValue#(b_type) func (a_type x);
```
```
Wrapper2 Similar to theWrapperinterface, but it takes two arguments.
```
```
interface Wrapper2#(type a1_type, type a2_type, type b_type);
method ActionValue#(b_type) func (a1_type x, a2_type y);
```

Bluespec SystemVerilog Reference Guide

```
Wrapper3 Similar to theWrapperinterface, but it takes three arguments.
```
```
interface Wrapper3#(type a1_type, type a2_type, type a3_type,
type b_type);
method ActionValue#(b_type) func (a1_type x, a2_type y, a3_type z);
```
Modules

The interfacesWrapper,Wrapper2, andWrapper3are provided by the modulesmkUniqueWrapper,
mkUniqueWrapper2, andmkUniqueWrapper3. These modules vary only in the number of aguments
in the parameter function.

If a function has more than three arguments, it can always be rewritten or wrapped as one which
takes the arguments as a single tuple; thus the one-argument versionmkUniqueWrappercan be used
with this function.

```
mkUniqueWrapper
```
```
Takes a function,func, with a single parameterxand provides the interfaceWrapper.
module mkUniqueWrapper#(function b_type func(a_type x))
(Wrapper#(a_type, b_type))
provisos (Bits#(a_type, sizea), Bits#(b_type, sizeb));
```
```
mkUniqueWrapper2
```
```
Takes a function,func, with a two parameters,xandy, and provides the interface
Wrapper2.
module mkUniqueWrapper2#(function b_type func(a1_type x, a2_type y))
(Wrapper2#(a1_type, a2_type, b_type))
provisos (Bits#(a1_type, sizea1), Bits#(a2_type, sizea2),
Bits#(b_type, sizeb));
```
```
mkUniqueWrapper3
```
```
Takes a function,func, with a three parameters,x,y, andz, and provides the interface
Wrapper3.
module mkUniqueWrapper3#(function b_type
func(a1_type x, a2_type y, a3_type z))
(Wrapper3#(a1_type, a2_type, a3_type, b_type))
provisos (Bits#(a1_type, sizea1), Bits#(a2_type, sizea2),
Bits#(a3_type, sizea3), Bits#(b_type, sizeb));
```
Example: Complex Multiplication

// This module defines a single hardware multiplier, which is then
// used by multiple method calls to implement complex number
// multiplication (a + bi)(c + di)


Reference Guide Bluespec SystemVerilog

typedef Int#(18) CFP;

module mkComplexMult1Fifo( ArithOpGP2#(CFP) ) ;
FIFO#(ComplexP#(CFP)) infifo1 <- mkFIFO;
FIFO#(ComplexP#(CFP)) infifo2 <- mkFIFO;
let arg1 = infifo1.first ;
let arg2 = infifo2.first ;

```
FIFO#(ComplexP#(CFP)) outfifo <- mkFIFO;
```
```
Reg#(CFP) rr <- mkReg(0) ;
Reg#(CFP) ii <- mkReg(0) ;
Reg#(CFP) ri <- mkReg(0) ;
Reg#(CFP) ir <- mkReg(0) ;
```
```
// Declare and instantiate an interface that takes 2 arguments, multiplies them
// and returns the result. It is a Wrapper2 because there are 2 arguments.
Wrapper2#(CFP,CFP, CFP) smult <- mkUniqueWrapper2( \* ) ;
```
```
// Define a sequence of actions
// Since smult is a UnqiueWrapper the method called is smult.func
Stmt multSeq =
seq
action
let mr <- smult.func( arg1.rel, arg2.rel ) ;
rr <= mr ;
endaction
action
let mr <- smult.func( arg1.img, arg2.img ) ;
ii <= mr ;
endaction
action
// Do the first add in this step
let mr <- smult.func( arg1.img, arg2.rel ) ;
ir <= mr ;
rr <= rr - ii ;
endaction
action
let mr <- smult.func( arg1.rel, arg2.img );
ri <= mr ;
// We are done with the inputs so deq the in fifos
infifo1.deq ;
infifo2.deq ;
endaction
action
let ii2 = ri + ir ;
let res = Complex{ rel: rr , img: ii2 } ;
outfifo.enq( res ) ;
endaction
endseq;
```
```
// Now convert the sequence into a FSM ;
// Bluespec can assign the state variables, and pick up implict
// conditions of the actions
```

Bluespec SystemVerilog Reference Guide

FSM multfsm <- mkFSM(multSeq);
rule startFSM;
multfsm.start;
endrule
endmodule

#### C.8.9 DefaultValue

Package

import DefaultValue :: * ;

Description

This package defines a type class ofDefaultValueand instances of the type class for many commonly
used datatypes. Users can create their own default value instances for other types. This type class
is particularly useful for defining default values for user-defined structures.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Typeclasses

typeclass DefaultValue #( type t );
t defaultValue ;
endtypeclass

The following instances are defined in theDefaultValuepackage. You can define your own instances
for user-defined structures and other types.

```
DefaultValueInstances
Literal#(t) Any typetin theLiteralclass can have a default value which is de-
fined here as 0. The types in theLiteralclass includeBit#(n),Int#(n),
UInt#(n),Real,Integer,FixedPoint, andComplex.
```
```
instance DefaultValue # (t)
provisos (Literal#(t));
defaultValue = fromInteger (0);
```
```
Bool The default value for aBoolis defined asFalse.
instance DefaultValue #( Bool );
defaultValue = False ;
```
```
void The default value for a void is defined as?.
```
```
instance DefaultValue #(void);
defaultValue = ?;
```

Reference Guide Bluespec SystemVerilog

```
Maybe The default value for aMaybeis defined astagged Invalid.
```
```
instance DefaultValue #( Maybe#(t) );
defaultValue = tagged Invalid ;
```
The default value for aTupleis composed of the default values of each member type. Instances are
defined forTuple2throughTuple8.

```
Tuple2#(a,b) The default value of aTuple2is the default value of elementaand the
default value of elementb.
```
```
instance DefaultValue #( Tuple2#(a,b) )
provisos (DefaultValue#(a)
,DefaultValue#(b) );
defaultValue = tuple2 (defaultValue, defaultValue );
```
```
Vector The default value for a Vector replicates the element’s default value type
for each element.
```
```
instance DefaultValue #( Vector#(n,t) )
provisos (DefaultValue#(t));
defaultValue = replicate (defaultValue) ;
```
Examples

Example 1: Specifying the initial or reset values for a structure.

```
Reg#(Int#(17)) rint <- mkReg#(defaultValue); // initial value 0
Reg#(Tuple2#(Bool,UInt#(5))) tbui <- mkReg#(defaultValue); // value is(False,0)
Reg#(Vector#(n,Bool) vbool <- mkReg#(defaultValue); // initial value all False
```
Example 2: Using default values to replace the unsafe use of unpack.

import DefaultValue :: *;

typedef struct {
UInt#(4) size;
UInt#(3) depth ;
} MyStruct
deriving (Bits, Eq);

instance DefaultValue #( MyStruct );
defaultValue = MyStruct { size : 0,
depth : 1 };
endinstance

then you can use:

```
Reg#(MyStruct) mstr <- mkReg(defaultValue);
```

Bluespec SystemVerilog Reference Guide

instead of:

```
Reg#(MyStruct) mybad <- mkReg(unpack(0)); // Bad use of unpack
```
Example 3: Module instantiation which requires a large structure as an argument.

```
ModParam modParams = defaultValue ; // generate default value
modParams.field1 = 5 ; // override some default values
modParams.field2 = 1.4 ;
ModIfc <- mkMod (modArgs) ; // construct the module
```
#### C.8.10 TieOff

Package

import TieOff :: * ;

Description

This package provides a typeclassTieOff#(t)which may be userful to provide default enable meth-
ods of some interfacet, some of which must bealways_enabledor require some action.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Typeclasses

typeclass TieOff #(type t);
module mkTieOff#(t ifc) (Empty);
endtypeclass

Example: Defining aTieOfffor aGetinterface

This is a sink module which pulls data from theGetinterface and displays the data.

instance TieOff #(Get #(t) )
provisos (Bits#(t,st),
FShow#(t) );
module mkTieOff ( Get#(t) ifc, Empty inf);
rule getSink (True);
t val <- ifc.get;
$display( "Get tieoff %m", fshow(val) );
endrule
endmodule
endinstance


Reference Guide Bluespec SystemVerilog

#### C.8.11 Assert

Package

import Assert :: *;

Description

TheAssertpackage contains definitions to test assertions in the code. Thecheck-assertflag must
be set during compilation. By default the flag is set toFalseand assertions are ignored. The flag,
when set, instructs the compiler to abort compilation if an assertion fails.

Functions

```
staticAssert Compile time assertion. Can be used anywhere a compile-time statement
is valid.
```
```
module staticAssert(Bool b, String s);
```
```
dynamicAssert Run time assertion. Can be used anywhere an Action is valid, and is
tested whenever it is executed.
```
```
function Action dynamicAssert(Bool b, String s);
```
```
continuousAssert Continuous run-time assertion (expected to be True on each clock). Can
be used anywhere a module instantiation is valid.
```
```
function Action continuousAssert(Bool b, String s);
```
Examples using Assertions:

import Assert:: *;
module mkAssert_Example ();
// A static assert is checked at compile-time
// This code checks that the indices are within range
for (Integer i=0; i<length(cs); i=i+1)
begin
Integer new_index = (cs[i]).index;
staticAssert(new_index < valueOf(n),
strConcat("Assertion index out of range: ", integerToString(new_index)));
end

```
rule always_fire (True);
counter <= counter + 1;
endrule
// A continuous assert is checked on each clock cycle
continuousAssert (!fail, "Failure: Fail becomes True");
```
```
// A dynamic assert is checked each time the rule is executed
```

Bluespec SystemVerilog Reference Guide

rule test_assertion (True);
dynamicAssert (!fail, "Failure: Fail becomes True");
endrule
endmodule: mkAssert_Example

#### C.8.12 Probe

Package

import Probe :: * ;

Description

AProbeis a primitive used to ensure that a signal of interest is not optimized away by the compiler
and that it is given a known name. In terms of BSV syntax, theProbeprimitive it used just like
a register except that only a write method exists. Since reads are not possible, the use of aProbe
has no effect on scheduling. In the generated Verilog, the associated signal will be named just like
the port of any Verilog module, in this case<instance_name>$PROBE. No actualProbeinstance will
be created however. The only side effects of a BSVProbeinstantiation relate to the naming and
retention of the associated signal in the generated Verilog.

Interfaces

interface Probe #(type a_type);
method Action _write(a_type x1);
endinterface: Probe

Modules

The modulemkProbeis used to instantiate aProbe.

```
mkProbe Instantiates aProbe
```
```
module mkProbe(Probe#(a_type))
provisos (Bits#(a_type, sizea));
```
Example - Creating and writing to registers and probes

import FIFO::*;
import ClientServer::*;
import GetPut::*;
import Probe::*;

typedef Bit#(32) LuRequest;
typedef Bit#(32) LuResponse;

module mkMesaHwLpm(ILpm);
// Create registers for requestB32 and responseB32
Reg#(LuRequest) requestB32 <- mkRegU();
Reg#(LuResponse) responseB32 <- mkRegU();

```
// Create a probe responseB32_probe
Probe#(LuResponse) responseB32_probe <- mkProbe();
....
// Define the interfaces:
```

Reference Guide Bluespec SystemVerilog

```
interface Get response;
method get() ;
actionvalue
let resp <- completionBuffer.drain.get();
// record response for debugging purposes:
let {r,t} = resp;
responseB32 <= r; // a write to a register
responseB32_probe <= r; // a write to a probe
```
```
// count responses in status register
return(resp);
endactionvalue
endmethod: get
endinterface: response
```
endmodule

#### C.8.13 Reserved

Package

import Reserved :: * ;

Description

TheReservedpackage defines three abstract data types which only have the purpose of taking up
space. They are useful when defining astructwhere you need to enforce a certain layout and want
to use the type checker to enforce that the value is not accidently used. One can enforce a layout
unsafely withBit#(n), butReserved#(n)gives safety. A value of typeReserved#(n)takes up
exactlynbits.

```
typedef···abstract···Reserved#(type n);
```
Types and Type classes

There are three types defined in theReservedpackage:Reserved,ReservedZero, andReservedOne.
TheReservedtype is an abstract data type which takes up exactlynbits and always returns an
unspecified value. TheReservedZeroandReservedOnedata types are equivalent to theReserved
type except thatReservedZeroalways returns’0andReservedOnealways returns’1.

```
Type Classes used byReserved
Bits Eq Literal Arith Ord Bounded Bit Bit Bit
wise Reduction Extend
Reserved
```
##### √ √ √ √

```
ReservedZero
```
##### √ √ √ √

```
ReservedOne
```
##### √ √ √ √

- BitsThe only purpose of these types is to allow the value to exist in hardware (at port
    boundaries and in states). The user should have no reason to usepack/unpackdirectly.

```
ConvertingReservedto or fromBitsreturns a don’t care (?).
```
```
ConvertingReservedZeroto or fromBitsreturns a’0.
```

Bluespec SystemVerilog Reference Guide

```
ConvertingReservedOneto or fromBitsreturns a’1.
```
- EqandOrd
    Any twoReserved,ReservedZero, orReservedOnevalues are considered to be equal.
- Bounded
    The upper and lower bound return don’t care (?),’1or’0values depending on the type.

Example: Structure with a 8 bits reserved.

typedef struct {
Bit#(8) header; // Frame.header
Vector#(2, Bit#(8)) payload; // Frame.payload
Reserved#(8) dummy; // Can’t access 8 bits reserved
Bit#(8) trailer; // Frame.trailer
} Frame;

```
header payload0 payload1 dummy trailer
8 8 8 8 8
```
#### C.8.14 TriState

Package

import TriState :: * ;

Description

TheTriStatepackage implements a tri-state buffer, as shown in Figure 5. Depending on the value
of theoutput_enable,inoutcan be an input or an output.

```
Figure 5: TriState Buffer
```
The buffer has two inputs, aninputof typevalue_typeand a Booleanoutput_enablewhich
determines the direction ofinout. Ifoutput_enableis True, the signal is coming in frominput
and out throughinoutandoutput. Ifoutput_enableis False, then a value can be driven in from
inout, and theoutputvalue will be the value ofinout. The behavior is described in the tables
below.

```
outputenable = 0
output = inout
Inputs
input inout output
0 0 0
0 1 1
1 0 0
1 1 1
```

Reference Guide Bluespec SystemVerilog

```
outputenable = 1
output = in
inout = in
Outputs
input inout output
0 0 0
1 1 1
```
This module is not supported in Bluesim.

Interfaces and Methods

TheTriStateinterface is composed of anInoutinterface and a_readmethod. The_readmethod
is similar to the_readmethod of a register in that to read the method you reference the interface
in an expression.

```
TriState Interface
Name Type Description
io Inout#(value_type) Inout subinterface providing a value of type
value_type
_read value_type Returns the value ofoutput
```
(* always_ready, always_enabled *)
interface TriState#(type value_type);
interface Inout#(value_type) io;
method value_type _read;
endinterface: TriState

Modules and Functions

TheTriStatepackage provides a module constructor function,mkTriState, which provides the
TriStateinterface. The interface includes anInoutsubinterface and the value ofoutput.

```
mkTriState Creates a module which provides theTriStateinterface.
```
```
module mkTriState#(Bool output_enable, value_type input)
(TriState#(value_type))
provisos(Bits#(value_type, size_value));
```
Verilog Modules

TheTriStatemodule is implemented by the Verilog moduleTriState.vwhich can be found in the
Bluespec Verilog library,$BLUESPECDIR/Verilog/.

#### C.8.15 ZBus

Package

import ZBus :: * ;


Bluespec SystemVerilog Reference Guide

Description

BSV provides theZBuslibrary to allow users to implement and use tri-state buses. Since BSV does
not support high-impedance or undefined values internally, the library encapsulates the tri-state bus
implementation in a module that can only be accessed through predefined interfaces which do not
allow direct access to internal signals (which could potentially have high-impedance or undefined
values).

The Verilog implementation of the tri-state module includes a number of primitive submodules
that are implemented using Verilog tri-state wires. The BSV representation of the bus, however,
only models the values of the bus at the associated interfaces and thus the need to represent high-
impedance or undefined values in BSV is avoided.

A ZBus consists of a series of clients hanging off of a bus. The combination of the client and the
bus is provided by theZBusDualIFCinterface which consists of 2 subinterfaces, the client and the
bus. The client subinterface is provided by theZBusClientIFCinterface. The bus subinterface is
provided by theZBusBusIFCinterface. The user never needs to manipulate the bus side, this is all
done internally. The user builds the bus out ofZBusDualIFCs and then drives values onto the bus
and reads values from the bus using theZBusClientIFC.

Interfaces and Methods

There are three interfaces are defined in this package;ZBusDualIFC,ZBusClientIFC, andZBusBusIFC.

TheZBusDualIFCinterface provides two subinterfaces; aZBusBusIFCand aZBusClientIFC. For a
given bus, oneZBusDualIFCinterface is associated with each bus client.

```
ZBusDualIFC
Name Type Description
busIFC ZBusBusIFC#() The subinterface providing the bus side of the
ZBus.
clientIFC ZBusClientIFC#(t) The subinterface providing the client side to the
ZBus.
```
interface ZBusDualIFC #(type value_type) ;
interface ZBusBusIFC#(value_type) busIFC;
interface ZBusClientIFC#(value_type) clientIFC;
endinterface

TheZBusClientIFCallows a BSV module to connect to the tri-state bus. Thedrivemethod is
used to drive a value onto the bus. Theget()andfromBusValid()methods allow each bus client
to access the current value on the bus. If the bus is in an invalid state (i.e. has a high-impedance
value or an undefined value because it is being driven by more than one client simultaneously), then
theget()method will return 0 and thefromBusValid()method will returnFalse. In all other
cases, thefromBusValid()method will returnTrueand theget()method will return the current
value of the bus.

```
ZBusClientIFC
Method Argument
Name Type Description Name Description
drive Action Drives a current value on
to the bus
```
```
value The value being put on
the bus, datatype of
value_type.
get value_type Returns the current
value on the bus.
fromBusValid Bool ReturnsFalseif the bus
has a high-impedance
value or is undefined.
```

Reference Guide Bluespec SystemVerilog

interface ZBusClientIFC #(type value_type) ;
method Action drive(value_type value);
method value_type get();
method Bool fromBusValid();
endinterface

TheZBusBusIFCinterface connects to the bus structure itself using tri-state values. This interface
is never accessed directly by the user.

interface ZBusBusIFC #(type value_type) ;
method Action fromBusSample(ZBit#(value_type) value, Bool isValid);
method ZBit#(t) toBusValue();
method Bool toBusCtl();
endinterface

Modules and Functions

The library provides a module constructor function,mkZBusBuffer, which allows the user to create
a module which provides theZBusDualIFCinterface. This module provides the functionality of a
tri-state buffer.

```
mkZBusBuffer Creates a module which provides theZBusDualIFCinterface.
```
```
module mkZBusBuffer (ZBusDualIFC #(value_type))
provisos (Eq#(value_type), Bits#(value_type, size_value));
```
ThemkZBusmodule constructor function takes a list ofZBusBusIFCinterfaces as arguments and
creates a module which ties them all together in a bus.

```
mkZBus Ties a list ofZBusBusIFCinterfaces together in a bus.
```
```
module mkZBus#(List#(ZBusBusIFC#(value_type)) ifc_list)(Empty)
provisos (Eq#(value_type), Bits#(value_type, size_value));
```
Examples - ZBus

Creating a tri-state buffer for a 32 bit signal. The interface is namedbuffer_0.

```
ZBusDualIFC#(Bit#(32)) buffer_0();
mkZBusBuffer inst_buffer_0(buffer_0);
```
Drive a value of 12 onto the associated bus.

```
buffer_0.clientIFC.drive(12);
```
The following code fragment demonstrates the use of the modulemkZBus.

```
ZBusDualIFC#(Bit#(32)) buffer_0();
mkZBusBuffer inst_buffer_0(buffer_0);
```
```
ZBusDualIFC#(Bit#(32)) buffer_1();
mkZBusBuffer inst_buffer_1(buffer_1);
```

Bluespec SystemVerilog Reference Guide

```
ZBusDualIFC#(Bit#(32)) buffer_2();
mkZBusBuffer inst_buffer_2(buffer_2);
```
```
List#(ZBusIFC#(Bit#(32))) ifc_list;
```
```
bus_ifc_list = cons(buffer_0.busIFC,
cons(buffer_1.busIFC,
cons(buffer_2.busIFC,
nil)));
```
```
Empty bus_ifc();
mkZBus#(bus_ifc_list) inst_bus(bus_ifc);
```
#### C.8.16 CRC

Package

import CRC :: * ;

Description

CRC’s are designed to protect against common types of errors on communication channels. The
CRCpackage defines modules to calculate a check value for each 8-bit block of data, which can
then be verified to determine if data was transmitted and/or received correctly. There are many
commonly used and standardized CRC algorithms. TheCRCpackage provides both a generalized
CRC module as well as module implementations for the CRC-CCITT, CRC-16-ANSI, and CRC-32
(IEEE 802.3) standards. The size of the CRC polynomial is polymorphic and the data size is a byte
(Bit#(8)), which is relevant for many applications. The generalized module uses five arguments
to define the CRC algorithm: the CRC polynomial, the initial CRC value, a fixed bit pattern to
Xor with the remainder, a boolean indicating whether to reverse the data bit order and a boolean
indicating whether to reverse the result bit order. By specifying these arguments, you can implement
many CRC algorithms. This package provides modules for three specific algorithms by defining the
arguments for those algorithms.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Interfaces and Methods

The CRC modules provide theCRCinterface. Theaddmethod is used to calculate the check value
on thedataargument. In this package, the argument is always aBit#(8).

```
CRCInterface
Method Arguments
Name Type Description Name Description
add Action Update the CRC Bit#(8) data 8-bit data block
clear Action Reset to the initial value
result Bit#(n) Returns the current value of the check value
complete ActionValue(Bit#(n)) Return the result and reset
```

Reference Guide Bluespec SystemVerilog

interface CRC#(numeric type n);
method Action add(Bit#(8) data);
method Action clear();
method Bit#(n) result();
method ActionValue#(Bit#(n)) complete();
endinterface

Modules

The implementation of the generalized CRC module takes the following five arguments:

- Bit#(n)polynomial: the crc operation polynomial, for examplex^16 +x^12 +x^5 + 1 is written
    as ‘h 1021
- Bit#(n)initval: the initial CRC value
- Bit#(n)finalXor: the result is xor’d with this value if desired
- BoolreflectData: if True, reverse the data bit order
- BoolreflectRemainder: if True, reverse the result bit order

```
mkCRC The generalized CRC module. The provisos enforce the requirement that poly-
nomial and initial value must be at least 8 bits.
```
```
module mkCRC#( Bit#(n) polynomial
, Bit#(n) initval
, Bit#(n) finalXor
, Bool reflectData
, Bool reflectRemainder
)(CRC#(n))
provisos( Add#(8, n8, n) );
```
```
CRC Arguments for Common Standards
Name polynomial initval finalXor reflectData reflectRemainder
CRC-CCITT ’h1021 ’hFFFF ’h0000 False False
CRC-16-ANSI ’h8005 ’h0000 ’h0000 True True
CRC-32 (IEEE 802.3) ’h04C11DB7 ’hFFFFFFFF ’hFFFFFFFF True True
```
```
mkCRC_CCIT Implements the 16-bit CRC-CCITT standard.
(x^16 +x^15 +x^2 + 1).
```
```
module mkCRC_CCITT(CRC#(16));
```
```
mkCRC16 Implementation of the 16-bit CRC-16-ANSI standard.
(x^16 +x^15 +x^2 + 1).
```
```
module mkCRC16(CRC#(16));
```

Bluespec SystemVerilog Reference Guide

```
mkCRC32 Implementation of the 32-bit CRC-32 (IEEE 802.3) standard.
(x^32 +x^26 +x^23 +x^22 +x^16 +x^12 +x^11 +x^10 +x^8 +x^7 +x^5 +x^4 +x^2 +x^1 + 1)
```
```
module mkCRC32(CRC#(32));
```
```
reflect Thereflectfunction reverses the data bits if the value ofreflectDataisTrue.
```
```
function Bit#(a) reflect(Bool doIt, Bit#(a) data);
return (doIt)? reverseBits(data) : data;
endfunction
```
#### C.8.17 OVLAssertions

Package

import OVLAssertions :: * ;

Description

The OVLAssertions package provides the BSV interfaces and wrapper modules necessary to al-
low BSV designs to include assertion checkers from the Open Verification Library (OVL). The
OVL includes a set of assertion checkers that verify specific properties of a design. For more
details on the complete OVL, refer to the Accellera Standard OVL Library Reference Manual
(http://www.accellera.org).

Interfaces and Methods

The following interfaces are defined for use with the assertion modules. Each interface has one or
moreActionmethods. Each method takes a single argument which is either aBoolor polymorphic.

AssertTestIFC Used for assertions that check a test expression on every clock cycle.

```
AssertTest_IFC
Method Argument
Name Type Name Type Description
test Action test_value a_type Expression to be checked.
```
interface AssertTest_IFC #(type a_type);
method Action test(a_type test_value);
endinterface

AssertSampleTestIFC Used for assertions that check a test expression on every clock cycle only
if the sample, indicated by the boolean valuesample_testis asserted.

```
AssertSampleTest_IFC
Method Argument
Name Type Name Type Description
sample Action sample_test Bool Assertion only checked ifsample_testis
asserted.
test Action test_value a_type Expression to be checked.
```

Reference Guide Bluespec SystemVerilog

interface AssertSampleTest_IFC #(type a_type);
method Action sample(Bool sample_test);
method Action test(a_type test_value);
endinterface

AssertStartTestIFC Used for assertions that check a test expression only subsequent to a
startevent, specified by the Boolean valuestart_test.

```
AssertStartTest_IFC
Method Argument
Name Type Name Type Description
start Action start_test Bool Assertion only checked after start is as-
serted.
test Action test_value a_type Expression to be checked.
```
interface AssertStartTest_IFC #(type a_type);
method Action start(Bool start_test);
method Action test(a_type test_value);
endinterface

AssertStartStopTestIFC Used to check a test expression between a startevent and a stopevent.

```
AssertStartStopTest_IFC
Method Argument
Name Type Name Type Description
start Action start_test Bool Assertion only checked after start is as-
serted.
stop Action stop_test Bool Assertion only checked until the stop is
asserted.
test Action test_value a_type Expression to be checked.
```
interface AssertStartStopTest_IFC #(type a_type);
method Action start(Bool start_test);
method Action stop(Bool stop_test);
method Action test(a_type test_value);
endinterface

AssertTransitionTestIFC Used to check a test expression that has a specified start state and
next state, i.e. a transition.

```
AssertTransitionTest_IFC
Method Argument
Name Type Name Type Description
test Action test_value a_type Expression that should transition to the
next_value.
start Action start_test a_type Expression that indicates the start state
for the assertion check. If the value
of start_test equals the value of
test_value, the check is performed.
next Action next_value a_type Expression that indicates the only valid
next state for the assertion check.
```

Bluespec SystemVerilog Reference Guide

interface AssertTransitionTest_IFC #(type a_type);
method Action test(a_type test_value);
method Action start(a_type start_value);
method Action next(a_type next_value);
endinterface

AssertQuiescentTestIFC Used to check that a test expression is equivalent to the specified
expression when the sample state is asserted.

```
AssertQuiescentTest_IFC
Method Argument
Name Type Name Type Description
sample Action sampe_test Bool Expression which initiates the quiescent
assertion check when it transistions to
true.
state Action state_value a_type Expression that should have the same
value ascheck_value
check Action check_value a_type Expressionstate_valueis compared to.
```
interface AssertQuiescentTest_IFC #(type a_type);
method Action sample(Bool sample_test);
method Action state(a_type state_value);
method Action check(a_type check_value);
endinterface

AssertFifoTestIFC Used with assertions checking a FIFO structure.

```
AssertFifoTest_IFC
Method Argument
Name Type Name Type Description
push Action push_value a_type Expression which indicates the number of
push operations that will occur during the
current cycle.
pop Action pop_value a_type Expression which indicates the number of
pop operations that will occur during the
current cycle.
```
interface AssertFifoTest_IFC #(type a_type, type b_type);
method Action push(a_type push_value);
method Action pop(b_type pop_value);
endinterface

Datatypes

The parametersseverity_level,property_type,msg, andcoverage_levelare common to all
assertion checkers.


Reference Guide Bluespec SystemVerilog

```
Common Parameters for all Assertion Checkers
Parameter Valid Values
* indicates default value
severity_level OVL_FATAL
*OVL_ERROR
OVL_WARNING
OVL_Info
property_type *OVL_ASSERT
OVL_ASSUME
OVL_IGNORE
msg *VIOLATION
coverage_level OVL_COVER_NONE
*OVL_COVER_ALL
OVL_COVER_SANITY
OVL_COVER_BASIC
OVL_COVER_CORNER
OVL_COVER_STATISTIC
```
Each assertion checker may also use some subset of the following parameters.

```
Other Parameters for Assertion Checkers
Parameter Valid Values
action_on_new_start OVL_IGNORE_NEW_START
OVL_RESET_ON_NEW_START
OVL_ERROR_ON_NEW_START
edge_type OVL_NOEDGE
OVL_POSEDGE
OVL_NEGEDGE
OVL_ANYEDGE
necessary_condition OVL_TRIGGER_ON_MOST_PIPE
OVL_TRIGGER_ON_FIRST_PIPE
OVL_TRIGGER_ON_FIRST_NOPIPE
inactive OVL_ALL_ZEROS
OVL_ALL_ONES
OVL_ONE_COLD
```
```
Other Parameters for Assertion Checkers
Parameter Valid Values
num_cks Int#(32)
min_cks Int#(32)
max_cks Int#(32)
min_ack_cycle Int#(32)
max_ack_cycle Int#(32)
max_ack_length Int#(32)
req_drop Int#(32)
deassert_count Int#(32)
depth Int#(32)
value a_type
min a_type
max a_type
check_overlapping Bool
check_missing_start Bool
simultaneous_push_pop Bool
```

Bluespec SystemVerilog Reference Guide

Setting Assertion Parameters

Each assertion checker module has a set of associated parameter values that can be customized for
each module instantiation. The values for these parameters are passed to each checker module in
the form of a single struct argument of typeOVLDefaults#(a)A typical use scenario is illustrated
below:

let defaults = mkOVLDefaults;

defaults.min_clks = 2;
defaults.max_clks = 3;

AssertTest_IFC#(Bool) assertWid <- bsv_assert_width(defaults);

The defaults struct (created bymkOVLDefaults) includes one field for each possible parameter.
Initially each field includes the associated default value. By editing fields of the struct, individual
parameter values can be modified as needed to be non-default values. The modifieddefaultsstruct
is then provided as a module argument during instantiation.

Modules

Each module in this package corresponds to an assertion checker from the Open Verification Library
(OVL). The BSV name for each module is the same as the OVL name withbsv_appended to the
beginning of the name.

```
Module bsv_assert_always
Description Concurrent assertion that the value of the expression is alwaysTrue.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_always#(OVLDefaults#(Bool) defaults)
(AssertTest_IFC#(Bool));
```
```
Module bsv_assert_always_on_edge
Description Checks that the test expression evaluatesTruewhenever the sample
method is asserted.
Interface Used AssertSampleTest_IFC
Parameters common assertion parameters
edge_type(default value =OVL_NOEDGE)
Module Declaration
module bsv_assert_always_on_edge#(OVLDefaults#(Bool)
defaults)(AssertSampleTest_IFC#(Bool));
```

Reference Guide Bluespec SystemVerilog

```
Module bsv_assert_change
Description Checks that once the start method is asserted, the expression will change
value withinnum_ckscycles.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
action_on_new_start(default value =OVL_IGNORE_NEW_START)
num_cks(default value = 1)
Module Declaration
module bsv_assert_change#(OVLDefaults#(a_type) defaults)
(AssertStartTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_cycle_sequence
Description Ensures that if a specified necessary condition occurs,it is followed by a
specified sequence of events.
Interface Used AssertTest_IFC
Parameters common assertion parameters
necessary_condition(default value =OVL_TRIGGER_ON_MOST_PIPE)
Module Declaration
module bsv_assert_cycle_sequence#(OVLDefaults#(a_type)
defaults)(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_decrement
Description Ensures that the expression decrements only by the value specifiedR.
Interface Used AssertTest_IFC
Parameters common assertion parameters
value(default value = 1)
Module Declaration
module bsv_assert_decrement#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea), Literal#(a_type),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_delta
Description Ensures that the expression always changes by a value within the range
specified byminandmax.
Interface Used AssertTest_IFC
Parameters common assertion parameters
min(default value = 1)
max(default value = 1)
Module Declaration
module bsv_assert_delta#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea), Literal#(a_type),
Bounded#(a_type), Eq#(a_type));
```

Bluespec SystemVerilog Reference Guide

```
Module bsv_assert_even_parity
Description Ensures that value of a specified expression has even parity, that is an
even number of bits in the expression are active high.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_even_parity#(OVLDefaults#(a_type)
defaults) (AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_fifo_index
Description Ensures that a FIFO-type structure never overflows or underflows. This
checker can be configured to support multiple pushes (FIFO writes) and
pops (FIFO reads) during the same clock cycle.
Interface Used AssertFifoTest_IFC
Parameters common assertion parameters
depth(default value = 1)
simultaneous_push_pop(default value =True)
Module Declaration
module bsv_assert_fifo_index#(OVLDefaults#(Bit#(0))
defaults)(AssertFifoTest_IFC#(a_type, b_type))
provisos (Bits#(a_type, sizea), Bits#(b_type, sizeb));
```
```
Module bsv_assert_frame
Description Checks that once the start method is asserted, the test expression eval-
uates true not beforemin_cksclock cycles and not aftermax_cksclock
cycles.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
action_on_new_start(default value =OVL_IGNORE_NEW_START)
min_cks(default value = 1)
max_cks(default value = 1)
Module Declaration
module bsv_assert_frame#(OVLDefaults#(Bool) defaults)
(AssertStartTest_IFC#(Bool));
```
```
Module bsv_assert_handshake
Description Ensures that the specified request and acknowledge signals follow a spec-
ified handshake protocol.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
action_on_new_start(default value =OVL_IGNORE_NEW_START)
min_ack_cycle(default value = 1)
max_ack_cycle(default value = 1)
Module Declaration
module bsv_assert_handshake#(OVLDefaults#(Bool) defaults)
(AssertStartTest_IFC#(Bool));
```

Reference Guide Bluespec SystemVerilog

```
Module bsv_assert_implication
Description Ensures that a specified consequent expression isTrueif the specified
antecedent expression isTrue.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_implication#(OVLDefaults#(Bool) defaults)
(AssertStartTest_IFC#(Bool));
```
```
Module bsv_assert_increment
Description ensure that the test expression always increases by the value of specified
byvalue.
Interface Used AssertTest_IFC
Parameters common assertion parameters
value(default value = 1)
Module Declaration
module bsv_assert_increment#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea), Literal#(a_type),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_never
Description Ensures that the value of a specified expression is neverTrue.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_never#(OVLDefaults#(Bool) defaults)
(AssertTest_IFC#(Bool));
```
```
Module bsv_assert_never_unknown
Description Ensures that the value of a specified expression contains only 0 and 1
bits when a qualifying expression isTrue.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_never_unknown#(OVLDefaults#(a_type)
defaults)(AssertStartTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```

Bluespec SystemVerilog Reference Guide

```
Module bsv_assert_never_unknown_async
Description Ensures that the value of a specified expression always contains only 0
and 1 bits
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_never_unknown_async#(OVLDefaults#(a_type)
defaults)(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea), Literal#(a_type),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_next
Description Ensures that the value of the specified expression is true a specified
number of cycles after a start event.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
num_cks(default value = 1)
check_overlapping(default value =True)
check_missing_start(default value =False)
Module Declaration
module bsv_assert_next#(OVLDefaults#(Bool) defaults)
(AssertStartTest_IFC#(Bool));
```
```
Module bsv_assert_no_overflow
Description Ensures that the value of the specified expression does not overflow.
Interface Used AssertTest_IFC
Parameters common assertion parameters
min(default value =minBound)
max(default value =maxBound)
Module Declaration
module bsv_assert_no_overflow#(OVLDefaults#(a_type)
defaults) (AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_no_transition
Description Ensures that the value of a specified expression does not transition from
a start state to the specified next state.
Interface Used AssertTransitionTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_no_transition#(OVLDefaults#(a_type)
defaults) (AssertTransitionTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```

Reference Guide Bluespec SystemVerilog

```
Module bsv_assert_no_underflow
Description Ensures that the value of the specified expression does not underflow.
Interface Used AssertTest_IFC
Parameters common assertion parameters
min(default value =minBound)
max(default value =maxBound)
Module Declaration
module bsv_assert_no_underflow#(OVLDefaults#(a_type)
defaults)(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_odd_parity
Description Ensures that the specified expression had odd parity; that an odd num-
ber of bits in the expression are active high.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_odd_parity#(OVLDefaults#(a_type)
defaults)(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_one_cold
Description Ensures that exactly one bit of a variable is active low.
Interface Used AssertTest_IFC
Parameters common assertion parameters
inactive(default value =OLV_ONE_COLD)
Module Declaration
module bsv_assert_one_cold#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type))
```
```
Module bsv_assert_one_hot
Description Ensures that exactly one bit of a variable is active high.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_one_hot#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```

Bluespec SystemVerilog Reference Guide

```
Module bsv_assert_proposition
Description Ensures that the test expression is always combinationallyTrue. Like
assert_alwaysexcept that the test expression is not sampled by the
clock.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_proposition#(OVLDefaults#(Bool) defaults)
(AssertTest_IFC#(Bool));
```
```
Module bsv_assert_quiescent_state
Description Ensures that the value of a specified state expression equals a corre-
sponding check value if a specified sample event has transitioned to
TRUE.
Interface Used AssertQuiescentTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_quiescent_state#(OVLDefaults#(a_type)
defaults)(AssertQuiescentTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_range
Description Ensure that an expression is always within a specified range.
Interface Used AssertTest_IFC
Parameters common assertion parameters
min(default value =minBound)
max(default value =maxBound)
Module Declaration
module bsv_assert_range#(OVLDefaults#(a_type) defaults)
(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_time
Description Ensures that the expression remainsTruefor a specified number of clock
cycles after a start event.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
action_on_new_start(default value =OVL_IGNORE_NEW_START)
num_cks(default value = 1)
Module Declaration
module bsv_assert_time#(OVLDefaults#(Bool) defaults)
(AssertStartTest_IFC#(Bool));
```

Reference Guide Bluespec SystemVerilog

```
Module bsv_assert_transition
Description Ensures that the value of a specified expression transitions properly
froma start state to the specified next state.
Interface Used AssertTransitionTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_transition#(OVLDefaults#(a_type)
defaults)(AssertTransitionTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_unchange
Description Ensures that the value of the specified expression does not change during
a specified number of clock cycles after a start event initiates checking.
Interface Used AssertStartTest_IFC
Parameters common assertion parameters
action_on_new_start(default value =OVL_IGNORE_NEW_START)
num_cks(default value = 1)
Module Declaration
module bsv_assert_unchange#(OVLDefaults#(a_type) defaults)
(AssertStartTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_width
Description Ensures that when the test expression goes high it stays high for at least
minand at mostmaxclock cycles.
Interface Used AssertTest_IFC
Parameters common assertion parameters
min_cks(default value = 1)
max_cks(default value = 1)
Module Declaration
module bsv_assert_width#(OVLDefaults#(Bool) defaults)
(AssertTest_IFC#(Bool));
```
```
Module bsv_assert_win_change
Description Ensures that the value of a specified expression changes in a specified
window between a start event and a stop event.
Interface Used AssertStartStopTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_win_change#(OVLDefaults#(a_type)
defaults)(AssertStartStopTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```

Bluespec SystemVerilog Reference Guide

```
Module bsv_assert_win_unchange
Description Ensures that the value of a specified expression does not change in a
specified window between a start event and a stop event.
Interface Used AssertStartStopTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_win_unchange#(OVLDefaults#(a_type)
defaults)(AssertStartStopTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
```
Module bsv_assert_window
Description Ensures that the value of a specified event isTruebetween a specified
window between a start event and a stop event.
Interface Used AssertStartStopTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_window#(OVLDefaults#(Bool) defaults)
(AssertStartStopTest_IFC#(Bool));
```
```
Module bsv_assert_zero_one_hot
Description ensure that exactly one bit of a variable is active high or zero.
Interface Used AssertTest_IFC
Parameters common assertion parameters
Module Declaration
module bsv_assert_zero_one_hot#(OVLDefaults#(a_type)
defaults)(AssertTest_IFC#(a_type))
provisos (Bits#(a_type, sizea),
Bounded#(a_type), Eq#(a_type));
```
Example using bsvassertincrement

This example checks that a test expression is always incremented by a value of 3. The assertion
passes for the first 10 increments and then starts failing when the increment amount is changed from
3 to 1.

import OVLAssertions::*; // import the OVL Assertions package

module assertIncrement (Empty);

```
Reg#(Bit#(8)) count <- mkReg(0);
Reg#(Bit#(8)) test_expr <- mkReg(0);
```
```
// set the default values
let defaults = mkOVLDefaults;
```
```
// override the default increment value and set = 3
defaults.value = 3;
```

Reference Guide Bluespec SystemVerilog

```
// instantiate an instance of the module bsv_assert_increment using
// the name assert_mod and the interface AssertTest_IFC
AssertTest_IFC#(Bit#(8)) assert_mod <- bsv_assert_increment(defaults);
```
```
rule every (True); // Every clock cycle
assert_mod.test(test_expr); // the assertion is checked
endrule
```
rule increment (True);
count <= count + 1;
if (count < 10) // for 10 cycles
test_expr <= test_expr + 3; // increment the expected amount
else if (count < 15)
test_expr <= test_expr + 1; // then start incrementing by 1
else
$finish;
endrule
endmodule

Using The Library

In order to use the OVLAssertions package, a user must first download the source OVL library from
Accellera (http://www.accellera.org). In addition, that library must be made available when
building a simulation executable from the BSV generated Verilog.

If the bsc compiler is being used to generate the Verilog simulation executable, theBSC_VSIM_FLAGS
environment variable can be used to set the required simulator flags that enable use of the OVL
library.

For instance, if theiverilogsimulator is being used and the OVL library is located in the directory
shared/std_ovl, theBSC_VSIM_FLAGSenvironment variable can be set to ̈-I shared/std_ovl -Y
.vlib -y shared/std_ovl -DOVL_VERILOG=1 -DOVL_ASSERT_ON=1 ̈. These flags:

- Addshared/std_ovlto the Verilog and include search paths.
- Set.vlibas a possible file suffix.
- Set flags used in the OVL source code.

The exact flags to be used will differ based on what OVL behavior is desired and which Verilog
simulator is being used.

#### C.8.18 Printf

Package

import Printf:: * ;

Description

ThePrintfpackage provides thesprintffunction to allow users to construct strings using typical
C printf patterns. The function supports a full range of C-style format options.

Thesprintffunction uses two advanced features, type classes (Section 14.1) and partial function
application, to implement a variable number of arguments. That is why the type signature of the
function includes a proviso forSPrintfType, also defined in this package.


Bluespec SystemVerilog Reference Guide

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Type classes

ThePrintfpackage includes theSPrintfandPrintfArgtypeclasses. The provisoSPrintfspecifies
that the function can take a variable number of arguments, and further the types of those arguments
can be displayed. This last requirement is captured by thePrintfArgtypeclass, which is the class
of types that can be displayed.

typeclass SPrintfType#(type t);
function t spr(String fmt, List#(UPrintf) args);
endtypeclass

ThePrintfArgtypeclass defines a separate conversion for each type in the class.

typeclass PrintfArg#(type t);
function UPrintf toUPrintf(t arg);
endtypeclass

Functions

```
sprintf Constructs a string given a C-style format string and any input values for
that format.
function r sprintf(String fmt) provisos (SPrintfType#(r));
```
Thesprintffunction constructs a string from a format string followed by a variable number of
arguments. Examples:

String s1 = sprintf("Hello");

Bit#(8) x = 0;
String s2 = sprintf("x = %d", x);

Real r = 1.2;
String s3 = sprintf("x = %d, r = %g", x, r);

The behavior ofsprintfdepends on the types of the arguments. If the type of an argument is
unclear, you may be required to give specific types to those arguments.

For instance, an integer literal can represent many types, so you need to specify which one you are
using:

String s4 = sprintf("%d, %d", 1, 2); // ambiguous

// Example of two ways to specify the type
UInt#(8) n = 1;
String s4 = sprintf("%d, %d", n, Bit#(4)’(2));

When callingsprintfon a value whose type is not known, as in a polymorphic function, you may
be required to add a proviso to the function for the type variable.

ThePrintfArgproviso on polymorphic functions is required when the type of the argument is not
known. The type class instances define the conversion functions for each printable type.


Reference Guide Bluespec SystemVerilog

function Action disp(t x);
action
String s = sprintf("x=%d", x);
$display(s);
endaction
endfunction

will generate an error message. By adding the proviso, the function compiles correctly:

function Action disp(t x)
provisos( PrintfArg#(t) );
action
String s = sprintf("x=%d", x);
$display(s);
endaction
endfunction

#### C.8.19 BuildVector

Package

import BuildVector :: * ;

Description

TheBuildVectorpackage provides theBuildVectortype class to implement a vector construction
function which can take any number of arguments (>0).

In pseudo code, we can show this as:

```
function Vector#(n, a) vec(a v1, a v2, ..., a vn);
```
Examples:

```
Vector#(3, Bool) v1 = vec(True, False, True);
Vector#(4, Integer) v2 = vec(2, 17, 22, 42);
```
This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Functions

```
vec A function for creating a Vector of sizenfromnarguments. The variable number
of arguments is implemented via theBuildVectortypeclass, which is a proviso
of this function.
```
```
function r vec(a x) provisos(BuildVector#(a,r,0));
```

Bluespec SystemVerilog Reference Guide

### C.9 Multiple Clock Domains and Clock Generators

Package

import Clocks :: * ;

Description

The BSVClockslibrary provide features to access and change the default clock. Moreover, there
are hardware primitives to generate clocks of various shapes, plus several primitives which allow the
safe crossing of signals and data from one clock domain to another.

TheClockspackage uses the data typesClockandResetas well as clock functions which are
described below but defined in thePreludepackage.

Each section describes a related group of modules, followed by a table indicating the Verilog modules
used to implement the BSV modules.

Types and typeclasses

TheClockspackage uses the abstract data typesClockandReset, which are defined in thePrelude
package. These are first class objects. BothClockandResetare in theEqtype class, meaning two
values can be compared for equality.

Clockis an abstract type of two components: a singleBitoscillator and aBoolgate.

```
typedef ... Clock ;
```
Resetis an abstract type.

```
typedef ... Reset ;
```
```
Type Classes forClockandReset
Bits Eq Literal Arith Ord Bounded Bitwise Bit Bit
Reduction Extend
Clock
```
##### √

```
Reset
```
##### √

Example: Declaring a new clock

```
Clock clk0;
```
Example: Instantiating a register with clock and reset

```
Reg#(Byte) a <- mkReg(0, clocked_by clks0, reset_by rst0);
```
Functions

The following functions are defined in thePreludepackage but are used with multiple clock domains.

```
Clock Functions
```
```
exposeCurrentClock This function returns a value of typeClock, which is the current clock
of the module.
```
```
module exposeCurrentClock ( Clock c );
```

Reference Guide Bluespec SystemVerilog

```
exposeCurrentReset This function returns a value of typeReset, which is the current reset
of the module.
```
```
module exposeCurrentReset ( Reset r );
```
BothexposeCurrentClockandexposeCurrentResetuse the module instantiation syntax (<-) to
return the value. Hence these can only be used from within a module.

Example: setting a reset to the current reset

```
Reset reset_value <- exposeCurrentReset;
```
Example: setting a clock to the current clock

```
Clock clock_value <- exposeCurrentClock;
```
```
sameFamily A Boolean function which returnsTrueif the clocks are in the same
family,Falseif the clocks are not in the same family. Clocks in the
same family have the same oscillator but may have different gate con-
ditions.
```
```
function Bool sameFamily ( Clock clka, Clock clkb ) ;
```
```
isAncestor A Boolean function which returnsTrueifclkais an ancestor ofclkb,
that isclkbis a gated version ofclka(clkaitself may be gated) or if
clkaandclkbare the same clock. The ancestry relation is a partial
order (ie., reflexive, transitive and antisymmetric).
```
```
function Bool isAncestor ( Clock clka, Clock clkb ) ;
```
```
clockOf Returns the current clock of the objectobj.
```
```
function Clock clockOf ( a_type obj ) ;
```
```
noClock Specifies anullclock, a clock where the oscillator never rises.
```
```
function Clock noClock() ;
```
```
resetOf Returns the current reset of the objectobj.
```
```
function Reset resetOf ( a_type obj ) ;
```

Bluespec SystemVerilog Reference Guide

```
noReset Specifies anullreset, a reset which is never asserted.
```
```
function Reset noReset() ;
```
```
invertCurrentClock Returns a value of typeClock, which is the inverted current clock of
the module.
```
```
module invertCurrentClock(Clock);
```
```
invertCurrentReset Returns a value of typeReset, which is the inverted current reset of
the module.
```
```
module invertCurrentReset(Reset);
```
#### C.9.1 Clock Generators and Clock Manipulation

Description

This section provides modules to generate new clocks and to modify the existing clock.

The modulesmkAbsoluteClock,mkAbsoluteClockFull,mkClock, andmkUngatedClockall define a
new clock, one not based on the current clock. BothmkAbsoluteClockandmkAbsoluteClockFull
define new oscillators and are not synthesizable.mkClockandmkUngatedClockuse an existing oscil-
lator to create a clock, and is synthesizable. The modules,mkGatedClockandmkGatedClockFromCC
use existing clocks to generate another clock in the same family.

Interfaces and Methods

TheMakeClockIfcsupports user-defined clocks with irregular waveforms created withmkClock
andmkUngatedClock, as opposed to the fixed-period waveforms created with themkAbsoluteClock
family.

```
MakeClockIfc Interface
Method and subinterfaces Arguments
Name Type Description Name Description
setClockValue Action Changes the value of the
clock at the next edge of
the clock
```
```
value Value the clock will
be set to, must be a
one bit type
getClockValue one_bit_type Retrieves the last value of
the clock
setGateCond Action Changes the gating condi-
tion
```
```
gate Must be of the type
Bool
getGateCond Bool Retrieves the last gating
condition set
new_clk Interface Clock interface provided
by the module
```

Reference Guide Bluespec SystemVerilog

```
interface MakeClockIfc#(type one_bit_type);
method Action setClockValue(one_bit_type value) ;
method one_bit_type getClockValue() ;
method Action setGateCond(Bool gate) ;
method Bool getGateCond() ;
interface Clock new_clk ;
endinterface
```
TheGatedClockIfcis used for adding a gate to an existing clock.

```
GatedClockIfc Interface
Method and subinterfaces Arguments
Name Type Description Name Description
setGateCond Action Changes the gating condi-
tion
```
```
gate Must be of the type
Bool
getGateCond Bool Retrieves the last gating
condition set
new_clk Interface Clock interface provided
by the module
```
```
interface GatedClockIfc ;
method Action setGateCond(Bool gate) ;
method Bool getGateCond() ;
interface Clock new_clk ;
endinterface
```
Modules

ThemkClockmodule creates a Clock type from a one-bit oscillator and a Boolean gate condition.
There is no family relationship between the current clock and the clock generated by this module.
The initial values of the oscillator and gate are passed as parameters to the module. When the
module is out of reset, the oscillator value can be changed using thesetClockValuemethod and the
gate condition can be changed by calling thesetGateCondmethod. The oscillator value and gate
condition can be queried with thegetClockValueandgetGateCondmethods, respectively. The
clock created bymkClockis available as thenew_clksubinterface. When setting the gate condition,
the change does not affect the generated clock until it is low, to prevent glitches.

ThemkUngatedClockmodule is an ungated version of themkClockmodule. It takes only an oscillator
argument (no gate argument) and returns the samenew_clockinterface. Since there is no gate,
an error is returned if the design calls thesetGetCondmethod. ThegetGateCondmethod always
returns True.

```
mkClock Creates a Clock type from a one-bit oscillator input, and a Boolean gate
condition. There is no family relationship between the current clock and the
clock generated by this module.
```
```
module mkClock #( one_bit_type initVal, Bool initGate)
( MakeClockIfc#(one_bit_type) ifc )
provisos( Bits#(one_bit_type, 1) ) ;
```

Bluespec SystemVerilog Reference Guide

```
Figure 6: Clock Generator
```
```
mkUngatedClock Creates an ungated Clock type from a one-bit oscillator input. There is no
family relationship between the current clock and the clock generated by this
module.
```
```
module mkUngatedClock #( one_bit_type initVal)
( MakeClockIfc#(one_bit_type) ifc )
provisos( Bits#(one_bit_type, 1) ) ;
```
ThemkGatedClockmodule adds (logic and) a Boolean gate condition to an existing clock, thus
creating another clock in the same family. The source clock is provided as the argumentclk_in.
The gate condition is controlled by an asynchronously-reset register inside the module. The register
is set with thesetGateCondAction method of the interface and can be read withgetGateCond
method. The reset value of the gate condition register is provided as an instantiation parameter.
The clock for the register (and thus these set and get methods) is the default clock of the module;
to specify a clock other than the default clock, use theclocked_bydirective.

```
Figure 7: Gated Clock Generator
```
```
mkGatedClock Creates another clock in the same family by adding logic and a Boolean gate
condition to the current clock.
```
```
module mkGatedClock#(Bool v) ( Clock clk_in, GatedClockIfc ifc );
```

Reference Guide Bluespec SystemVerilog

For convenience, we provide an alternate version in which the source clock is the default clock of the
module

```
mkGatedClockFromCC An alternate interface for the modulemkGatedClockin which the source
clock is the default clock of the module.
```
```
module mkGatedClockFromCC#(Bool v) ( GatedClockIfc ifc );
```
The modulesmkAbsoluteClockandmkAbsoluteClockFullprovide parametizable clock generation
modules which arenotsynthesizable, but may be useful for testbenches. InmkAbsoluteClock, the
first rising edge (start) and the period are defined by parameters. These parameters are measured
in Verilog delay times, which are usually specified during simulation with thetimescaledirective.
Refer to the Verilog LRM for more details on delay times. s Additional parameters are provided by
mkAbsoluteClockFull.

```
mkAbsoluteClock The first rising edge (start) and period are defined by parameters.
This module is not synthesizable.
```
```
module mkAbsoluteClock #( Integer start,
Integer period )
( Clock );
```
```
mkAbsoluteClockFull The valueinitValueis held until timestart, and then the clock
oscillates. The valuenot(initValue)is held for timecompValTime,
followed byinitValueheld for timeinitValTime. Hence the clock
period after startup iscompValTime + initValTime. This module is
not synthesizable.
```
```
module mkAbsoluteClockFull #( Integer start,
Bit#(1) initValue,
Integer compValTime,
Integer initValTime )
( Clock );
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkAbsoluteClock ClockGen.v
mkAbsoluteClockFull
mkClock MakeClock.v
mkUngatedClock
mkGatedClock GatedClock.v
mkGatedClockFromCC
```

Bluespec SystemVerilog Reference Guide

#### C.9.2 Clock Multiplexing

Description

Bluespec provides two gated clock multiplexing primitives: a simple combinational multiplexor and
a stateful module which generates an appropriate reset signal when the clock changes. The first
multiplexor uses the interfaceMuxClockIfc, which includes an `Action` method to select the clock
along with aClocksubinterface. The second multiplexor uses the interfaceSelectClockIfcwhich
also has aResetsubinterface.

Ungated versions of these modules are also provided. The ungated versions are identical to the gated
versions, except that the input and output clocks are ungated.

Interfaces and Methods

```
MuxClockIfc Interface
Method and subinterfaces Arguments
Name Type Description Name Description
select Action Method used to select the
clock based on the Boolean
valueab
```
```
ab if True, clock_out is
taken fromaclk
```
```
clock_out Interface Clock interface
```
```
interface MuxClkIfc ;
method Action select ( Bool ab ) ;
interface Clock clock_out ;
endinterface
```
```
SelectClockIfc Interface
Method and subinterfaces Arguments
Name Type Description Name Description
select Action Method used to select the
clock based on the Boolean
valueab
```
```
ab if True, clockout is
taken fromaclk
```
```
clock_out Interface Clock interface
reset_out Interface Reset interface
```
```
interface SelectClkIfc ;
method Action select ( Bool ab ) ;
interface Clock clock_out ;
interface Reset reset_out ;
endinterface
```
Modules

ThemkClockMuxmodule is a simple combinational multiplexor with a registered clock selection
signal, which selects between clock inputsaClkandbClk. The provided Verilog module does not
provide any glitch detection or removal logic; it is the responsibility of the user to provide additional
logic to provide glitch-free behavior. ThemkClockMuxmodule uses two arguments and provides a
Clock interface. TheaClkis selected ifabis True, whilebClkis selected otherwise.

ThemkUngatedClockMuxmodule is identical to themkClockMuxmodule except that the input and
output clocks are ungated. The signalsaClkgate,bClkgate, andoutClkgatein figure 8 don’t exist.


Reference Guide Bluespec SystemVerilog

```
Figure 8: Clock Multiplexor
```
```
mkClockMux Simple combinational multiplexor, which selects between aClk and
bClk.
```
```
module mkClockMux ( Clock aClk, Clock bClk )
( MuxClkIfc ) ;
```
```
mkUngatedClockMux Simple combinational multiplexor, which selects between aClk and
bClk. None of the clocks are gated.
```
```
module mkUngatedClockMux ( Clock aClk, Clock bClk )
( MuxClkIfc ) ;
```
ThemkClockSelectmodule is a clock multiplexor containing additional logic which generates a
reset whenever a new clock is selected. As such, the interface for the module includes anAction
method to select the clock (ifabis True clockout is taken fromaClk), provides aClockinterface,
and also aResetinterface.

The constructor for the module uses two clock arguments, and provides theMuxClockIfcinterface.
The underlying Verilog module isClockSelect.v; it is expected that users can substitute their own
modules to meet any additional requirements they may have. The parameterstagesis the number
of clock cycles in which the reset is asserted after the clock selection changes.

ThemkUngatedClockSelectmodule is identical to themkClockSelectmodule except that the input
and output clocks are ungated. The signalsaClkgate,bClkgate, andoutClk_gatein figure 9 don’t
exist.

```
mkClockSelect Clock Multiplexor containing additional logic which generates a reset
whenever a new clock is selected.
```
```
module mkClockSelect #( Integer stages,
Clock aClk,
Clock bClk,
( SelectClockIfc ) ;
```

Bluespec SystemVerilog Reference Guide

```
Figure 9: Clock Multiplexor with reset
```
```
mkUngatedClockSelect Clock Multiplexor containing additional logic which generates a reset
whenever a new clock is selected. The input and output clocks are
ungated.
```
```
module mkUngatedClockSelect #( Integer stages,
Clock aClk,
Clock bClk,
( SelectClockIfc ) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkClockMux ClockMux.v
mkClockSelect ClockSelect.v
mkUngatedClockMux UngatedClockMux.v
mkUngatedClockSelect UngatedClockSelect.v
```
#### C.9.3 Clock Division

Description

A clock divider provides a derived clock and also aClkNextRdysignal, which indicates that the
divided clock will rise in the next cycle. This signal is associated with the input clock, and can only
be used within that clock domain.

TheAlignedFIFOspackage (Section C.2.6) contains parameterized FIFO modules for creating syn-
chronizing FIFOs between clock domains with aligned edges.

Data Types

TheClkNextRdyis a Boolean signal which indicates that the slow clock will rise in the next cycle.


Reference Guide Bluespec SystemVerilog

```
typedef Bool ClkNextRdy ;
```
Interfaces and Methods

```
ClockDividerIfc Interface
Name Type Description
fastClock Interface The original clock
slowClock Interface The derived clock
clockReady Bool Boolean value which indicates that the slow clock will rise
in the next cycle. The method is in the clock domain of the
fast clock.
```
```
interface ClockDividerIfc ;
interface Clock fastClock ;
interface Clock slowClock ;
method ClkNextRdy clockReady() ;
endinterface
```
Modules

Thedividerparameter may be any integer greater than 1. For even dividers the generated clock’s
duty cycle is 50%, while for odd dividers, the duty cycle is (divider/2)/divider. Sincedivisoris an
integer, the remainder is truncated when divided. The current clock (or theclocked_byargument)
is used as the source clock.

```
Figure 10: Clock Divider
```
```
mkClockDivider Basic clock divider.
module mkClockDivider #( Integer divisor )
( ClockDividerIfc ) ;
```
```
mkGatedClockDivider A gated verison of the basic clock divider.
```
```
module mkGatedClockDivider #( Integer divisor
)( ClockDividerIfc ) ;
```
ThemkClockDividerOffsetmodule provides a clock divider where the rising edge can be defined
relative to other clock dividers which have the same divisor. An offset of value 2 will produce a rising
edge one fast clock after a divider with offset 1. mkClockDivideris justmkClockDividerOffset
with an offset of value 0.


Bluespec SystemVerilog Reference Guide

```
mkClockDividerOffset Provides a clock divider, where the rising edge can be defined rel-
ative to other clock dividers which have the same divisor.
```
```
module mkClockDividerOffset #( Integer divisor,
Integer offset )
( ClockDividerIfc ) ;
```
ThemkClockInverterandmkGatedClockInvertermodules generate an inverted clock having the
same period but opposite phase as the current clock. ThemkGatedClockInverteris a gated version
ofmkClockInverter. The output clock includes a gate signal derived from the gate of the input
clock.

```
mkClockInverter Generates an inverted clock having the same period but opposite
phase as the current clock.
module mkClockInverter ( ClockDividerIfc ) ;
```
```
mkGatedClockInverter A gated version ofmkClockInverter.
```
```
module mkGatedClockInverter ( ClockDividerIfc ifc ) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkClockDivider ClockDiv.v
mkClockDividerOffset
mkGatedClockDivider GatedClockDiv.v
mkClockInverter ClockInverter.v
mkGatedClockInverter GatedClockInverter.v
```
#### C.9.4 Bit Synchronizers

Description

Bit synchronizers are used to safely transfer one bit of data from one clock domain to another. More
complicated synchronizers are provided in later sections.

Interfaces and Methods

TheSyncBitIfcinterface provides asendmethod which transmits one bit of information from one
clock domain to thereadmethod in a second domain.


Reference Guide Bluespec SystemVerilog

```
SyncBitIfc Interface
Methods Arguments
Name Type Description Name Description
send Action Transmits information from
one clock domain to the sec-
ond domain
```
```
bitData One bit of information
transmitted
```
```
read one_bit_type Reads one bit of data sent
from a different clock domain
```
```
interface SyncBitIfc #(type one_bit_type) ;
method Action send ( one_bit_type bitData ) ;
method one_bit_type read () ;
endinterface
```
Modules

ThemkSyncBit,mkSyncBitFromCCandmkSyncBitToCCmodules provide aSyncBitIfcacross clock
domains. The send method is in one clock domain, and the read method is in a second clock
domain, as shown in Figure 11. TheFromCCandToCCversions differ in that theFromCCmodule
moves datafromthe current clock (module’s clock), while theToCCmodule moves datatothe current
clock domain. The hardware implementation is a two register synchronizer, which can be found in
SyncBit.vin the Bluespec Verilog library directory.

```
Figure 11: Bit Synchronizer
```
```
mkSyncBit Moves data across clock domains. The in and out clocks, along with
the input reset, are explicitly provided. The default clock and reset
are ignored.
```
```
module mkSyncBit #( Clock sClkIn, Reset sRst,
Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBitFromCC Moves data from the current clock (the module’s clock) to a different
clock domain. The input clock and reset are the current clock and
reset.
```
```
module mkSyncBitFromCC #( Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```

Bluespec SystemVerilog Reference Guide

```
mkSyncBitToCC Moves data into the current clock domain. The output clock is the
current clock. The current reset is ignored.
```
```
module mkSyncBitToCC #( Clock sClkIn, Reset sRstIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
ThemkSyncBit15 module (one and a half) and its variants provide the same interface as the
mkSyncBitmodules, but the underlying hardware is slightly modified, as shown in Figure 12. For
these synchronizers, the first register clocked by the destination clock triggers on the falling edge of
the clock.

```
Figure 12: Bit Synchronizer 1.5 - first register in destination domain triggers on falling edge
```
```
mkSyncBit15 Similar tomkSyncBitexcept it triggers on the falling edge of the clock.
The in and out clocks, along with the input reset, are explicitly pro-
vided. The default clock and reset are ignored.
```
```
module mkSyncBit15 #( Clock sClkIn, Reset sRst,
Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBit15FromCC Moves data from the current clock and is triggered on the falling edge
of the clock. The input clock and reset are the current clock and reset.
```
```
module mkSyncBit15FromCC #(Clock dClkIn)
(SyncBitIfc #(one_bit_type))
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBit15ToCC Moves data into the current clock domain and is triggered on the falling
edge of the clock. The output clock is the current clock. The current
reset is ignored.
```
```
module mkSyncBit15ToCC #( Clock sClkIn, Reset sRstIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```

Reference Guide Bluespec SystemVerilog

ThemkSyncBit1module, shown in Figure 13, also provides the same interface but only uses one
register in the destination domain. Synchronizers like this, which use only one register, are not
generally used since meta-stable output is more probable. However, one can use this synchronizer
provided special meta-stable resistant flops are selected during physical synthesis or (for example) if
the output is immediately registered.

```
Figure 13: Bit Synchronizer 1.0 - single register in destination domain
```
```
mkSyncBit1 Moves data from one clock domain to another clock domain, with only
one register in the destination domain. The in and out clocks, along
with the input reset, are explicitly provided. The default clock and
reset are ignored.
```
```
module mkSyncBit1 #( Clock sClkIn, Reset sRst,
Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBit1FromCC Moves data from the current clock domain, with only one register in
the destination domain. The input clock and reset are the current
clock and reset.
```
```
module mkSyncBit1FromCC #( Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits #(one_bit_type, 1)) ;
```
```
mkSyncBit1ToCC Moves data into the current clock domain, with only one register in
the destination domain. The output clock is the current clock. The
current reset is ignored.
```
```
module mkSyncBit1ToCC #( Clock sClkIn, Reset sRstIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
ThemkSyncBit05module is similar tomkSyncBit1, but the destination register triggers on the
falling edge of the clock, as shown in Figure 14.


Bluespec SystemVerilog Reference Guide

```
Figure 14: Bit Synchronizer .5 - first register in destination domain triggers on falling edge
```
```
mkSyncBit05 Moves data from one clock domain to another clock domain, with
only one register in the destination domain. The destination register
triggers on the falling edge of the clock. The in and out clocks, along
with the input reset, are explicitly provided. The default clock and
reset are ignored.
```
```
module mkSyncBit05 #( Clock sClkIn, Reset sRst,
Clock dClkIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBit05FromCC Moves data from the current clock domain, with only one register in
the destination domain, the destination register triggers on the falling
edge of the clock. The input clock and reset are the current clock and
reset.
```
```
module mkSyncBit05FromCC #( Clock dClkIn )
(SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
```
mkSyncBit05ToCC Moves data into the current clock domain, with only one register in
the destination domain, the destination register triggers on the falling
edge of the clock. The output clock is the current clock. The current
reset is ignored.
```
```
module mkSyncBit05ToCC #( Clock sClkIn, Reset sRstIn )
( SyncBitIfc #(one_bit_type) )
provisos( Bits#(one_bit_type, 1)) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.


Reference Guide Bluespec SystemVerilog

```
BSV Module Name Verilog Module Name
```
```
mkSyncBit SyncBit.v
mkSyncBitFromCC
mkSyncBitToCC
mkSyncBit15 SyncBit15.v
mkSyncBit15FromCC
mkSyncBit15ToCC
mkSyncBit1 SyncBit1.v
mkSyncBit1FromCC
mkSyncBit1ToCC
mkSyncBit05 SyncBit05.v
mkSyncBit05FromCC
mkSyncBit05ToCC
```
#### C.9.5 Pulse Synchronizers

Description

Pulse synchronizers are used to transfer a pulse from one clock domain to another.

Interfaces and Methods

TheSyncPulseIfcinterface provides an Action method,send, which when invoked generates a True
value on thepulsemethod in a second clock domain.

```
SyncPulseIfc Interface
Methods
Name Type Description
send Action Starts transmittling a pulse from one clock domain to the
second clock domain.
pulse Bool Where the pulse is received in the second domain.pulseis
True if a pulse is recieved in this cycle.
```
```
interface SyncPulseIfc ;
method Action send () ;
method Bool pulse () ;
endinterface
```
Modules

ThemkSyncPulse,mkSyncPulseFromCCandmkSyncPulseToCCmodules provide clock domain cross-
ing modules for pulses. When thesendmethod is called from the one clock domain, a pulse will be
seen on thereadmethod in the second. Note that there is no handshaking between the domains,
so when sending data from a fast clock domain to a slower one, not all pulses sent may be seen in
the slower receiving clock domain. The pulse delay is two destination clocks cycles.

```
mkSyncPulse Sends a pulse from one clock domain to another. The in and out
clocks, along with the input reset, are explicitly provided. The default
clock and reset are ignored.
```
```
module mkSyncPulse #( Clock sClkIn, Reset sRstIn,
Clock dClkIn )
( SyncPulseIfc ) ;
```

Bluespec SystemVerilog Reference Guide

```
Figure 15: Pulse Synchronizer - no handshake
```
```
mkSyncPulseFromCC Sends a pulse from the current clock domain to the other clock domain.
The input clock and reset are the current clock and reset.
```
```
module mkSyncPulseFromCC #( Clock dClkIn )
( SyncPulseIfc ) ;
```
```
mkSyncPulseToCC Sends a pulse from the other clock domain to the current clock domain.
The output clock is the current clock. The current reset is ignored.
```
```
module mkSyncPulseToCC #( Clock sClkIn, Reset sRstIn )
( SyncPulseIfc ) ;
```
ThemkSyncHandshake,mkSyncHandshakeFromCCandmkSyncHandshakeToCCmodules provide clock
domain crossing modules for pulses in a similar way asmkSyncPulsemodules, except that a hand-
shake is provided in themkSyncHandshakeversions. The handshake enforces that another send does
not occur before the first pulse crosses to the other domain. Note that this only guarantees that
the pulse is seen in one clock cycle of the destination; it does not guarantee that the system on that
side reacted to the pulse before it was gone. It is up to the designer to ensure this, if necessary. The
modules are not ready in reset.

The pulse delay from thesendmethod to thereadmethod is two destination clocks. Thesend
method is re-enabled in two destination clock cycles plus two source clock cycles after thesend
method is called.

```
mkSyncHandshake Sends a pulse from one clock domain to another clock domain with
handshaking. The in and out clocks, along with the input reset, are
explicitly provided. The default clock and reset are ignored.
```
```
module mkSyncHandshake #( Clock sClkIn, Reset sRstIn,
Clock dClkIn )
( SyncPulseIfc ) ;
```
```
mkSyncHandshakeFromCC Sends a pulse with a handshake from the current clock domain.
The input clock and reset are the current clock and reset.
```
```
module mkSyncHandshakeFromCC #( Clock dClkIn )
( SyncPulseIfc ) ;
```

Reference Guide Bluespec SystemVerilog

```
Figure 16: Pulse Synchronizer with handshake
```
```
mkSyncHandshakeToCC Sends a pulse with a handshake to the current clock domain. The
output clock is the current clock. The current reset is ignored.
```
```
module mkSyncHandshakeToCC #( Clock sClkIn,
Reset sRstIn )
( SyncPulseIfc ) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkSyncPulse SyncPulse.v
mkSyncPulseFromCC
mkSyncPulseToCC
mkSyncHandshake SyncHandshake.v
mkSyncHandshakeFromCC
mkSyncHandshakeToCC
```
#### C.9.6 Word Synchronizers

Description

Word synchronizers are used to provide word synchronization across clock domains. The crossings
are handshaked, such that a second write cannot occur until the first is acknowledged (that the data
has been received, but the value may not have been read) by the destination side. The destination
read is registered.

Interfaces and Methods


Bluespec SystemVerilog Reference Guide

Word synchronizers use the commonReginterface (redescribed below), but there are a few subtle
differences which the designer should be aware. First, the_readand_writemethods are in different
clock domains and, second, the_writemethod has an implicit “ready” condition which means that
some synchronization modules cannot be written every clock cycle. Both of these conditions are
handled automatically by the Bluespec compiler relieving the designer of these tedious checks.

```
RegInterface
Method Arguments
Name Type Description Name Description
_write Action Writes a valuex1 x1 Data to be written
_read a_type Returns the value of the reg-
ister
```
```
interface Reg #(a_type);
method Action _write(a_type x1);
method a_type _read();
endinterface: Reg
```
Modules

ThemkSyncReg,mkSyncRegToCCandmkSyncRegFromCCmodules provide word synchronization across
clock domains.

Figure 17: Register Synchronization Module (see Figure 16 for the pulse synchronizer with hand-
shake)

```
mkSyncReg Provides word synchronization across clock domains. The in and out
clocks, along with the input reset, are explicitly provided. The default
clock and reset are ignored.
```
```
module mkSyncReg #( a_type initValue,
Clock sClkIn, Reset sRstIn,
Clock dClkIn )
( Reg #(a_type) )
provisos (Bits#(a_type, sa) ) ;
```

Reference Guide Bluespec SystemVerilog

```
mkSyncRegFromCC Provides word synchronization from the current clock domain. The
input clock and reset are the current clock and reset.
```
```
module mkSyncRegFromCC #( a_type initValue,
Clock dClkIn )
( Reg #(a_type) )
provisos (Bits#(a_type, sa)) ;
```
```
mkSyncRegToCC Provides word synchronization to the current clock domain. The out-
put clock is the current clock. The current reset is ignored.
```
```
module mkSyncRegToCC #( a_type initValue,
Clock sClkIn, Reset sRstIn )
( Reg #(a_type) )
provisos (Bits#(a_type, sa)) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkSyncReg SyncRegister.v
mkSyncRegFromCC
mkSyncRegToCC
```
#### C.9.7 FIFO Synchronizers

Description

TheSyncFIFOmodules use FIFOs to synchronize data being sent across clock domains, provid-
ing registered full and empty signals (notFullandnotEmpty). Additional FIFO synchronizers,
SyncFIFOLevelandSyncFIFOCountcan be found in theFIFOLevelpackage (Section C.2.3).

Interfaces and Methods

TheSyncFIFOIfcinterface defines an interface similar to the FIFOF interface, except it does not
have aclearmethod.


Bluespec SystemVerilog Reference Guide

```
SyncFIFOIfcInterface
Method Arguments
Name Type Description Name Description
enq Action Adds an entry to the FIFO sendData Data to be added
deq Action Removes the first entry from
the FIFO
first a_type Returns the first entry
notFull Bool Returns True if there is space
and you can enq into the
FIFO
notEmpty Bool Returns True if there are el-
ements in the FIFO and you
candeqfrom the FIFO
```
```
interface SyncFIFOIfc #(type a_type) ;
method Action enq ( a_type sendData ) ;
method Action deq () ;
method a_type first () ;
method Bool notFull () ;
method Bool notEmpty () ;
endinterface
```
Modules

```
Figure 18: Synchronization FIFOs
```
ThemkSyncFIFO,mkSyncFIFOFromCCandmkSyncFIFOToCCmodules provide FIFOs for sending data
across clock domains. Data items enqueued on the source side will arrive at the destination side and
remain there until they are dequeued. The depth of the FIFO is specified by thedepthparameter.
The full and empty signals are registered. The modulemkSyncFIFO1is a 1 element synchronized
FIFO.


Reference Guide Bluespec SystemVerilog

```
mkSyncFIFO Provides a FIFO for sending data across clock domains. Theenq
method is in the source (sClkIn) domain, while thedeqandfirst
methods are in the destination (dClkIn) domain. The in and out
clocks, along with the input reset, are explicitly provided. The default
clock and reset are ignored.
module mkSyncFIFO #( Integer depth,
Clock sClkIn, Reset sRstIn,
Clock dClkIn )
( SyncFIFOIfc #(a_type) )
provisos (Bits#(a_type, sa));
```
```
mkSyncFIFOFromCC Provides a FIFO to send data from the current clock domain into a
second clock domain. The input clock and reset are the current clock
and reset.
```
```
module mkSyncFIFOFromCC #( Integer depth,
Clock dClkIn )
( SyncFIFOIfc #(a_type) )
provisos (Bits#(a_type, sa));
```
```
mkSyncFIFOToCC Provides a FIFO to send data from a second clock domain into the
current clock domain. The output clock is the current clock. The
current reset is ignored.
```
```
module mkSyncFIFOToCC #( Integer depth,
Clock sClkIn, Reset sRstIn )
( SyncFIFOIfc #(a_type) )
provisos (Bits#(a_type, sa));
```
```
mkSyncFIFO1 Provides a 1 element FIFO for sending data across clock domains.
The 1 element module does not have a dedicated output register and
registers for full and empty, as in the depth>1 module. This module
should be used in clock crossing applications where complete FIFO
handshaking is required, but data throughput or storage is minimal.
```
```
module mkSyncFIFO #( Clock sClkIn, Reset sRstIn,
Clock dClkIn )
( SyncFIFOIfc #(a_type) )
provisos (Bits#(a_type, sa));
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.


Bluespec SystemVerilog Reference Guide

```
BSV Module Name Verilog Module Name
```
```
mkSyncFIFO SyncFIFO.v
mkSyncFIFOFromCC
mkSyncFIFOToCC
mkSyncFIFO1 SyncFIFO1.v
```
#### C.9.8 Asynchronous RAMs

Description

An asynchronous RAM provides a domain crossing by having its read and write methods in separate
clock domains.

Interfaces and Methods

```
DualPortRamIfcInterface
Method Arguments
Name Type Description Name Description
write Action Writes data to a an ad-
dress in a RAM
```
```
wr_addr Address of datatypeaddr_t
```
```
din Data of datatypedata_t
read data_d Reads the data from the
RAM
```
```
rd_addr Address to be read from
```
```
interface DualPortRamIfc #(type addr_t, type data_t);
method Action write( addr_t wr_addr, data_t din );
method data_t read ( addr_t rd_addr);
endinterface: DualPortRamIfc
```
```
Figure 19: Ansynchronous RAM
```
```
mkDualRam Provides an asynchronous RAM for when the read and the write meth-
ods are in separate clock domains. The write method is clocked by the
default clock, the read method is not clocked.
```
```
module mkDualRam( DualPortRamIfc #(addr_t, data_t) )
provisos ( Bits#(addr_t, sa),
Bits#(data_t, da) ) ;
```

Reference Guide Bluespec SystemVerilog

Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkDualRam DualPortRam.v
```
#### C.9.9 Null Crossing Primitives

Description

In these primitives, no synchronization is actually done. It is up to the designer to verify that it is
safe for the signal to be used in the other domain. ThemkNullCrossingWireis a wire synchronizer.
ThemkNullCrossingRegmodules are equivalent to a register (mkReg,mkRegA, ormkRegUdepending
on the module) followed by amkNullCrossingWire.

The oldermkNullCrossingprimitive is deprecated.

Interfaces

ThemkNullCrossingWiremodule, shown in Figure 20, provides theReadOnlyinterface which is
defined in the Prelude library B.4.8.

ThemkNullCrossingRegmodules provide theCrossingReginterface.

Interfaces and Methods

```
CrossingRegInterface
Method Arguments
Name Type Description Name Description
_write Action Writes a valuedatain datain Data to be written.
_read a_type Returns the value of the
register in the source clock
domain
crossed a_type Returns the value of the
register in the destination
clock domain
```
interface CrossingReg #( type a_type ) ;
method Action _write(a datain) ;
method a_type _read() ;
method a_type crossed() ;
endinterface

Modules

```
mkNullCrossingWire Defines a synchronizer that contains only a wire. It is left up to the
designer to ensure the clock crossing is safe.
```
```
module mkNullCrossingWire #( Clock dClk, a_type dataIn )
( ReadOnly#(a_type) )
provisos (Bits#(a_type, sa)) ;
```

Bluespec SystemVerilog Reference Guide

```
Figure 20: Wire synchronizer
```
```
Figure 21: Register with wire synchronizer
```
```
mkNullCrossingReg Defines a synchronizer that contains a register with a synchronous reset
value, followed by a wire synchronizer. It is left up to the designer to
ensure the clock crossing is safe.
module mkNullCrossingReg( Clock dClk, a_type resetval,
CrossingReg#(a_type) ifc )
provisos (Bits#(a_type, sz_a)) ;
```
```
mkNullCrossingRegA Defines a synchronizer that contains a register with a given reset value
where reset is asynchronous, followed by a wire synchronizer. It is left
up to the designer to ensure the clock crossing is safe.
```
```
module mkNullCrossingRegA( Clock dClk, a_type resetval,
CrossingReg#(a_type) ifc )
provisos (Bits#(a_type, sz_a)) ;
```
```
mkNullCrossingRegU Defines a synchronizer that contains a register without a reset, followed
by a wire synchronizer. It is left up to the designer to ensure the clock
crossing is safe.
```
```
module mkNullCrossingRegU( Clock dClk,
CrossingReg#(a_type) ifc )
provisos (Bits#(a_type, sz_a)) ;
```
Example: instantiating a null synchronizer

```
// domain2sig is domain1sig synchronized to clk0 with just a wire.
ReadOnly#(Bit#(2)) domain2sig <- mkNullCrossingWire (clk0, domain1sig);
```

Reference Guide Bluespec SystemVerilog

Note: no synchronization is actually done. This is purely a way to tell BSC that it is safe to use the
signal in the other domain. It is the responsibility of the designer to verify that this is correct.

There are some restrictions on the use of amkNullCrossingWire. The expression used as the data
argument must not have an implicit condition, and there cannot be another rule which is required
to schedule before any method called in the expression.

mkNullCrossingWires may not be used in sequence to pass a signal across multiple clock boundaries
without synchronization. Once a signal has been crossed from one domain to a second domain
without synchronization, it cannot be subsequently passed unsynchronized to a third domain (or
back to the first domain).

Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name
```
```
mkNullCrossingWire BypassWire.v
```
#### C.9.10 Reset Synchronization and Generation

Description

This section describes the interfaces and modules used to synchronize reset signals from one clock
domain to another and to create reset signals. Reset generation converts a Boolean type to a Reset
type, where the reset is associated with the default orclocked_byclock domain.

Interfaces and Methods

TheMakeResetIfcinterface is provided by the reset generatorsmkResetandmkResetSync.

```
MakeResetIfcInterface
Method
Name Type Description
assertReset Action Method used to assert the reset
isAsserted Bool Indicates whether the reset is asserted
new_rst Reset Generated output reset
```
```
interface MakeResetIfc;
method Action assertReset();
method Bool isAsserted();
interface Reset new_rst;
endinterface
```
The interfaceMuxRstIfcis provided by themkResetMuxmodule.

```
MuxRstIfcInterface
Method Arguments
Name Type Description Name Description
select Action Method used to select
the reset based on the
Boolean valueab
```
```
ab Value determines which
input reset to select
```
```
reset_out Reset Generated output reset
```

Bluespec SystemVerilog Reference Guide

```
interface MuxRstIfc;
method Action select ( Bool ab );
interface Reset reset_out;
endinterface
```
Modules

Reset Synchronization To synchronize resets from one clock domain to another, both syn-
chronous and asynchronous modules are provided. Thestagesargument is the number of full clock
cycles the output reset is held for after the input reset is deasserted. This is shown as the number of
flops in figures 22 and 23. Specifying a 0 for thestagesargument results in the creation of a simple
wire betweensRstanddRstOut.

```
Figure 22: Module for asynchronous resets
```
```
mkAsyncReset Provides synchronization of a source reset (sRst) to the destination
domain. The output reset occurs immediately once the source reset is
asserted.
```
```
module mkAsyncReset #( Integer stages,
Reset sRst,
Clock dClkIn )
( Reset ) ;
```
```
mkAsyncResetFromCR Provides synchronization of the current reset to the destination do-
main. There is no source resetsRstargument because it is taken
from the current reset. The output reset occurs immediately once the
current reset is asserted.
```
```
module mkAsyncResetFromCR #( Integer stages,
Clock dClkIn )
( Reset ) ;
```
The less commonmkSyncResetmodules are provided for convenience, but these modulesrequire
thatsRstbe held during a positive edge ofdClkInfor the reset assertion to be detected. Both
mkSyncResetandmkSyncResetFromCRuse the model in figure 23.


Reference Guide Bluespec SystemVerilog

```
Figure 23: Module for synchronous resets
```
```
mkSyncReset Provides synchronization of a source reset (sRst) to the destination
domain. The reset is asserted at the next rising edge of the clock.
```
```
module mkSyncReset #( Integer stages
Reset sRst,
Clock dClkIn )
( Reset ) ;
```
```
mkSyncResetFromCR Provides synchronization of the current reset to the destination do-
main. The reset is asserted at the next rising edge of the clock.
```
```
module mkSyncResetFromCR #( Integer stages
Clock dClkIn )
( Reset ) ;
```
Example: instantiating a reset synchronizer

```
// 2 is the number of stages
Reset rstn2 <- mkAsyncResetFromCR (2, clk0);
```
```
// if stages = 0, the default reset is used directly
Reset rstn0 <- mkAsyncResetFromCR (0, clk0);
```
Reset Generation Two modules are provided for reset generation,mkResetandmkResetSync,
where each module has one parameter,stages. Thestagesparameter is the number of full clock
cycles the output reset is held after theinRst, as seen in figure 24, is deasserted. Specifying a 0
for thestagesparameter results in the creation of a simple wire between the input register and
the output reset. That is, the reset is asserted immediately and not held after the input reset is
deasserted. It becomes the designer’s responsibility to ensure that the input reset is asserted for
sufficient time to allow the design to reset properly. The reset is controlled using theassertReset
method of theMakeResetIfcinterface.

The difference betweenmkResetandmkResetSync is that for the former, the assertion of reset
is immediate, while the later asserts reset at the next rising edge of the clock. Note that use of
mkResetSyncis less common, since the reset requires clock edges to take effect; failure to assert
reset for a clock edge will result in a reset not being seen at the output reset.


Bluespec SystemVerilog Reference Guide

```
Figure 24: Module for generating resets
```
```
mkReset Provides conversion of a Boolean type to a Reset type, where the reset
is associated withdClkIn. This module uses the model in figure 24.
startInRstindicates the reset value of the register. IfstartInRst
is True, the reset value of the register is 0, which means the output
reset will be asserted whenever the currentReset (sRst) is asserted.
rst_outwill remain asserted for the number of clock cycles given
by the stages parameter aftersRstis deasserted. IfstartInRstis
False, the output reset will not be asserted whensRstis asserted,
but only when theassert_resetmethod is invoked. At the start of
simulationrst_outwill only be asserted ifstartinRstis True and
sRstis initially asserted.
```
```
module mkReset #( Integer stages,
Bool startInRst,
Clock dClkIn )
( MakeResetIfc ) ;
```
```
mkResetSync Provides conversion of a Boolean type to a Reset type, where the reset
is associated withdClkInand the assertion of reset is at the next
rising edge of the clock. This module uses the model in figure 24.
startInRstindicates the reset value of the register. IfstartInRst
is True, the reset value of the register is 0, which means the output
reset will be asserted whenever the currentReset (sRst) is asserted.
rst_outwill remain asserted for the number of clock cycles given
by the stages parameter aftersRstis deasserted. IfstartInRstis
False, the output reset will not be asserted whensRstis asserted,
but only when theassert_resetmethod is invoked. At the start of
simulationrst_outwill only be asserted ifstartinRstis True and
sRstis initially asserted.
```
```
module mkResetSync #( Integer stages,
Bool startInRst,
Clock dClkIn )
( MakeResetIfc ) ;
```

Reference Guide Bluespec SystemVerilog

A reset multiplexormkResetMux, as seen in figure 25, creates one reset signal by selecting between
two existing reset signals.

```
Figure 25: Reset Multiplexor
```
```
mkResetMux Multiplexor which selects between two input resets,aRstandbRst,
to create a single output resetrst_out. The reset is selected through
a Boolean value provided to theselectmethod where True selects
aRst.
```
```
module mkResetMux #( Reset aRst, Reset bRst )
( MuxRstIfc rst_out ) ;
```
For testbenches, in which an absolute clock is being created, it is helpful to generate a reset for
that clock. The modulemkInitialResetis available for this purpose. It generates a reset which is
asserted at the start of simulation. The reset is asserted for the number of cycles specified by the
parametercycles, counting the start of time as 1 cycle. Therefore, acyclesvalue of 1 will cause
the reset to turn off at the first clock tick. This module is not synthesizable.

```
mkInitialReset Generates a reset forcyclescycles, where thecyclesparameter must
be greater than zero. Theclocked_byclause indicates the clock the
reset is associated with. This module is not synthesizable.
module mkInitialReset #( Integer cycles )
( Reset ) ;
```
Example:

```
Clock c <- mkAbsoluteClock (10, 5);
// a reset associated with clock c:
Reset r <- mkInitialReset (2, clocked_by c);
```
When two reset signals need to be combined so that some logic can be reset when either input reset
is asserted, themkResetEithermodule can be used.

```
mkResetEither Generates a reset which is asserted whenever either input reset is as-
serted.
```
```
module mkResetEither ( Reset aRst,
Reset bRst)
( Reset out_ifc );
```

Bluespec SystemVerilog Reference Guide

```
Figure 26: Reset Either
```
Example:

```
Reset r <- mkResetEither(rst1, rst2);
```
```
mkResetInverter Generates an inverted Reset.
```
```
module mkResetInverter#(Reset in)
(Reset);
```
```
isResetAsserted Tests whether a Reset is asserted, providing a Boolean value in the
clock domain associated with the Reset.
```
```
module isResetAsserted( ReadOnly#(Bool) ifc ) ;
```
Verilog Modules

The BSV modules correspond to the following Verilog modules, which are found in the Bluespec
Verilog library,$BLUESPECDIR/Verilog/.

```
BSV Module Name Verilog Module Name Comments
```
```
mkASyncReset SyncReset0.v when stages==0
mkASyncResetFromCR SyncResetA.v
mkSyncReset SyncReset0.v when stages==0
mkSyncResetFromCR SyncReset.v
mkReset MakeReset0.v when stages==0
MakeResetA.v instantiatesSyncResetA
mkResetSync MakeReset0.v when stages==0
MakeReset.v instantiatesSyncReset
mkResetMux ResetMux.v
mkResetEither ResetEither.v
mkResetInverter ResetInverter.v
isResetAsserted ResetToBool.v
```
### C.10 Special Collections

#### C.10.1 ModuleContext

Package


Reference Guide Bluespec SystemVerilog

import ModuleContext :: * ;

Description

An ordinary Bluespec module, when instantiated, adds state elements and rules to the growing
accumulation of elements and rules already in the design. In some designs, items other than state
elements and rules must be accumulated as well. While there is a need to add these items, it is also
desirable to keep these additional design details separate from the main design, keeping the natural
structure of the design intact.

TheModuleContextpackage provides the capability of accumulating items and maintaining the
compile-time state of additional items, in such a way that it doesn’t change the structure of the
original design.

TheModuleContextmechanism allows the designer tohidethe details of the additional interfaces.
Before the module can be synthesized, it must be converted (orexposed) into a module containing
only rules and state elements, as the compiler does not know how to handle the other items. The
ModuleContextpackage provides the mechanisms to allow additional items to be collected, processed,
and exposed.

This package is provided as both a compiled library package and as BSV source code to facilitate
customization. The source code file can be found in the$BLUESPECDIR/BSVSource/Contextsdi-
rectory. To customize a package, copy the file into a local directory and then include the local
directory in the path when compiling. This is done by specifying the search path with the-poption
as described in the BSV Users Guide.

Types and Type Classes

The default BSV module type isModule, but you can define other BSV module types as well. The
ModuleContexttype is a variation on theModuletype that allows additional items, other than states
and rules, to be collected while elaborating the module structure.

TheModuleContextpackage defines the typeclassContext, which includes functionsgetContext
andputContext. AContexttypeclass has two type parameters: a module type (mc1) and the
context (c2).

```
typeclass Context#(type mc1, type c2);
module [mc1] getContext(c2) provisos (IsModule#(mc1, a));
module [mc1] putContext#(c2 s)(Empty) provisos (IsModule#(mc1, a));
endtypeclass
```
A regular module type (Module) will have a context ofvoid:

```
instance Context#(Module, void);
```
A module type ofModuleContextwill return the context of the module:

```
instance Context#(ModuleContext#(st1), st1);
```
An instance is defined where the context typest1of theModuleContextand the context typest2
are different, butGettable(as defined inHlistSection C.10.4):

```
instance Context#(ModuleContext#(st1), st2)
provisos (Gettable#(st1, st2));
```

Bluespec SystemVerilog Reference Guide

The modulesapplyToContextandapplyToContextMare used to apply a function over a context.
TheapplyToContextMmodules is used for monadic functions.

```
applyToContext Applies a function over a context.
```
```
module [mc1] applyToContext#(function c2 f(c2 c))(Empty)
provisos (IsModule#(mc1, a), Context#(mc1, c2));
```
```
applyToContextM Applies a monadic function over a context.
```
```
module [mc1] applyToContextM#(function module#(c2) m(c2 c))
(Empty)
provisos (IsModule#(mc1, a), Context#(mc1, c2));
```
ClockContext

The structureClockContextis defined to be comprised of two clocks:clk1andclk2and two resets:
rst1andrst2.

```
typedef struct {
Clock clk1;
Clock clk2;
Reset rst1;
Reset rst2;
} ClockContext;
```
AninitClockContextis defined with the values of both clocks set tonoClockand both resets set
tonoReset:

```
ClockContext initClockContext = ClockContext {
clk1: noClock, clk2: noClock, rst1: noReset, rst2: noReset };
```
Expose

TheExposetypeclass converts a context to an interface for a synthesis boundary, converting it
to a module type of Module. TheExposetypeclass provides the modulesunburyContextand
unburyContextWithClocks.

```
typeclass Expose#(type c, type ifc)
dependencies (c determines ifc);
```
AnHListof contexts is convertible if its elements are, and results in aTupleof subinterfaces.

```
instance Expose#(HList1#(ct1), ifc1)
provisos (Expose#(ct1,ifc1));
```
```
instance Expose#(HCons#(c1,c2), Tuple2#(ifc1,ifc2))
provisos (Expose#(c1,ifc1), Expose#(c2,ifc2));
```
```
instance Expose#(ClockContext, Empty);
```

Reference Guide Bluespec SystemVerilog

TheunburyContextmodule is for use at the top level of a module to be separately synthesized. It
takes as an argument a module which is to be instantiated in a particular context, and an initial
state for that context. The module is instantiated, and the final context converted into an extra
interface, returned in pair with the intantiated module’s own interface.

```
unburyContext Converts a context to an interface for a synthesis boundary. AnHListof
contexts is convertible if its elements are, and results in a tuple of subinter-
faces.
```
```
module unburyContext#(c x)(ifc);
```
```
module unburyContext#(HList1#(ct1) c1)(ifc1);
```
```
module unburyContext#(HCons#(c1,c2) c12)(Tuple2#(ifc1,ifc2));
```
```
module unburyContext#(ClockContext x)();
```
TheunburyContextWithClockstakes aClockContextalong with theContextit is specifically
handling

```
unburyContextWithClocks Converts a context to an interface for a synthesis boundary and takes
a ClockContext as a second argument.
```
```
module unburyContextWithClocks#(c x, ClockContext cc)
(ifc);
```
```
module unburyContextWithClocks#(HList1#(ct1) c1,
ClockContext cc)(ifc1);
```
```
module unburyContextWithClocks#(HCons#(c1,c2) c12,
ClockContext cc)
(Tuple2#(ifc1,ifc2));
```
```
module unburyContextWithClocks#(ClockContext x,
ClockContext cc)();
```
Hide

TheHidetypeclass provides the modulereburyContext, which takes an interface as an argument
(and provides an Empty interface). It is intended to be run in a context which can absorb the
information from the interface. As withExpose, aTupleof interfaces can be hidden if each element
can be hidden.

```
reburyContext Connects the provided interface with the surrounding context.
```
```
module [mc] reburyContext#(ifc i)(Empty);
```
```
module [mc] reburyContext#(Empty i)(Empty);
```
```
module [mc] reburyContext#(Tuple2#(ifc1,ifc2) i12)(Empty);
```
ContextRun


Bluespec SystemVerilog Reference Guide

TheContextRunandContextsRuntypeclasses provides modules to run modules in contexts. The
modulerunWithContextruns a module with an entirely new context.

typeclass ContextRun#(type m, type c1, type ctx2)
dependencies ((m, c1) determines ctx2);

typeclass ContextsRun#(type m, type c1, type ctx2)
dependencies ((m, c1) determines ctx2);

```
runWithContext Runs a module with an entirely new context.
```
```
module [m] runWithContext #(c1 initState,
ModuleContext#(ctx2, ifcType) mkI)
(Tuple2#(c1, ifcType));
```
```
module [ModuleContext#(ctx)] runWithContext#(c1 initState,
ModuleContext#(HCons#(c1, ctx), ifcType) mkI)
(Tuple2#(c1, ifcType));
```
```
module [Module] runWithContext#(c1 initState,
ModuleContext#(c1,ifcType) mkI)
(Tuple2#(c1, ifcType));
```
```
runWithContexts Runs a module with an entirely new context.
```
```
module [m] runWithContexts#(c1 initState,
ModuleContext#(ctx2, ifcType) mkI)
(Tuple2#(c1, ifcType));
```
```
module [ModuleContext#(ctx)] runWithContexts#(c1 initState,
ModuleContext#(ctx2, ifcType) mkI)
(Tuple2#(c1, ifcType));
```
```
module [Module] runWithContexts#(c1 initState,
ModuleContext#(c1, ifcType) mkI)
(Tuple2#(c1, ifcType));
```
Contexts.defines

Bluespec provides macros in theContext.definesfile to handle the treatment of the module con-
texts at synthesis boundaries.

1. The designer defines a leaf or intermediate node module, with module type[ErrorReporter]
    or[ErrorReporterA], appending a 0 to its name (e.g.mkM0). Elsewhere in the package the
    appropriate macro is chosen from the macrosSynthBoundaryandSynthBoundaryWithClocks.
2. The macro defines a synthesizable version of the module,mkMV, which provides the original
    interface together with an error-reporting subinterface. It also defines a module with the origi-
    nal namemkMto be used for instantiating the original module. It uses the Context mechanism
    to re-bury the error-reporting plumbing and returns the original interface of the originalmkM)
    module


Reference Guide Bluespec SystemVerilog

These macros assume that the complete module context (such as anHListof individual contexts) is
namedCompleteContextand that its initial value may be obtained from eithermkInitialCompleteContext
ormkInitialCompleteContextWithClocks.

Example Without Clocks

SynthBoundary(mkM,IM)

Becomes

(*synthesize*)
module [Module] mkMV(Tuple2#(CompleteContextIfc,IM));
let init <- mkInitialCompleteContext;
let _ifc <- unbury(init, mkM0);
return _ifc;
endmodule

module [ModuleContext#(CompleteContext)] mkM(IM);
let _ifc <- rebury(mkMV);
return _ifc;
endmodule

Example With Clocks

SynthBoundaryWithClocks(mkM,IM)

Becomes

(*synthesize*)
module [Module] mkMV#(Clock c1,Reset r1,Clock c2,Reset r2)(Tuple2#(CompleteContextIfc,IM));
let init <- mkInitialCompleteContextWithClocks(c1, r1, c2, r2);
let _ifc <- unburyWithClocks(initialCompleteContext, c1, r1, c2, r2, mkM0);
return _ifc;
endmodule

module [ModuleContext#(CompleteContext)] mkM(IM);
let _ifc <- reburyWithClocks(mkMV);
return _ifc;
endmodule

#### C.10.2 ModuleCollect

Package

import ModuleCollect :: * ;

Description

TheModuleCollectpackage provides the capability of adding additional items, such as configuration
bus connections, to a design in such a way that it does not change the structure of the design. This
section provides a brief overview of the package. For a more detailed description of its usage, see the
CBuspackage (C.10.3), which utilizesModuleCollect. There is also a detailed example and more
complete discussion of theCBuspackage in the configbus tutorial in the BSV/tutorials directory.


Bluespec SystemVerilog Reference Guide

An ordinary Bluespec module, when instantiated, adds its own state elements and rules to the grow-
ing accumulation of state elements and rules defined in the design. In some designs, for example a
configuration bus, additional items, such as the logic for the bus address decoding must be accumu-
lated as well. While there is a need to add these items, it is also desirable to keep these additional
design details separate from the main design, keeping the natural structure of the design intact.

TheModuleCollectmechanism allows the designer tohidethe details of the additional interfaces. A
module which is going to be synthesized must contain only rules and state elements, as the compiler
does not know how to handle the additional items. Therefore, the collection must be brought into
the open, or exposed, before the module can be synthesized. TheModuleCollectpackage provides
the mechanisms to allow these additional items to be collected, processed and exposed.

This package is provided as both a compiled library package and as BSV source code to facilitate
customization. The source code file can be found in the$BLUESPECDIR/BSVSource/Contextsdi-
rectory. To customize a package, copy the file into a local directory and then include the local
directory in the path when compiling. This is done by specifying the search path with the-poption
as described in the BSV Users Guide.

Types and Type Classes

TheModuleCollecttype is a variation on theModuletype that allows additional items, other than
states and rules, to be collected while elaborating the module structure. A module defining the
accumulation of a special collection will have the type ofModuleCollectwhich is defined as a type
ofModuleContext(Section C.10.1):

typedef ModuleContext#(HList1#(UAList#(a))) ModuleCollect#(type a_type);

wherea_typedefines the type of the items being collected. The collection is kept as anHList,
therefore each item in the collection does not have the same type.

Your new type of module is aModuleCollectdefined to collect a specific type. It is often convenient
to give a name to your new type of module using thetypedefkeyword.

For example:

```
typedef ModuleCollect#(element_type, ifc_device)
MyModuleType#(type ifc_device)
```
specifies a type namedMyModuleType.

An ordinary module, one defined with the keywordmodulewithout a type in square brackets im-
mediately after it, can be of any module type. It is polymorphic, and when instantiated takes the
type of the surrounding module context. Only modules of typeModulecan be synthesized, so the
*synthesize*attribute forces the type to beModule. This is equivalent to writing:

```
module [Module]...
```
Normally, all the modules instantiated inside a synthesized module take the typeModule.

A module which is accumulating a collection must have the appropriate type, specified in square
brackets immedately after the keyword, as shown in the following example:

```
module [AssertModule] mkAssertionReg...
```
The complete example is found later in this section. This implies that any module instantiating
mkAssertionRegis no longer polymorphic, its type is constrained by the inner module, so it will
have to be explicitly given theAssertModuletype too. Note, however, that you can continue
to instantiate other modules not concerned with the collection (for example,mkReg,mkFIFO, etc.)
alongsidemkAssertRegjust as before. But now they will take the typeAssertModulefrom the
context instead of the typeModule.


Reference Guide Bluespec SystemVerilog

Since only modules of typeModulecan be synthesized, before this group ofAssertModuleinstanti-
ations can be synthesized, you must useexposeCollectionto contain the collection in a top-level
module of typeModule.

Interfaces

TheIWithCollectioninterface couples the normal module interface (thedeviceinterface) with
the collection of collected items (thecollectioninterface). This is the interface provided by the
exposeCollectionfunction. It separates the collection list and the device module interface, to allow
the module to be synthesized.

```
interface IWithCollection #(type a, type i);
method i device();
method List#(a) collection();
endinterface: IWithCollection
```
OLD:

```
interface IWithCollection #(type collection_type, type item_type);
interface item_type device();
interface List#(collection_type) collection();
endinterface: IWithCollection
```
Modules and Functions

In the course of evaluating a module body during its instantiation, an item may be added to the
current collection by using the functionaddToCollection.

```
addToCollection Adds an item to the collection.
```
```
function ModuleCollect#(a_type, ifc)
addToCollection(a_type item);
```
Once a set of items has been collected, those items must be exposed before synthesis. TheexposeCollection
module constructor is used to bring the collection out into the open. TheexposeCollection
module takes as an argument aModuleCollectmodule (m) with interfaceifc, and provides an
IWithCollectioninterface.

```
exposeCollection Expose the collection to allow the module to be synthesized.
```
```
module exposeCollection#(ModuleCollect#(a_type, ifc) m)
(IWithCollection#(a_type, ifc));
```
Finally, theModuleCollectpackage provides a function,mapCollection, to apply a function to
each item in the current collection.

```
mapCollection Apply a function to each item added to the collection within the second
argument.
```
```
function ModuleCollect#(a_type, ifc)
mapCollection(function a_type x1(a_type x1),
ModuleCollect#(a_type, ifc) x2);
```
Example - Assertion Wires


Bluespec SystemVerilog Reference Guide

// This example shows excerpts of a design which places various
// test conditions (Boolean expressions) at random places in a design,
// and lights an LED (setting an external wire to 1), if the condition
// is ever satisfied.

import ModuleCollect::*;
import List::*;
import Vector::*;
import Assert::*;

// The desired interface at the top level is:
interface AssertionWires#(type n);
method Bit#(n) wires;
method Action clear;
endinterface

// The "wires" method tells which conditions have been set, and the
// "clear" method resets them all to 0.
// The items in our extra collection will be interfaces of the
// following type:

interface AssertionWire;
method Integer index; //Indicates which wire is to be set if
method Bool fail; // fail method ever returns true.
method Action clear;
endinterface

// We next define the "AssertModule" type. This is to behave like an
// ordinary module providing an interface of type "i", except that it
// also can collect items of type "AssertionWire":

typedef ModuleCollect#(AssertionWire, i) AssertModule#(type i);

typedef Tuple2#(AssertionWires#(n), i) AssertIfc#(type i, type n);

...

// The next definition shows how items are added to the collection.
// This is the module which will be instantiated at various places in
// the design, to test various conditions. It takes one static
// parameter, "ix", to specify which wire is to carry this condition,
// and one dynamic parameter (one varying at run-time) "c", giving the
// value of the condition itself.

interface AssertionReg;
method Action set;
method Action clear;
endinterface

module [AssertModule] mkAssertionReg#(Integer ix)(AssertionReg);

```
Reg#(Bool) cond <- mkReg(False);
```
```
// an item is defined and added to the collection
```

Reference Guide Bluespec SystemVerilog

```
let item = (interface AssertionWire;
method index;
return (ix);
endmethod
method fail;
return(cond);
endmethod
method Action clear;
cond <= False;
endmethod
endinterface);
addToCollection(item);
```
endmodule

// the collection must be exposed before synthesis
module [Module] exposeAssertionWires#(AssertModule#(i) mkI)(AssertIfc#(i, n));

```
IWithCollection#(AssertionWire, i) ecs <- exposeCollection(mkI);
```
```
...(c_ifc is created from the list ecs.collection)
```
```
// deliver the array of values in the registers
let dut_ifc = ecs.device;
```
// return the values in the collection, and the ifc of the device
return(tuple2(c_ifc, dut_ifc));
endmodule

#### C.10.3 CBus

Package

import CBus :: * ;

Description

TheCBuspackage provides the interface, types and modules to implement a configuration bus
capability providing access to the control and status registers in a given module hierarchy. This
package utilizes theModuleCollectpackage and functionality, as described in section C.10.2. The
ModuleCollectpackage allows items in addition to usual state elements and rules to be accumulated.
This is required to collect up the interfaces of the control status registers included in a module and
to add the associated logic and ports required to allow them to be accessed via a configuration bus.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

For a more complete discussion of theCBuspackage, consult the configbus tutorial in the BSV/tutorials
directory.

Types and Type Classes

The typeCBusItemdefines the type of item to be collected byModuleCollect. The items to be
collected are the same as the ifc which we will later expose, so we use a type alias:


Bluespec SystemVerilog Reference Guide

typedef CBus#(size_address, size_data)
CBusItem #(type size_address, type size_data);

The typeModWithCBusdefines the type of module which is collectingCBusItems. An ordinary
module, one not collecting anything other than state elements and rules, has the typeModule. Since
CBusItems are being collected, a module typeModWithCBusis defined. When the module type is
notModule, the type must be specified in square brackets immediately after themodulekeyword in
the module definition.
typedef ModuleCollect#(CBusItem#(size_address, size_data), item)
ModWithCBus#(type size_address, type size_data, type item);

Interface and Methods

TheCBusinterface providesreadandwritemethods to access control status registers. It is poly-
morphic in terms of the size of the address bus (size_address) and size of the data bus (size_data).

```
CBusInterface
Name Description
```
```
write Writes thedatavalue to the register if and only if the value of
addrmatches the address of the register.
```
```
read Returns the value of the associated register if and only ifaddr
matches the register address. In all other cases thereadmethod
returns anInvalidvalue.
```
interface CBus#(type size_address, type size_data);
method Action write(Bit#(size_address) addr, Bit#(size_data) data);
(* always_ready *)
method ActionValue#(Bit#(size_data)) read(Bit#(size_address) addr);
endinterface

TheIWithCBusinterface combines theCBusinterface with a normal module interface. It is defined as
a structured interface with two subinterfaces:cbus_ifc(the associated configuration bus interface)
anddevice_ifc(the associated device interface). It is polymorphic in terms of the type of the
configuation bus interface and the type of the device interface.
interface IWithCBus#(type cbus_IFC, type device_IFC);
interface cbus_IFC cbus_ifc;
interface device_IFC device_ifc;
endinterface

Modules

ThecollectCBusIFCmodule takes as an argument a module with anIWithCBusinterface, adds the
associatedCBusinterface to the current collection (usingaddToCollectionfrom theModuleCollect
package), and returns a module with the normal interface. Note thatcollectCBusIFCis of module
typeModWithCBus.

```
collectCBusIFC Adds theCBusto the collection and returns a module with just the device
interface.
```
```
module [ModWithCBus#(size_address, size_data)]
collectCBusIFC#(Module#(IWithCBus#(
CBus#(size_address,size_data),i)) m)(i);
```

Reference Guide Bluespec SystemVerilog

TheexposeCBusIFCmodule is used to create anIWithCBusinterface given a module with a normal
interface and an associated collection ofCBusItems. This module takes as an argument a module
of typeModWithCBusand provides an interface of typeIWithCBus. TheexposeCBusIFCmodule
exposes the collectedCBusItems, processes them, and provides a new combined interface. This
module is synthesizable, because it is of typeModule.

```
exposeCBusIFC A module wrapper that takes a module with a normal interface, processes the
collected CBusItems and provides an IWithCBus interface.
```
```
module [Module] exposeCBusIFC#(ModWithCBus#(
size_address, size_data, item) sm)
(IWithCBus#(CBus#(size_address, size_data), item));
```
TheCBuspackage provides a set of module primitives each of which adds aCBusinterface to the
collection and provides a normalReginterface from the local block point of view. These modules are
used in designs where a normal register would be used, and can be read and written to as registers
from within the design.

```
mkCBRegR A wrapper to provide a read onlyCBusinterface to the collection and a normal
Reginterface to the local block.
```
```
module [ModWithCBus#(size_address, size_data)]
mkCBRegR#(CRAddr#(size_address2) addr, r x)
(Reg#(r))
provisos (Bits#(r, sr), Add#(k, sr, size_data),
Add#(ignore, size_address2, size_address));
```
```
mkCBRegRW A wrapper to provide a read/writeCBusinterface to the collection and a
normalReginterface to the local block.
```
```
module [ModWithCBus#(size_address, size_data)]
mkCBRegRW#(CRAddr#(size_address2) addr, r x)
(Reg#(r))
provisos (Bits#(r, sr), Add#(k, sr, size_data),
Add#(ignore, size_address2, size_address));
```
```
mkCBRegW A wrapper to provide a write onlyCBusinterface to the collection and a
normalReginterface to the local block.
```
```
module [ModWithCBus#(size_address, size_data)]
mkCBRegW#(CRAddr#(size_address2) addr, r x)
(Reg#(r))
provisos (Bits#(r, sr), Add#(k, sr, size_data),
Add#(ignore, size_address2, size_address));
```

Bluespec SystemVerilog Reference Guide

```
mkCBRegRC A wrapper to provide a read/clearCBusinterface to the collection and a
normalReginterface to the local block. This register can read from the config
bus but the write is clear mode; for each write bit a 1 means clear, while a 0
means don’t clear.
module [ModWithCBus#(size_address, size_data)]
mkCBRegRC#(CRAddr#(size_address2) addr, r x)
(Reg#(r))
provisos (Bits#(r, sr), Add#(k, sr, size_data),
Add#(ignore, size_address2, size_address));
```
ThemkCBRegFilemodule wrapper adds aCBusinterface to the collection and provides aRegFile
interface to the design. This module is used in designs as a normalRegFilewould be used.

```
mkCBRegFile A wrapper to provide a normalRegFileinterface and automatically add the
CBusinterface to the collection.
```
```
module [ModWithCBus#(size_address, size_data)]
mkCBRegFile#(Bit#(size_address) reg_addr,
Bit#(size_address) size)
(RegFile#(Bit#(size_address), r))
provisos (Bits#(r, sr), Add#(k, sr, size_data));
```
Example

Provided here is a simple example of a CBus implementation. The example is comprised of three
packages: CfgDefines,Block, andTb. TheCfgDefinespackage contains the definition for the
configuration bus,Blockis the design block, andTbis the testbench which executes the block.

TheBlockpackage contains the local design. As seen in Figure 27, the configuration bus registers
look like a single field from the CBus (cfgResetAddr, cfgStateAddr, cfgStatusAddr), while each
field (reset, init, cnt, etc.) in the configuration bus registers looks like a regular register from
from the local block point of view.

```
Figure 27: CBus Registers used in Block example
```
import CBus::*; // this is a Bluespec library


Reference Guide Bluespec SystemVerilog

import CfgDefines::*; // user defines - address,registers, etc

interface Block;
// TODO: normally this block would have at least a few methods
// Cbus interface is hidden, but it is there
endinterface

// In order to access the CBus at this parent, we need to expose the bus.
// Only modules of type [Module] can be synthesized.
module [Module] mkBlock(IWithCBus#(DCBus, Block));
let ifc <- exposeCBusIFC( mkBlockInternal );
return ifc;
endmodule

// Within this module the CBus looks like normal Registers.
// This module can’t be synthesized directly.
// How these registers are combined into CBus registers is
// defined in the CfgDefines package.

module [DModWithCBus] mkBlockInternal( Block );
// all registers are read/write from the local block point of view
// config register interface types can be
// mkCBRegR -> read only from config bus
// mkCBRegRW -> read/write from config bus
// mkCBRegW -> write only from config bus
// mkCBRegRC -> read from config bus, write is clear mode
// i.e. for each bit a 1 means clear, 0 means don’t clear
// reset bit is write only from config bus
// we presume that you use this bit to fire some local rules, etc
Reg#(TCfgReset) reg_reset_reset <- mkCBRegW(cfg_reset_reset, 0 /* init val */);

```
Reg#(TCfgInit) reg_setup_init <- mkCBRegRW(cfg_setup_init, 0 /* init val */);
Reg#(TCfgTz) reg_setup_tz <- mkCBRegRW(cfg_setup_tz, 0 /* init val */);
Reg#(TCfgCnt) reg_setup_cnt <- mkCBRegRW(cfg_setup_cnt, 1 /* init val */);
```
```
Reg#(TCfgOnes) reg_status_ones <- mkCBRegRC(cfg_status_ones, 0 /* init val */);
Reg#(TCfgError) reg_status_error <- mkCBRegRC(cfg_status_error, 0 /* init val */);
```
```
// USER: you know have registers, so do whatever it is you do with registers :)
// for instance
rule bumpCounter ( reg_setup_cnt != unpack(’1) );
reg_setup_cnt <= reg_setup_cnt + 1;
endrule
```
rule watch4ones ( reg_setup_cnt == unpack(’1) );
reg_status_ones <= 1;
endrule
endmodule

TheCfgDefinespackage contains the user defines describing how the local registers are combined
into the configuration bus.

package CfgDefines;
import CBus::*;


Bluespec SystemVerilog Reference Guide

##### ////////////////////////////////////////////////////////////////////////////////

/// basic defines
////////////////////////////////////////////////////////////////////////////////
// width of the address bus, it’s easiest to use only the width of the bits needed
// but you may have other reasons for passing more bits around (even if some address
// bits are always 0)
typedef 2 DCBusAddrWidth; // roof( log2( number_of_config_registers ) )

// the data bus width is probably defined in your spec
typedef 32 DCBusDataWidth; // how wide is the data bus

////////////////////////////////////////////////////////////////////////////////
// Define the CBus
////////////////////////////////////////////////////////////////////////////////
typedef CBus#( DCBusAddrWidth,DCBusDataWidth) DCBus;
typedef CRAddr#(DCBusAddrWidth,DCBusDataWidth) DCAddr;
typedef ModWithCBus#(DCBusAddrWidth, DCBusDataWidth, i) DModWithCBus#(type i);

////////////////////////////////////////////////////////////////////////////////
/// Configuration Register Types
////////////////////////////////////////////////////////////////////////////////
// these are configuration register from your design. The basic
// idea is that you want to define types for each individual field
// and later on we specify which address and what offset bits these
// go to. This means that config register address fields can
// actually be split across modules if need be.
//
typedef bit TCfgReset;

typedef Bit#(4) TCfgInit;
typedef Bit#(6) TCfgTz;
typedef UInt#(8) TCfgCnt;

typedef bit TCfgOnes;
typedef bit TCfgError;

////////////////////////////////////////////////////////////////////////////////
/// configuration bus addresses
////////////////////////////////////////////////////////////////////////////////
Bit#(DCBusAddrWidth) cfgResetAddr = 0; //
Bit#(DCBusAddrWidth) cfgStateAddr = 1; //
Bit#(DCBusAddrWidth) cfgStatusAddr = 2; // maybe you really want this to be 0,4,8 ???

////////////////////////////////////////////////////////////////////////////////
/// Configuration Register Locations
////////////////////////////////////////////////////////////////////////////////
// DCAddr is a structure with two fields
// DCBusAddrWidth a ; // this is the address
// // this does a pure comparison
// Bit#(n) o ; // this is the offset that this register
// // starts reading and writting at

DCAddr cfg_reset_reset = DCAddr {a: cfgResetAddr, o: 0}; // bits 0:0


Reference Guide Bluespec SystemVerilog

DCAddr cfg_setup_init = DCAddr {a: cfgStateAddr, o: 0}; // bits 0:0
DCAddr cfg_setup_tz = DCAddr {a: cfgStateAddr, o: 4}; // bits 9:4
DCAddr cfg_setup_cnt = DCAddr {a: cfgStateAddr, o: 16}; // bits 24:16

DCAddr cfg_status_ones = DCAddr {a: cfgStatusAddr, o: 0}; // bits 0:0
DCAddr cfg_status_error = DCAddr {a: cfgStatusAddr, o: 0}; // bits 1:1

////////////////////////////////////////////////////////////////////////////////
///
////////////////////////////////////////////////////////////////////////////////
endpackage

TheTbpackage executes the block.

import CBus::*; // bluespec library
import CfgDefines::*; // address defines, etc
import Block::*; // test block with cfg bus
import StmtFSM::*; // just for creating a test sequence

(* synthesize *)
module mkTb ();
// In order to access this cfg bus we need to use IWithCBus type
IWithCBus#(DCBus,Block) dut <- mkBlock;

```
Stmt test =
seq
// write the bits need to the proper address
// generally this comes from software or some other packing scheme
// you can, of course, create functions to pack up several fields
// and drive that to bits of the correct width
// For that matter, you could have your own shadow config registers
// up here in the testbench to do the packing and unpacking for you
dut.cbus_ifc.write( cfgResetAddr, unpack(’1) );
```
```
// put some ones in the status bits
dut.cbus_ifc.write( cfgStateAddr, unpack(’1) );
```
```
// show that only the valid bits get written
$display("TOP: state = %x at ", dut.cbus_ifc.read( cfgStateAddr ), $time);
```
```
// clear out the bits
dut.cbus_ifc.write( cfgStateAddr, 0 );
```
```
// but the ’ones’ bit was set when it saw all ones on the count
// so read it to see that...
$display("TOP: status = %x at ", dut.cbus_ifc.read( cfgStatusAddr ), $time);
```
```
// now clear it
dut.cbus_ifc.write( cfgStatusAddr, 1 );
```
```
// see that it’s clear
$display("TOP: status = %x at ", dut.cbus_ifc.read( cfgStatusAddr ), $time);
```

Bluespec SystemVerilog Reference Guide

// and if we had other interface methods, that where not part of CBUS
// we would access them via dut.device_ifc
endseq;
mkAutoFSM( test );
endmodule

#### C.10.4 HList

Package

import HList :: * ;

Description

TheHListpackage defines a datatypeHListwhich stores a list of data of different types. The
package also provides typeclasses and functions to perform various list operations on theHList
type.

The primitive data structures for anHListareHNiland the polymorphicHCons. The various
functions are provided by typeclasses, one for each function.

The package defines a typeclassGettablefor finding (getIt) and replacing (putIt) items in an
HList. This requires that all the items in theHListare different types. If two types are the same,
they must be disambiguated by encapsulating at least one of them (but preferably each of them) in
a new struct type. The functions of theGettabletypeclass require that theHListbe flat (no nested
HLists) and well-formed (terminating inHNil). That is, the target of a recursive search must be
either the completehHeador found within thehTail.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

TheHListpackages defines a typeclassHList:

typeclass HList#(type l);

TheHNildatatype defines a nil instance, the empty set. AnHListis usually terminated by aHNil.

typedef struct {} HNil deriving (Eq);

TheHConsdatatype is a structure with two members, a head of datatypeeand a tail of datatypel.

typedef struct {
e hd;
l tl;
} HCons#(type e, type l) deriving (Eq);

Functions

The various functions for heterogenous lists are provided by typeclasses, one for each functions.


Reference Guide Bluespec SystemVerilog

```
HHead Returns the first element of the list.
```
```
typeclass HHead#(type l, type h)
dependencies (l determines h);
function h hHead(l x);
endtypeclass
```
```
instance HHead#(HCons#(e, l), e);
```
```
HTail Returns the tail element from the list.
```
```
typeclass HTail#(type l, type lt)
dependencies (l determines lt);
function lt hTail(l xs);
endtypeclass
```
```
instance HTail#(HCons#(e, l), l);
```
```
HLength Returns a numeric value with the length of the list. For aHNil, will return
0.
```
```
typeclass HLength#(type l, numeric type n);
endtypeclass
```
```
instance HLength#(HNil, 0);
```
```
instance HLength#(HCons#(e, l), nPlus1)
provisos (HLength#(l, n), Add#(n,1,nPlus1));
```
```
HAppend Appends two lists, returning the combined list. The elements do not have
to be of the same data type. The combined list will be of typel2, and will
contain all the elements ofxsfollowed in order by all the elements ofys.
```
```
typeclass HAppend#(type l, type l1, type l2)
dependencies ((l, l1) determines l2);
function l2 hAppend(l xs, l1 ys);
```
```
instance HAppend#(HNil, l, l);
```
```
instance HAppend#(HCons#(e, l), l1, HCons#(e, l2))
provisos (HList#(l), HAppend#(l, l1, l2));
```

Bluespec SystemVerilog Reference Guide

```
HSplit ThehSplitfunction takes anHListof typeland returns aTuple2of two
HLists. This function is the inverse ofhAppend.
```
```
typeclass HSplit#(type l, type l1, type l2);
function Tuple2#(l1,l2) hSplit(l xs);
endtypeclass
```
```
instance HSplit#(HNil, HNil, HNil);
```
```
instance HSplit#(l, HNil, l);
```
```
instance HSplit#(HCons#(hd,tl), HCons#(hd,l3), l2)
provisos (HSplit#(tl,l3,l2));
```
```
Gettable This typeclass is for finding (getIt) and replacing (putIt) a particular
element in anHList. All items in theHListmust be of different types. If
two types are the same, they should be disambiguated by encapsulating at
least one of them (and preferably both of them) in a new struct type.
```
```
typeclass Gettable#(type c1, type c2);
function c2 getIt(c1 x);
function c1 putIt(c1 x, c2 y);
endtypeclass
```
```
instance Gettable#(HCons#(t1, t2), t1);
```
```
instance Gettable#(HCons#(t1, t2), t3)
provisos (Gettable#(t2, t3));
```
Small Lists

TheHListpackcage provides type definitions for small lists, ranging from 1 element to 8 elements,
along with constructor functions to build the lists.

HList1

typedef HCons#(t, HNil)
HList1#(type t);

function HList1#(t1) hList1(t1 x1) = hCons(x1, hNil);

HList2

typedef HCons#(t1, HCons#(t2, HNil))
HList2#(type t1, type t2);

function HList2#(t1, t2) hList2(t1 x1, t2 x2) = hCons(x1, hCons(x2, hNil));

HList3

typedef HCons#(t1, HCons#(t2, HCons#(t3, HNil)))
HList3#(type t1, type t2, type t3);


Reference Guide Bluespec SystemVerilog

function HList3#(t1, t2, t3) hList3(t1 x1, t2 x2, t3 x3)
= hCons(x1, hCons(x2, hCons(x3, hNil)));

HList4

typedef HCons#(t1, HCons#(t2, HCons#(t3, HCons#(t4, HNil))))
HList4#(type t1, type t2, type t3, type t4);

function HList4#(t1, t2, t3, t4) hList4(t1 x1, t2 x2, t3 x3, t4 x4)
= hCons(x1, hCons(x2, hCons(x3, hCons(x4, hNil))));

HList5

typedef HCons#(t1, HCons#(t2, HCons#(t3, HCons#(t4, HCons#(t5, HNil)))))
HList5#(type t1, type t2, type t3, type t4, type t5);

function HList5#(t1, t2, t3, t4, t5) hList5(t1 x1, t2 x2, t3 x3, t4 x4, t5 x5)
= hCons(x1, hCons(x2, hCons(x3, hCons(x4, hCons(x5, hNil)))));

HList6

typedef HCons#(t1, HCons#(t2, HCons#(t3, HCons#(t4, HCons#(t5, HCons#(t6, HNil))))))
HList6#(type t1, type t2, type t3, type t4, type t5, type t6);

function HList6#(t1, t2, t3, t4, t5, t6)
hList6(t1 x1, t2 x2, t3 x3, t4 x4, t5 x5, t6 x6)
= hCons(x1, hCons(x2, hCons(x3, hCons(x4, hCons(x5, hCons(x6, hNil))))));

HList7

typedef HCons#(t1, HCons#(t2, HCons#(t3, HCons#(t4, HCons#(t5,
HCons#(t6, HCons#(t7, HNil)))))))
HList7#(type t1, type t2, type t3, type t4, type t5, type t6, type t7);

function HList7#(t1, t2, t3, t4, t5, t6, t7)
hList7(t1 x1, t2 x2, t3 x3, t4 x4, t5 x5, t6 x6, t7 x7)
= hCons(x1, hCons(x2, hCons(x3, hCons(x4, hCons(x5, hCons(x6, hCons(x7, hNil)))))));

HList8

typedef HCons#(t1, HCons#(t2, HCons#(t3, HCons#(t4, HCons#(t5,
HCons#(t6, HCons#(t7, HCons#(t8, HNil))))))))
HList8#(type t1, type t2, type t3, type t4, type t5, type t6, type t7, type t8);

function HList8#(t1, t2, t3, t4, t5, t6, t7, t8)
hList8(t1 x1, t2 x2, t3 x3, t4 x4, t5 x5, t6 x6, t7 x7, t8 x8)
= hCons(x1, hCons(x2, hCons(x3, hCons(x4, hCons(x5, hCons(x6,
hCons(x7, hCons(x8, hNil))))))));


Bluespec SystemVerilog Reference Guide

#### C.10.5 UnitAppendList

Package

import UnitAppendList :: * ;

Description

This provides a representation of lists for which append(x,y) is O(1), rather than O(length(x)) as
in the normal representation; the downside is that there is no longer a unique representation for
a given list. These lists are useful for situations in which the list is constructed by recursively
amalgamating lists from sub-computations, and then subsequently processed. Functions formapand
mapMare provided for processing sublists during construction. For final processing it is almost always
preferable first to flatten the list (by a function also provided) into the conventional representation,
thus eliminating empty subtrees.

This package is provided as both a compiled library package and as BSV source code as documen-
tation. The source code file can be found in the$BLUESPECDIR/BSVSource/Miscdirectory. To
customize a package, copy the file into a local directory and then include the local directory in the
path when compiling. This is done by specifying the search path with the-poption as described in
the BSV Users Guide.

Types and type classes

TheUnitAppendListpackage defines the structureUAList:

typedef union tagged {
void NoItems;
a One;
Tuple2#(UAList#(a),UAList#(a)) Append;
} UAList#(type a);

UAListis a member of theDefaultValuetypeclass, which defines a default value for user defined
structures. The default value forUAListis defined as:

instance DefaultValue#(UAList#(a));
defaultValue = NoItems;
endinstance

Functions

```
flatten0 Given aUAList#(a)and aList#(a), returns a conventional list of typeList.
```
```
function List#(a) flatten0(UAList#(a) c, List#(a) xs);
```
```
flatten Converts a list of typeUAListinto a conventional list of typeList.
```
```
function List#(a) flatten(UAList#(a) c) = flatten0(c, Nil);
```
```
uaMap Maps a function of a list of typeUAList, returning aUAList.
```
```
function UAList#(b) uaMap(function b f(a x), UAList#(a) c);
```

Reference Guide Bluespec SystemVerilog

```
uaMapM Maps a monadic function of a list of typeUAList, returning aUAList.
```
```
module uaMapM#(function module#(b) f(a x), UAList#(a) c)(UAList#(b));
```