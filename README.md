# CBT1043
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. GitHub repos will be deleted and created during this period...

```
//***FILE1043 is from Edgar Hofmann and contains a REXX             *   FILE1043
//*           Preprocessor, which is integrated seamlessly into     *   FILE1043
//*           the TSO environment.                                  *   FILE1043
//*                                                                 *   FILE1043
//*           email:  sy011eho@gmail.com                            *   FILE1043
//*                                                                 *   FILE1043
//*     * Why a REXX-Preprocessor                                   *   FILE1043
//*                                                                 *   FILE1043
//*     The REXX-Standard does not define a concept how to deal     *   FILE1043
//*     with global variables.  External REXX-functions (or         *   FILE1043
//*     procedures) do not inherit access to a parent environment   *   FILE1043
//*     (read or update).  Retrieve and return of contents is only  *   FILE1043
//*     defined via passing arguments and passing a value at        *   FILE1043
//*     return.  But especially stem variables are not suitable     *   FILE1043
//*     for this approach.  The defined stack architecture, which   *   FILE1043
//*     may overcome limitations in this area is not well           *   FILE1043
//*     implemented in non z-Architecures and is not widely used.   *   FILE1043
//*                                                                 *   FILE1043
//*     There are possibilites to overcome this shortage:           *   FILE1043
//*                                                                 *   FILE1043
//*     *First*: Define your own Global Area where you exchange     *   FILE1043
//*     contents. For MVS/TSO some packages found on CBTTAPE which  *   FILE1043
//*     deal with this approach. But there is extra rexx code       *   FILE1043
//*     necessary to prepare and to retrieve variables and stems    *   FILE1043
//*     from the "external call" (note: in z/VM CMS PIPELINES       *   FILE1043
//*     provide access to different variable pools.  J.P. Hartmann  *   FILE1043
//*     created also an MVS/TSO port of his PIPELINES, but this     *   FILE1043
//*     port never gained comparable commonnes as in z/VM).         *   FILE1043
//*                                                                 *   FILE1043
//*     *Second*: Avoid external functions. Internal subroutines    *   FILE1043
//*     may share the variable pool of the caller. But there is a   *   FILE1043
//*     big disadvantage. The code of the subroutine needs to be    *   FILE1043
//*     in the same file, which makes maintenance of common code    *   FILE1043
//*     very cumbersome.                                            *   FILE1043
//*                                                                 *   FILE1043
//*     For non z-Architectures some implementations exist.         *   FILE1043
//*     Namely BREXX (V. Vlachoudis) implemented this concept.      *   FILE1043
//*     Before an exec is interpreted, files are included,          *   FILE1043
//*     containing internal subroutine code. With this the code     *   FILE1043
//*     of the internal subroutine is unique in a separate file     *   FILE1043
//*     and may be shared among different execs, thus eases         *   FILE1043
//*     maintenance of this procedures.                             *   FILE1043
//*                                                                 *   FILE1043
//*     * TSO/REXX Implementation of a REXX-Preprocessor            *   FILE1043
//*                                                                 *   FILE1043
//*     TSO related, REXX Procedures are mostly called implicitly   *   FILE1043
//*     in a TSO-Environment.  The provided TSO-Command processor   *   FILE1043
//*     is IRXEXEC. An idea may be to include IRXEXEC in a          *   FILE1043
//*     wrapper program. This is possible but has some negative     *   FILE1043
//*     side effects with the handling of the internal stuctures    *   FILE1043
//*     of the MVS-REXX environment and MVS system maintenance.     *   FILE1043
//*                                                                 *   FILE1043
//*     The correct way is to use the provided exit points, as      *   FILE1043
//*     documented.                                                 *   FILE1043
//*                                                                 *   FILE1043
//*     - IRXTSPRM: IRXEXECU IRXEXECT IRXEXECI                      *   FILE1043
//*        -- register here IRXINIT-Exit (intialize a new           *   FILE1043
//*           EXECBLK)                                              *   FILE1043
//*        -- register here IRXEXEC-Exit (modify Contents before    *   FILE1043
//*           IRXEXEC is called)                                    *   FILE1043
//*        -- register here IRXTERM-Exit (cleanup user data areas   *   FILE1043
//*           used for modification)                                *   FILE1043
//*                                                                 *   FILE1043
//*     -- IRXEXECI                                                 *   FILE1043
//*        --- get Storage for modified EXEC Contents               *   FILE1043
//*        --- get Storage for (necessary) modified table of        *   FILE1043
//*                                                                 *   FILE1043
//*     -- IRXEXECU                                                 *   FILE1043
//*        --- get Storage for modified EXEC Contents               *   FILE1043
//*        --- get Storage for (necessary) modified table of        *   FILE1043
//*            exec row addresses (IRXINSTB INStorageBlock).        *   FILE1043
//*        --- IKJFTSR: call the REXX-Preprocessor via TSO          *   FILE1043
//*            (DCJRXPP).                                           *   FILE1043
//*                                                                 *   FILE1043
//*     -- IRXEXECT                                                 *   FILE1043
//*        --- cleanup.                                             *   FILE1043
//*                                                                 *   FILE1043
//*     TSO/REXX uses subpool 78. Both IRXEXECI and IRXEXECU are    *   FILE1043
//*     needed for Storage-Mgmt.                                    *   FILE1043
//*                                                                 *   FILE1043
//*     -- DCJRXPP                                                  *   FILE1043
//*       TSO Command Processor DCJRXPP (written & compiled with    *   FILE1043
//*       JCC) is a sample user program which handles the contents  *   FILE1043
//*       of the IRXINSTB, modifying and adding sourcelines to the  *   FILE1043
//*       loaded (but before executed) exec. Similar to             *   FILE1043
//*       preprocessors in other programming languages I've         *   FILE1043
//*       implemented two directives:                               *   FILE1043
//*                                                                 *   FILE1043
//*     --- #MACRO                                                  *   FILE1043
//*       Include an external member on the DD SYSEXEC              *   FILE1043
//*       concatenation. DD RXSYSLIB is allowed as an alternative   *   FILE1043
//*       to keep macros and syslibs away from normal rexx code.    *   FILE1043
//*                                                                 *   FILE1043
//*       AT THE CURRENT LINE:  Insertion of contents of the        *   FILE1043
//*        member at this sourceline ** NO SYNTAX CHECK **.  You    *   FILE1043
//*        are responsible that syntax and logic of the main        *   FILE1043
//*       exec is maintaned after the insertion is done.            *   FILE1043
//*                                                                 *   FILE1043
//*       Arguments for the macro will be resolved. Starting in     *   FILE1043
//*       the #MACRO line after the first semicolon the values      *   FILE1043
//*       for POSITIONAL parameters, delimited each with            *   FILE1043
//*       semicolon too, will be resolved in order of their         *   FILE1043
//*       specification.  The positional parameter identifier in    *   FILE1043
//*       the macro are specified as &P0 to &Pn.                    *   FILE1043
//*                                                                 *   FILE1043
//*     --- #SYSLIB                                                 *   FILE1043
//*       Append an external member on the DD SYSEXEC               *   FILE1043
//*       concatenation. DD RXSYSLIB is allowed as an alternative   *   FILE1043
//*       to keep macros and syslibs away from normal rexx-code.    *   FILE1043
//*                                                                 *   FILE1043
//*       AT THE END of the orignal code: In my experience this     *   FILE1043
//*       is the more important directive, as it emulates a         *   FILE1043
//*       syslib environment. Only one argument is honored          *   FILE1043
//*       (membername). You are responsible for the results the     *   FILE1043
//*       REXX-Interpreter produces.                                *   FILE1043
//*                                                                 *   FILE1043
//*     There is no recursion provided in the DCJRXPP program.      *   FILE1043
//*     Only the first level will be expanded. As a prerequisite    *   FILE1043
//*     the keyword PREPROCESS needs to be specified in the first   *   FILE1043
//*     line of the exec.                                           *   FILE1043
//*                                                                 *   FILE1043
//*     * Hints, Warnings & Caveats                                 *   FILE1043
//*                                                                 *   FILE1043
//*      - as mentioned above PREROCESS keyword.                    *   FILE1043
//*                                                                 *   FILE1043
//*      - when specified #SYSLIB, check EXIT or RETURN before      *   FILE1043
//*        appended syslib code, to prevent interpretation run      *   FILE1043
//*        into this appended lines.                                *   FILE1043
//*                                                                 *   FILE1043
//*      - JCC and standard streams. JCC supports DD STDOUT and     *   FILE1043
//*        DD STDERR. DCJRXPP uses STDERR to print some             *   FILE1043
//*        Informations. In a last action DCJRXPP sends the         *   FILE1043
//*        complete expanded exec to DD STDOUT, which could be      *   FILE1043
//*        executed without modification. This is your door for     *   FILE1043
//*        trace & debug. There is no need to allocate both         *   FILE1043
//*        DDnames in normal situations. In foreground a TSO        *   FILE1043
//*        ALLOC DD(STDERR) DA(*) will do the job when needed,      *   FILE1043
//*        vice versa TSO FREE DD(STDERR) quits the output.         *   FILE1043
//*                                                                 *   FILE1043
//*      - Destroying your TSO/REXX Environment (and ISPF) is a     *   FILE1043
//*        bad thing. If you lock yourself and perhaps others out   *   FILE1043
//*        of the system, be prepared to have a backdoor.           *   FILE1043
//*                                                                 *   FILE1043
//*        -- pack your code into a STEPLIB for one of your LOGON   *   FILE1043
//*           Procedures and have a tested emergency LOGON          *   FILE1043
//*           Procedure in place. The named exits need to be in     *   FILE1043
//*           an APF authorized (STEPLIB) library, but AC bit       *   FILE1043
//*           off.                                                  *   FILE1043
//*                                                                 *   FILE1043
//*           If you are able to delete the faulty exits from       *   FILE1043
//*           such a STEPLIB the recovery is complete. The          *   FILE1043
//*           default IRX-Enviroment in the LPA will be active      *   FILE1043
//*           again.                                                *   FILE1043
//*                                                                 *   FILE1043
//*        -- same things apply to the preprocessor. This is a      *   FILE1043
//*           simple call out of a system exit. Bad user code is    *   FILE1043
//*           able to destroy the REXX enviroment.  Keep the        *   FILE1043
//*           module (and source) saved, to maintain your system    *   FILE1043
//*           integrity.                                            *   FILE1043
//*                                                                 *   FILE1043
//*        -- the things turn out to be much easier to test and     *   FILE1043
//*           control in a job-based environment. Here is the       *   FILE1043
//*           level of potential impact far better predictable      *   FILE1043
//*           and the benefits far outweigh possible risks.         *   FILE1043
//*                                                                 *   FILE1043
//*     * Conclusion & Note of Thanks                               *   FILE1043
//*                                                                 *   FILE1043
//*      Especially I want to thank Jason Winter for his great C    *   FILE1043
//*      compiler. Anthony Rudd for his excellent book about        *   FILE1043
//*      TSO/REXX (TSO/REXX Practical Usage). There are some other  *   FILE1043
//*      packages dealing with ways to enhance the usability of     *   FILE1043
//*      TSO/E REXX environment. A incomplete list of packages      *   FILE1043
//*      dealing with global variables contains CBT791, CBT411 and  *   FILE1043
//*      CBT413 and all the other packages dealing with generating  *   FILE1043
//*      and providing REXX Function pacs.                          *   FILE1043
//*                                                                 *   FILE1043
```
