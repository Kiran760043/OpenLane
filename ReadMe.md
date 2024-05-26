#+title: VSD 
#+date:[2024-05-23 Thu 16:23]
#+options: toc: 4

* Table pf Contents :toc:
- [[#vsd-day-1][VSD DAY 1]]
  - [[#openlane][OpenLane]]
  - [[#process-design-kit-pdk][Process Design Kit (PDK)]]
  -  [[#digital-standard-cell][Digital Standard Cell]]
  - [[#standard-cell-libraries][Standard Cell Libraries]]
  - [[#technology-libraries][Technology libraries]]
  - [[#run-openlane][Run OpenLane]]

* VSD DAY 1

The goal of day 1 is to install openlane via the lab manual and familiarize with the tool.

** OpenLane 

Openlane is collaboration software the includes multiple opensource tools to automate the steps involed in RTL to GDSII flow. The openlane can be installed from [[https://github.com/The-OpenROAD-Project/OpenLane][GitHub]], this [[https://woset-workshop.github.io/PDFs/2020/a21.pdf][Paper]] discussing it two main uses case:
1. Harderning a macro 
2. Integrating a System-on-Chip (SoC).

#+name: OpenLane Flow
#+caption: Figure 1: OpenLan Flow
#+attr_html: : width 600px
[[~/OrgFiles/orgRoam/resources/VSD_Day1_openLane_flow.png]]


For each design step in the ASIC desing flow an equivalent open source tool is utilized to accomplish the task. The open source tool list for each design step is as follows according to [[https://www.semanticscholar.org/paper/Building-OpenLANE%3A-A-130nm-OpenROAD-based-Tapeout-%3A-Shalan-Edwards/512e49a704bb9f461a7ee12edd0639b29f8a4976][Paper]]:

| Desing Step         | EDA Tool                  |
| ------------------- | ------------------------- |
| Logic Synthesis     | Yosys and ABC             |
| DFT Scan Insertion  | Fault                     |
| DFT ATPG            | Fault                     |
| Formal verification | Yosys                     |
| Placement           | RePlAce and OpenDP        |
| Routing             | FastRoute and TritonRoute |
| CTS                 | TritonCTS                 |
| Extraction          | Magic                     |
| Timing Analysis     | OpenSTA                   |
| Chip Floorplanning  | PADGen                    |
| LVS                 | Netgen                    |
| DRC                 | Magic                     |
| GDS Streaming Out   | Magic                     |


From Figure 1, the OpenLane tool requires two catergories of files: 
1. Design Files: These are the RTL files that are written by an RTL design engineer.
2. PDKs: Process Design Kit (PDK) is a set of files used to model a fabrication process. Generally, the PDK files are degined by a foundary with a certain technology.

** Process Design Kit (PDK)
The OpenLane tools uses PDKs from SkyWater 130nm process. The complete documentation for SkyWater SKY130 PDK can be found [[https://skywater-pdk.readthedocs.io/en/main/index.html][here]].

The nomenclature used to identify SKY130 PDK is as follows:

<process name>_<library_source_abbreviation>_<library_type_abbreviation>[_library_name]

- Process name: is the name of the process technology and is always "sky130".

- Library source abbreviation: specifies the foundry source of the library. The table below shows the list of the library sources:
| Library source            | Library source abbreviation |
|---------------------------+-----------------------------|
| SkyWater                  | fd                          |
| Efabless                  | ef                          |
| Oklahoma State University | osu                         |

- Library type abbreviation: content found in the library, the table below shows the list of abbreviations:
  | Library type                   | Library type abbreviation |
  |--------------------------------+---------------------------|
  | Primitive cell                 | pr                        |
  | Digital Standard cell          | sc                        |
  | Build Space (flash, SRAM, etc) | bd                        |
  | IO and Preiphery               | io                        |
  | Miscellaneous                  | xx                        |
  
- Library name: is a the representation of the type of content found in the library. 

**  Digital Standard Cell 

- sky130_fd_sc_hd: SKY130 High Density Digital Cells
- sky130_fd_sc_hdll: SKY130 High Density Low Leakage Digital Standard Cells
- sky130_fd_sc_hs: SKY130 High Speed Digital Standard Cells
- sky130_fd_sc_ls: SKY130 Low Speed Digital Standard Cells
- sky130_fd_sc_ms: SKY130 Medium Speed Digital Standard Cells

From the name it can be seen that all the standard cells libraries are provided by SkyWater.

** Standard Cell Libraries
Digital Standard Cells can found in a folder called "~/tools/openlane_working_dir/pdks/skywater-pdk/libraries/". Among the available standard cells let us look at a high density two input AND gate in the folder "sky130_fc_sc_hd/latest/cells".

#+name: AND Gate Symbol
#+caption: Figure 2: AND Gate Cell Symbol
#+attr_html: :width 600px

[[./resources/VSD_Day1_and2.png]]

#+name: source code
#+caption: Verilog Behavioral code for two input AND Gate

#+begin_src verilog
/*
 * Copyright 2020 The SkyWater PDK Authors
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 * SPDX-License-Identifier: Apache-2.0
*/


`ifndef SKY130_FD_SC_HD__AND2_BEHAVIORAL_PP_V
`define SKY130_FD_SC_HD__AND2_BEHAVIORAL_PP_V

/**
 * and2: 2-input AND.
 *
 * Verilog simulation functional model.
 */

`timescale 1ns / 1ps
`default_nettype none

// Import user defined primitives.
`include "../../models/udp_pwrgood_pp_pg/sky130_fd_sc_hd__udp_pwrgood_pp_pg.v"

`celldefine
module sky130_fd_sc_hd__and2 (
    X   ,
    A   ,
    B   ,
    VPWR,
    VGND,
    VPB ,
    VNB
);

    // Module ports
    output X   ;
    input  A   ;
    input  B   ;
    input  VPWR;
    input  VGND;
    input  VPB ;
    input  VNB ;

    // Local signals
    wire and0_out_X       ;
    wire pwrgood_pp0_out_X;

    //                                 Name         Output             Other arguments
    and                                and0        (and0_out_X       , A, B                  );
    sky130_fd_sc_hd__udp_pwrgood_pp$PG pwrgood_pp0 (pwrgood_pp0_out_X, and0_out_X, VPWR, VGND);
    buf                                buf0        (X                , pwrgood_pp0_out_X     );

endmodule
`endcelldefine

`default_nettype wire
`endif  // SKY130_FD_SC_HD__AND2_BEHAVIORAL_PP_V

#+end_src

** Technology libraries

 To run the 

** Run OpenLane

*** Task: Synthesize picorv32a design and find the flop ratio. 

1. Open a terminal in "~/Desktop/work/tools/openlane_working_dir/openlane" and excute "docker" and the "./flow.tcl -interactive" and add required packages. See the figure below for more information.

#+attr_html: :width 600px
[[./resources/VSD_Day1_openlane.png]]


Picorv32a is simple risc-v processor core present in the "design" dicrectory. Before synthesizing the design prep the design so that it links to all the tech file and lib files for the design to be synthesized.
2. Execute "prep -design picorv32a".
[[./resources/VSD_Day1_prep.png]]3. Run synthesis by executing "run_synthesis"
[[./resources/VSD_Day1_syn.png]]
The total number of cells is 14876 and the number of D-FF utilized is 1613, therefore, the flop ratio is 10.84%. 
