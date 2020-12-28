Bluespec SystemVerilog (BSV) is aimed at hardware designers who are using or expect to use Verilog
[IEE05], VHDL [IEE02], SystemVerilog [IEE13], or SystemC [IEE12] to design ASICs or FPGAs. It
is also aimed at people creating _synthesizable_ models, transactors, and verification components to
run on FPGA emulation platforms. BSV substantially extends the design subset of SystemVerilog,
including SystemVerilog types, modules, module instantiation, interfaces, interface instantiation,
parameterization, static elaboration, and “generate” elaboration. BSV can significantly improve the
hardware designer’s productivity with some key innovations:

- It expresses synthesizable behavior with _Rules_ instead of synchronous `always` blocks. Rules
    are powerful concepts for achieving _correct_ concurrency and eliminating race conditions. Each
    rule can be viewed as a declarative assertion expressing a potential _atomic_ state transition.
    Although rules are expressed in a modular fashion, a rule may span multiple modules, i.e., it
    can test and affect the state in multiple modules. Rules need not be disjoint, i.e., two rules
    can read and write common state elements. The BSV compiler produces efficient RTL code
    that manages all the potential interactions between rules by inserting appropriate arbitration
    and scheduling logic, logic that would otherwise have to be designed and coded manually. The
    atomicity of rules gives a scalable way to avoid unwanted concurrency (races) in large designs.
- It enables more powerful generate-like elaboration. This is made possible because in BSV,
    actions, rules, modules, interfaces and functions are all first-class objects. BSV also has more
    general type parameterization (polymorphism). These enable the designer to “compute with
    design fragments,” i.e., to reuse designs and to glue them together in much more flexible ways.
    This leads to much greater succinctness and correctness.
- It provides formal semantics, enabling formal verification and formal design-by-refinement.
    BSV rules are based on Term Rewriting Systems, a clean formalism supported by decades
    of theoretical research in the computer science community [Ter03]. This, together with a
    judicious choice of a design subset of SystemVerilog, makes programs in BSV amenable to
    formal reasoning.

This manual is meant to be a stand-alone reference for BSV, i.e., it fully describes the subset of
Verilog and SystemVerilog used in BSV. It is not intended to be a tutorial for the beginner. A reader
with a working knowledge of Verilog 1995 or Verilog 2001 should be able to read this manual easily.
Prior knowledge of SystemVerilog is not required.


### Meta Notation 

The grammar in this document is given using an extended BNF (Backus-Naur Form). Grammar alternatives
are separated by a vertical bar (“|”). Items enclosed in square brackets (“[ ]”) are optional. Items 
enclosed in curly braces (“{ }”) can be repeated zero or more times. 

Another BNF extension is parameterization. For example, a _moduleStmt_ can be a _moduleIf_, and an
_actionStmt_ can be an _actionIf_. A _moduleIf_ and an _actionIf_ are almost identical; the only 
difference is that the former can contain (recursively) _moduleStmt_s whereas the latter can contain
_actionStmt_s. Instead of tediously repeating the grammar for _moduleIf_ and _actionIf_, we parameterize
it by giving a single grammar for `<ctxt>If`, where `<ctxt>` is either _module_ or _action_. In the
productions for `<ctxt>If`, we call for `<ctxt>Stmt` which, therefore, either represents a _moduleStmt_
or an _actionStmt_, depending on the context in which it is used. 
