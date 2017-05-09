# PREREQUISITE
  * infiniband-diags
  * Graphviz and pydot (optional, for visualizing topology only)
Help on module ibcheck:


# DOCUMENTATION
  Run "pydoc ./ibcheck" to generate the following documentation.




## NAME
    ibcheck - Swiss army knife for Infiniband (IB) troubleshooting.

## FILE
    ./ibcheck

## DESCRIPTION
    ############################################################################
    #                                                                          #
    # Copyright (c) 2010-2012, The Regents of the University of California,    #
    # through Lawrence Berkeley National Laboratory (subject to receipt of any #
    # required approvals from the U.S. Dept. of Energy).  All rights reserved. #
    #                                                                          #
    #                                                                          #
    # Author: Yong Qin <yong.qin@lbl.gov>                                      #
    #         High Performance Computing Services (http://scs.lbl.gov/)        #
    #                                                                          #
    ############################################################################


    IBCHECK is a utility that uses the standard tools provided by OFED
    infiniband-diags package to perform comprehensive in-band IB troubleshooting.


    IBCHECK modules:
      1. Topology analyzer.
      2. Subnet Manager (SM) scanner.
      3. Performance Manager (PM) scanner.


    IBCHECK files:
      1. Fabric definition file - supplied with "-f" or "-c" option.

         Fabric definition file provides a key/value pair definition for a given
         GUID/NodeDesc (node description) mapping.  It can also be organized as
         virtual fabric which is useful when used with "-C" option. E.g.,

           [lustre]
           0x0005ad00001df29a=ib000.lustre (Cisco SFS7000D, W29-36)
           0x0002c9020028ea84=n0000.lustre (mds00)
           0x00066a0098009c8a=n0002.lustre (oss00)
           0x0005ad00000c6410=n0003.lustre (oss01)
           ...

      2. Topology definition file - supplied with "-t" option.

         Topology definition file has the same format as the output from
         "ibnetdiscover" command.  When this file is provided, IBCHECK will perform
         topology comparison (see topology analyzer section for details).  The best
         practice for creating such file would be during the fabric bring-up stage,
         after everything is tested and confirmed in a good working manner.  One
         can use the following command to generate this file.

           $ ibnetdiscover > ibnetdiscover_myfabric.txt


    IBCHECK fabric selection:
      By default all devices on the fabric are selected to perform an action.
    However the following options can also be used to further narrow down the
    portion of the fabric to work with.

      1. By pre-defined virtual fabric in the fabric file: -c <CONF> -C <FABRIC>
      2. By device type: -T <TYPE>
      3. By spine index: -S <SPINE>
      4. By leaf/line index: -L <LEAF>
      5. By NodeDesc: -N <NODEDESC>
      6. By GUID: -g <GUID>
      7. By LID: -l <LID>
      8. By link speed: -s <SPEED>
      9  By link width: -w <WIDTH>

      One can also use the "-A" option to switch from the default "or" operation
    to an "and" operation to further control the selection.  In complex environment,
    it is recommended to construct a virtual fabric and use "-C" to control the
    target selection.


    IBCHECK detail level:
      Detail level ("-d") option can be stacked and controls the granularity of the
    action.  "-d" has the same controllability as without providing a detail level,
    which presents all devices that are selected (see the fabric selection section).
    "-dd" presents all ports with a link on the selected devices.  "-ddd" presents
    all ports on the selected devices regardless whether there is a link or not.
    Behavior for more than 3 levels of details ("-ddd") is not defined.


    IBCHECK topology analyzer:
      Topology analysis is always performed.  Depending on the detail level that
    IBCHECK is provided, it shows different level of details of the fabric.

      If a topology file is provided, IBCHECK will compare the current physical
    fabric with the fabric defined in the topology file to identify differences.
    It is useful when troubleshooting large fabric with dead/dropped links or ib
    devices, or comparing fabric changes.

      Topology analyzer also has a visualization component ("-G") which requires
    graphviz and pydot packages to be available.  This feature is experimental.

      It can also be run in the "dryrun" mode ("-D"), which requires a topology
    file (provided with "-t" option).  In this mode it does not try to retrieve
    the physical fabric topology but perform analysis only on the provided file,
    which is useful in offline analysis and visualization.


    IBCHECK SM scanner:
      SM scanner is activated with "-M" option on selected devices.  Note that
    a detail level setting controls its behavior as well.  If "-d" is used, only
    device level scanning is performed, which could lead to less information than
    expected, as the device GUID for HCA could be different from its port GUID,
    which a host-based SM typically runs on.  Thus "-dd" is recommended in all
    situations for the SM scanner.  "-ddd" leads to scanning of empty ports
    without a link, which is not necessary.


    IBCHECK PM scanner:
      PM scanner ("-E") has two running modes: 1). Monitor mode; 2). Batch mode.

      Monitor mode is default and it launches a text-based user interface (TUI)
    with an embedded mini help page.  One can use the 'h' key to activate it.  Keys
    '[0-9]' and '[a-f]' can be used to toggle different error/performance counters
    to show.  Arrow keys and 'n', 'p', ',', '.', PgUp/PgDn can be used to scroll
    up/down, and left/right if the display is out of range.  'q' quits from the
    Monitor mode. Space bar switches between showing devices/ports with error
    counters above the thresholds and all devices/ports (similar to "-a" option).

      Batch mode ("-b") performs the same type of PM scan, but it displays results
    in a loggable way, which can be used for offline analysis, such as trending
    analysis.  In Batch mode, all error/performance counters are displayed.

      One can also reset the counters on the selected devices/ports.  In the monitor
    mode, this can be done by pressing the 'r' key; in the batch mode, "-r" achieves
    the same goal.

      By default PM scanner only displays IB devices with error counters greater
    than the preset thresholds.  This can be turned off by supplying "-a" option.
    Also by default it runs with all available cores on the host. This behavior can
    be adjusted by providing "-p <THREADS>" option, which controls how many
    background processes to spawn.  The default scan interval is 60 seconds, and it
    can be changed with "-i <INTERVAL>".  In an online monitoring mode, "-i 5" gives
    a good refresh rate.  However in the batch mode, "-i 300" or greater can be
    used as a good practice.  Another option "-n <ITERATIONS>" controls how many
    times the PM scanner should be run in the batch mode.

      Again, "-dd" is recommended for PM scanner module in all cases.


    Examples:
      1. wwibcheck -h
         Display the help page.

      2. wwibcheck
         Sweep the fabric, display device counts.

      3. wwibcheck -d
         Sweep the fabric, display all devices found.

      4. wwibcheck -dd
         Sweep the fabric, display all devices, as well as active ports.

      5. wwibcheck -ddd
         Sweep the fabric, display all devices and all ports.

      6. wwibcheck -N n0000 -d
         Display all devices on the fabric with a node description starting with
         "n0000".

      7. wwibcheck -f fabrics.conf -d
         Display all devices, use their designated node description defined in
         fabrics.conf

      8. wwibcheck -f fabrics.conf -N n0000 -d
         Display all devices with a node description starting with "n0000", use
         their designated node descriptions.

      9. wwibcheck -f fabrics.conf -N n0000.lr -d
         Display device which has a node description "n0000.lr" defined in
         fabrics.conf

      10. wwibcheck -f fabrics.conf -t ibnetdiscover_lr.txt
          Sweep the fabric, compare it with the fabric topology defined in
          "ibnetdiscover_lr.txt".  This helps to identify dead links/devices
          immediately.

      11. wwibcheck -f fabrics.conf -C lr -d
          Sweep the fabric, only show devices defined in virtual fabric "lr".

      12. wwibcheck -f fabrics.conf -N ib000.lr,n0000.lr -d
          Only display devices "ib000.lr" and "n0000.lr".

      13. wwibcheck -f fabrics.conf -g 0x0002c90200431448 -d
          Only display device with a GUID "0x0002c90200431448".

      14. wwibcheck -f fabrics.conf -l 32 -d
          Only display device with a LID "32".

      15. wwibcheck -f fabrics.conf -dd -M
          Scan and display all subnet managers on the fabric.

      16. wwibcheck -f fabrics.conf -dd -E
          Activate the Monitor mode of the PM scanner, run with a 60 seconds
          interval, until 'q' or 'ESC' is pressed.

      17. wwibcheck -f fabrics.conf -dd -C lr -E -i5
          Monitor the error counters of the virtual fabric "lr", with a scan
          interval of 5 seconds.

      18. wwibcheck -f fabrics.conf -dd -C lr -E -b
          Activate the Batch mode of the PM scanner on the virtual fabric "lr",
          run once and exit.

      19. wwibcheck -f fabrics.conf -dd -C lr -E -b -i5 -n3
          Run PM scanner in Batch mode, run 3 times with a scan interval of 5
          seconds.

      20. wwibcheck -f fabrics.conf -dd -C lr -E -r -i5
          Reset error counters on the virtual fabric "lr", then start PM scanner
          in Monitor mode with a scan interval of 5 seconds.

## VERSION
    0.1

## DATE
    September 14, 2012

## AUTHOR
    Yong Qin <yong.qin@lbl.gov>


