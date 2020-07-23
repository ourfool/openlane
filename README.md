      ___   ____   ___  ____   _       ____  ____     ___
     /   \ |    \ /  _]|    \ | |     /    ||    \   /  _]
    |     ||  o  )  [_ |  _  || |    |  o  ||  _  | /  [_
    |  O  ||   _/    _]|  |  || |___ |     ||  |  ||    _]
    |     ||  | |   [_ |  |  ||     ||  _  ||  |  ||   [_
     \___/ |__| |_____||__|__||_____||__|__||__|__||_____|
     

# Table of contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Getting Started - Directory Setup](#getting-started---directory-setup)
- [Getting Started - OpenLANE Flow Setup](#getting-started---openlane-flow-setup)
- [Getting Started - PDK Setup](#getting-started---pdk-setup)
    - [Setting up skywater-pdk](#setting-up-skywater-pdk)
    - [Setting up other PDKs](#setting-up-other-pdks)
- [Getting Started - How to Run](#getting-started---how-to-run)
    - [Running the flow](#running-the-flow)
    - [Command line arguments](#command-line-arguments)
    - [Adding a design](#adding-a-design)
- [The Flow](#the-flow)
    - [Stages](#stages)
    - [Flow output](#flow-output)
    - [Flow configuration](#flow-configuration)
    - [Interactive Mode](#interactive-mode)
- [Regression And Design Configurations Exploration](#regression-and-design-configurations-exploration)

# Overview
OpenLaneは、OpenRoad、Yosys、Magic、Netgen、Fault、およびデザインの探索と最適化のためのカスタムメソドロジースクリプトを含むいくつかのコンポーネントをベースにした、RTLからGDSIIまでの自動化されたフローです。このフローは、RTLからGDSIIに至るまで、ASICの完全な実装ステップを実行します。この機能は、SkyWaterにファブリケーションのために送られた完成したSoCデザインの例とともに、近日中にリリースされる予定です。



# Prerequisites
 
 - Docker

# Getting Started - Directory Setup
実行プロセスを簡単にするために、以下の設定が推奨されています:

```
openlane_working_dir
├── pdks
├── openlane
```

So before you start run the following commands:
```bash
    mkdir openlane_working_dir
    cd openlane_working_dir
    mkdir pdks
    mkdir openlane 
```

# Getting Started - OpenLANE Flow Setup

Clone & build OpenLANE:

```bash
    git clone https://github.com/efabless/openlane --branch rc1
    cd openlane/docker_build
    make merge
    cd ..
    cd ..
```

dockerコンテナとそのプロセスの詳細については、 [following instructions][1] では、dockerコンテナを使用して必要なツールを構築し、それをOpenLaneフローに統合するプロセスを説明しています。

# Getting Started - PDK Setup

フローを実行する前に、少なくとも1つのPDKを openlane_working_dir/pdks/ に設定しておく必要があります。

## Setting up skywater-pdk

このセクションでは、オープンレーン上での [skywater-pdk](https://github.com/google/skywater-pdk) の設定方法を説明します。説明した手順は、他のPDKのセットアップ方法の際にも参考となるでしょう。

- Clone and build one skywwater-pdk variant(s) inside the pdks directory:
    - To setup one variant only
    ```bash
        cd pdks
        git clone https://github.com/google/skywater-pdk.git
        cd skywater-pdk
        git checkout 4e5e318e0cc578090e1ae7d6f2cb1ec99f363120
        git submodule update --init libraries/sky130_fd_sc_hd/latest
        make sky130_fd_sc_hd 
        cd ..
    ```
    - To setup other variants:
        - replace sky130_fd_sc_hd with any of the following list:
            - sky130_fd_sc_hs
            - sky130_fd_sc_ms
            - sky130_fd_sc_ls
            - sky130_fd_sc_hdll

- Setup the configurations and tech files:
    - To perform physical verification you need to use the magic tool. Therefore, you need to setup the pdk using [open-pdks](https://github.com/efabless/open_pdks):
    ```bash
        git clone https://github.com/efabless/open_pdks.git
        cd open_pdks
        git checkout c2fec9fe64146000236dd807165b80b6a8b82b89
        make
        make install-local
        cd ..
        cd ..
    ```

    **Note:** You may want to change the pdk installation directory or control other options explained [here](https://github.com/efabless/open_pdks/blob/master/sky130/README).

    **Note:** If you don't want to use open_pdks and want to create the setup manually (This will only allow you to run the flow up to a routed-def), make sure you provide what is expected [here][24].

 - Point to the (PDK,PDK_VARIANT) pair:
    - Open [./openlane_working_dir/openlane/configuration/general.tcl](./configuration/general.tcl)
    - set PDK to the name of the folder containing the pdk under PDK_ROOT. <br> Default: sky130A
    - set PDK_VARIANT to the name of the library as it exists under libs.tech. <br> Default: sky130_fd_sc_hd

Refer to [this][24] for more details on the structure.

## Setting up other PDKs

 - Provide the setup expected [here][24].
 - To perform physical verification you need to use the magic tool setup.
 - Point to the (PDK,PDK_VARIANT) pair:
    - Go into [./configuration/general.tcl](./configuration/general.tcl)
    - set PDK to the name of the folder containing the pdk under PDK_ROOT.
    - set PDK_VARIANT to the name of the library as it exists under libs.tech

Refer to [this][24] for more details on the structure.


# Getting Started - How to Run

## Running the flow

dockerイメージをビルドした後、以下のコマンドを実行してプロジェクトのルートからdockerコンテナを開き、コンテナを終了した後も出力ファイルが持続するようにします。:


```bash
    docker run -itv $(pwd)/openlane:/openLANE_flow -v $(pwd)/pdks/open_pdks/sky130/pdks:/openLANE_flow/pdks -u $(id -u $USER):$(id -g $USER) openlane:rc1
```

**Note: this will mirror the openlane directory inside the container.**

This will mount the docker with the root folder being ```./openLANE_flow``` containing inside it the working directory specified, as well as the pdks directory in which you installed the pdks as a sub-directory

Then, you can test the flow using one of the provided designs (spm: Serial-Parallel Adder) by running:

```bash
./flow.tcl -design spm
```

To start a batch run (multiple designs at the same time), check this [section](#regression-and-design-configurations-exploration).

More details on running different designs in the following sections.


## Command line arguments

The following are arguments that can be passed to `flow.tcl`

<table>
    <tr>
        <th width="196">
        Argument
        </th>
        <th >
        Description
        </th>
    </tr>
    <tr>
        <td align="center">
            <code>-design</code> <br> (Required)
        </td>
        <td align="justify">
            Specifies the design folder. A design folder should contain a config.tcl definig the design parameters. <br> If the folder is not found, ./designs directory is searched
        </td>
    </tr>
    <tr>
        <td align="center">
            <code>-config &lt;name&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies the design's configuration file for while running the flow. <br> For example, to run the flow using <code>designs/spm/config2.tcl</code> <br> Use run <code>./flow.tcl -design spm -config config2.tcl</code> <br> By default <code>config.tcl</code> is used.
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-tag &lt;name&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a <code>name</code> for a specific run. If the tag is not specified, a timestamp is generated for identification of that run. <br> Can Specify the configuration file name in case of using <code>-init_design_config</code>
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-run_path &lt;path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a <code>path</code> to save the run in. By default the run is in <code>design_path/</code>, where the design path is the one passed to <code>-design</code>
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-save <br> (Optional)
        </td>
        <td align="justify">
            A flag to save a runs results like .mag and .lef in the design's folder
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-save_path &lt;path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a different path to save the design's result. This options is to be used with the <code>-save</code> flag
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-init_design_config </code> <br> (Optional)
        </td>
        <td td align="justify">
            Creates a tcl configuration file for a design. <code>-tag &lt;name&gt;</code> can be added to rename the config file to <code>&lt;name&gt;.tcl</code>
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-overwrite</code> <br> (Optional)
        </td>
        <td align="justify">
            Flag to overwirte an existing run with the same tag
        </td>
    </tr>    
    <tr>
        </tr>
        <td align="center">
            <code>-interactive</code> <br> (Optional)
        </td>
        <td align="justify">
            Flag to run openlane flow in interactive mode
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-file &lt;file_path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Passes a script of interactive commands in interactive mode 
        </td>
    </tr>
</table>


## Adding a design

To add a new design, follow the instructions provided [here](./designs/README.md)

This [file](./designs/README.md) also includes useful information about the design configuration files. It also includes useful utilities for exploring and updating design configurations for each (PDK,PDK_VARIANT) pair.

# The Flow

<table>
  <tr>
    <td  align="center"><img src="./doc/openlane.flow.1.png" ></td>
  </tr>

</table>


## Stages

OpenLane flow consists of several stages. Each stage may consist of multiple sub-stages. OpenLane can run either interactively (the designer executes the flow steps one by one) or non-interactively (default; running all the flow steps one after another):

1. **Synthesis**
    1. `yosys` - Performs RTL synthesis
    2. `abc` - Performs technology mapping
    3. `OpenSTA` - Pefroms static timing analysis on the resulting netlist to generate timing reports
2. **Floorplan and PDN**
    1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
    2. `ioplacer` - Places the macro input and output ports
    3. `pdn` - Generates the power distribution network
    4. `tapcell` - Inserts welltap and decap cells in the floorplan
3. **Placement**
    1. `RePLace` - Performs global placement
    2. `Resizer` - Performs optional optimizations on the design
    3. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. **CTS**
    1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. **Routing** *
    1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
    2. `TritonRoute` - Performs detailed routing
6. **GDSII Generation**
    1. `magic` - Streams out the final GDSII layout file from the routed def
7. **Checks**
    1. `magic` - Performs DRC Checks & Antenna Checks
    2. `Netgen` - Performs LVS Checks

OpenLane uses the following tools:
- RTL Synthesis, Technology Mapping, and Formal Verification : [yosys + abc][4]
- Static Timing Analysis: [OpenSTA][8]
- Floor Planning: [init_fp][5], [ioPlacer][6], [pdn][16] and [tapcell][7] 
- Placement: [RePLace][9] (Global), [Resizer][15] (Optimizations), and [OpenDP][10] (Detailed)
- Clock Tree Synthesis: [OpenROAD/TritonCTS][11]
- Fill Insertion: [OpenROAD/filler_placement][19]
- Routing: [FastRoute][12] (Global) and [TritonRoute][13] (Detailed)
- GDSII Streaming out: [Magic][14]
- DRC Checks: [Magic][14]
- LVS check: [Netgen][22]
- Antenna Checks: [Magic][14]

## Flow output

After running a design, a directory called `runs` is created inside that design folder. Each run is placed under that directory.
A run is either named after `-tag` or named with a timestamp of that run.
A run folder contains:

1. The run configuration
2. Four directories. Inside each, there is a folder per stage of the flow
    1. `logs` - Runtime log of each tool. Can be useful in case of a crash.
    2. `reports` - Reports generated by some of the tools. (i.e. synthesis report, timing report, etc...)
    3. `results` - Final files created by completing a stage. Can either be `.def` or `.v` or `.gds`.
    4. `tmp` - Files created by substages.

The resulting directory tree is as follows:

```
designs/spm
├── config.tcl
├── runs
│   ├── <tag>
│   │   ├── config.tcl
│   │   ├── logs
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── reports
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── results
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   └── tmp
│   │       ├── cts
│   │       ├── floorplan
│   │       ├── magic
│   │       ├── placement
│   │       ├── routing
│   │       └── synthesis
```

## Flow configuration

OpenLane requires some environment variables to be defined. These variables are split into three main categories:

1. PDK specific
2. Flow specific
3. Design specific

- A PDK should define at least one variant for the PDK. A common configuration file for all PDK variants is located in:

    ```
    ./pdks/<pdk>/common_config.tcl
    ```

    - Sometimes the PDK comes with several Standard Cell Libraries or Metal Stacks. Each is considered as a PDK variant. A variant configuration file defines extra variables specific to the variant. It may also override variables in the common PDK configuration file which is located in:

        ```
        ./pdks/<pdk>/<variant>/config.tcl
        ```
    - More on configuring a new PDK in this [section](#setting-up-a-pdk)

- Flow specific variables are related to the flow and are initialized with default values in:

    ```
    ./configuration/
    ```

- Finally, each design should have it's own configuration file with some required variables which are available in this [list][17]. A design configuration file may override any of the variables defined in PDK or flow configuration files. This is the global configurations for the design:

    ```
    ./designs/<design>/config.tcl
    ```
    - More on design configurations in [here](./designs/README.md)

A list of all available variables can be found [here][17].


## Interactive Mode
You may run the flow interactively by using the `-interactive` option:

```
./flow.tcl -interactive
```

A tcl shell will be opened where the openlane package is automatically sourced:
```
% package require openlane 0.9
```

Then, you should be able to run the following commands:

0. Any tcl command.
1. `prep -design <design> -tag <tag> -config <config> -init_design_config -overwrite` similar to the command line arguments, design is required and the rest is optional
2. `run_synthesis` 
3. `run_floorplan`
4. `run_placement`
5. `run_cts`
6. `run_routing`
7. `run_magic`
8. `run_magic_spice_export`
9. `run_magic_drc`
10. `run_netgen`
11. `run_magic_antenna_check`


The above commands can also be written in a file and passed to `flow.tcl`:

```
./flow.tcl -interactive -file <file>
```

**Note 1:** Currently, configuration variables have higher priority over the above commands so if `RUN_MAGIC` is 0, command `run_magic` will have no effect. 

**Note 2:** Currently, all these commands must be run in sequence and none should be omitted.

# Regression And Design Configurations Exploration

As mentioned earlier, everytime a new design or a new (PDK,PDK_VARIANT) pair is added, or any update happens in the flow tools, a re-configuration for the designs is needed. The reconfiguration is methodical and so an exploration script was developed to aid the designer in reconfiguring his designs if needed.
As explained [here](#adding-a-design) that each design has multiple configuration files for each (PDK,PDK_VARIANT) pair.

## Overview
OpenLane provides `run_designs.py`, a script that can do multiple runs in a parallel using different configurations. A run consists of a set of designs and a configuration file that contains the configuration values. It is useful to explore the design implementation using different configurations to figure out the best one(s). 

Also, it can be used for testing the flow by running the flow against several designs using their best configurations. For example the following has two runs: spm and xtea using their default configuration files `config.tcl.` :
```
python3 run_designs.py --designs spm xtea des aes256 --tag test --threads 3
```

For more information on how to run this script, refer to this [file](./Regression_Exploration.md)

For more information on design configurations, how to update them, and the need for an exploration for each design, refer to this [file](./designs/README.md)
 

[1]: ./docker_build/README.md
[2]: ./configuration/README.md
[3]: ./doc/flow.png
[4]: https://github.com/YosysHQ/yosys
[5]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/init_fp
[6]: https://github.com/The-OpenROAD-Project/ioPlacer/
[7]: https://github.com/The-OpenROAD-Project/tapcell
[8]: https://github.com/The-OpenROAD-Project/OpenSTA
[9]: https://github.com/The-OpenROAD-Project/RePlAce
[10]: https://github.com/The-OpenROAD-Project/OpenDP
[11]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/TritonCTS 
[12]: https://github.com/The-OpenROAD-Project/FastRoute/tree/openroad
[13]: https://github.com/The-OpenROAD-Project/TritonRoute
[14]: https://github.com/RTimothyEdwards/magic
[15]: https://github.com/The-OpenROAD-Project/Resizer
[16]: https://github.com/The-OpenROAD-Project/pdn/
[17]: ./configuration/README.md
[18]: https://github.com/RTimothyEdwards/qflow/blob/master/src/addspacers.c
[19]: https://github.com/The-OpenROAD-Project/
[20]: https://github.com/git-lfs/git-lfs/wiki/Installation
[21]: ./logs/README.md
[22]: https://github.com/RTimothyEdwards/netgen
[24]: ./pdks/README.md

