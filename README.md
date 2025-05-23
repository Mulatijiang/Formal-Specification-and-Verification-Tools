# FSVT

Specification Testing framework prototype for [Dafny](https://github.com/dafny-lang/dafny) Specifications. 

FSVT brings together three different spec-testing techniques; Automatic Sanity Checking(ASC), automatic specification mutation testing, and a Methodology for writing Spec Testing Proofs (STPs). This repo contains the framework for the automated aspects of FSVT, mutation testing and the ASC. 

# Dependencies

* [FSVT-dafny-grpc-server](https://github.com/Mulatijiang/FSVT-dafny-grpc-server)
* openjdk-13-jdk 
* gcc-8
* g++-8
* bazel-4.0.0
* dotnet-6 

The prototype is built off of Dafny v3.8.1


> **_NOTE:_** All dependencies will be installed when running `./setup/configureFSVT.sh` - detailed below
# Setup

The recommended environment to run FSVT is using [CloudLab](https://www.cloudlab.us/):

FSVT is designed to use one root node and `n` distributed nodes for verification performance. The root node is responsible for generating the specification mutations and verification conditions, the `n` separate nodes are used to parallelize checking verification conditions. The `n` additional nodes are not necessary to run FSVT, they are used to help increase the performance and reduce experiment runtime. 

In CloudLab, a default profile is available titled "IronSpecConfigNodes" and is configured to use 21, c8220 nodes - this can be modified in the experiment instantiation to change the configuration. A copy of this profile can be found in `./setup/cloudlabProfile.txt`

After creating a CloudLab experiment, make note of all the unique numerical names of the nodes from the Node column in the experiment page List View. For example, the IDs of Nodes "clnode008" and "clnode059" are 008 and 059 respectively.

> **_NOTE:_**  FSVT should work on other CloudLab hardware, but if not using nodes from the Clemson cluster, some of the setup scripts will break. This can be fixed by replacing `.clemson.cloudlab.us` with the appropriate extension. 

## Getting Started

To check basic functionality, it is sufficient to follow the following "Setup" steps with a single CloudLab node. After setup, to test to make sure that everything is installed and configured correctly, run `./runSimpleExperiments.sh` to run the mutation framework on a simple Max Spec found in: `./specs/max/maxSpec.dfy` and the Automatic Sanity Checker on `./specs/sort/sortMethod.dfy`


After executing this command, see the output in a file `./experimentOutput/MaxSpecCorrect/maxSpecCorrect_output.txt` and the tail of the output will look something like this (the mutation IDs may be slightly different, and the mutation at the root may be a different equivalent mutations):

```
...
...
--- END Mutation Classifications -- 

TOTAL Elapsed Time is 15749 ms
root = (0):(a > b ==> c == a) && (b + 1 > a ==> c == b)
7 :: 8 :: 42 :: 43 :: 44 :: 48 :: 50 :: 52 :: 60 :: 62 :: 63 
---
Total Alive Mutations = 1

```

and the output in `./experimentOutput/sortASC/sortASC_output.txt` with a high serverity flag being raised `-- FLAG(HIGH) -- : NONE of Ensures depend on Any input parameters `


### FSVT Local VMware Ubuntu Environment Setup  

#### **Goal**  
Compile and run FSVT on a local computer without multi-node distributed setup.  
**Advantages**: Local development and testing without CloudLab dependency.  
**Disadvantages**: Cannot simulate real multi-node environments; potential environment inconsistencies with CloudLab.  


#### **Steps**  
1. **Install Dependencies**  
   Ensure local environment has: `git`, `make`, `g++`, `Bazel 4.0.0`, `.NET SDK`, `python3`, `pip`, etc.  

2. **Clone Repositories**  
   ```sh
   git clone https://github.com/Mulatijiang/FSVT
   git clone https://github.com/Mulatijiang/FSVT-dafny-grpc-server
   ```  

3. **Run Setup Scripts**  
   ```sh
   cd FSVT
   ./setup/node_prep.sh
   ./setup/install_dotnet_ubuntu_20.04.sh  # For Ubuntu 20.04
   ```  

4. **Modify Username in Configuration**  
   Update `DafnyVerifier.cs` to use your actual username (e.g., `murat`):  
   ```sh
   sed -i "s/username/murat/" FSVT/Source/Dafny/DafnyVerifier.cs
   ```  

5. **Build FSVT and Z3**  
   ```sh
   make exe
   make z3-ubuntu  # For Ubuntu
   ```  

6. **Build gRPC Server**  
   ```sh
   cd FSVT-dafny-grpc-server
   bazel-4.0.0 build --cxxopt="-g" --cxxopt="--std=c++17" //src:server
   ```  

7. **Configure gRPC Server Address**  
   Ensure `ipPorts.txt` contains:  
   ```sh
   echo "127.0.0.1:50051" > ipPorts.txt
   ```  


#### **Local Testing**  
1. **Start gRPC Server** (run in a separate terminal foreground):  
   ```sh
   cd FSVT-dafny-grpc-server
   ./bazel-bin/src/server -v -d ../FSVT/Binaries/Dafny
   ```  

2. **Test Simple Examples**  
   Run the script for basic tests:  
   ```sh
   ./runSimpleExperiments.sh
   ```  

3. **Verify Output**  
   Check results in the `experimentOutput` directory:  
   ```sh
   cat experimentOutput/MaxSpecCorrect/maxSpecCorrect_output.txt
   cat experimentOutput/sortASC/sortASC_output.txt
   ```  


### CloudLab Remote Setup  

#### **Steps on CloudLab Node `clnode153`**  
1. **Configure SSH Keys Locally**  
   ```sh
   ./setup/configureSSHKeys.sh Mulati 153 155
   ```  

2. **Connect to CloudLab Node**  
   ```sh
   ssh Mulati@clnode153.clemson.cloudlab.us
   ```  

3. **Clone FSVT Repository**  
   ```sh
   git clone https://github.com/Mulatijiang/FSVT
   cd FSVT
   ```  

4. **Run Environment Configuration**  
   ```sh
   ./setup/configureFSVT.sh Mulati 153 155
   ```  

5. **Start/Stop Dafny Servers**  
   ```sh
   ./setup/startDafnyServers.sh Mulati 153 155  # Start servers
   ```
   ```
   ./setup/checkDafnyPID.sh Mulati 153 155       # Check server status
   ```
   ```
   ./setup/stopDafnyServers.sh Mulati 153 155    # Stop servers
   ```  

6. **Run Simple Experiments**  
   ```sh
   ./runSimpleExperiments.sh
   ```  

7. **View Experiment Results**  
   ```sh
   cat experimentOutput/MaxSpecCorrect/maxSpecCorrect_output.txt
   cat ./experimentOutput/sortASC/sortASC_output.txt
   ```  


### Additional Testing Steps  

#### **Mutation Testing**  
1. **Start gRPC Server** (as in local testing):  
   ```sh
   cd FSVT-dafny-grpc-server
   ./bazel-bin/src/server -v -d ../FSVT/Binaries/Dafny
   ```  

2. **Run Mutation Test Command**  
   ```sh
   cd ~/Desktop/graduate/FSVT
   mkdir -p experimentOutput/MaxSpecCorrect/outputLogs
   ./Binaries/Dafny /compile:0 /timeLimit:1520 /trace /arith:5 /noCheating:1 /mutationTarget:maxExample.maxSpec /proofName:maxTest.maxT /proofLocation:"$(pwd)/specs/max/lemmaMaxTestCorrect.dfy" /serverIpPortList:ipPorts.txt $(pwd)/specs/max/maxSpec.dfy &> experimentOutput/MaxSpecCorrect/maxSpecCorrect_output.txt
   ```  


#### **ASC Testing**  
1. **Run ASC Command (No gRPC Server Needed)**  
   ```sh
   cd ~/Desktop/graduate/FSVT
   ./cleanExperimentOutput.sh
   mkdir -p experimentOutput/sortASC
   ./Binaries/Dafny /compile:0 /timeLimit:1520 /trace /arith:5 /noCheating:1 /mutationTarget:sort.SortSpec /proofName:sort.merge_sort /mutationRootName:sort.SortSpec /proofLocation:"$(pwd)/specs/sort/sortMethod.dfy" /serverIpPortList:ipPorts.txt /checkInputAndOutputSpecified $(pwd)/specs/sort/sortMethod.dfy &> experimentOutput/sortASC/sortASC_output.txt
   ```  

# Specs

The FSVT prototype is designed to facilitate testing Dafny specifications. Examples of specifications can be found in the `./specs`. 

This directory consists of 14 specifications, 6 in-house specifications, and 8 specifications from open-source systems.

The 6 in-house specifications, max, sort, binary search, key-value state machine, token-wre, simpleAuction-wre, and two open source specs, div, and NthHarmonic are included in whole. div and NthHarmonic are transcribed to dafny from ["Exploring automatic specification repair in dafny programs"](https://conf.researchr.org/details/ase-2023/ase-2023--workshop--asyde/8/Exploring-Automatic-Specification-Repair-in-Dafny-Programs). Token-wre and simpleAuction-wre were taken from ["Deductive verification
of smart contracts with dafny"](https://arxiv.org/abs/2208.02920) with small modifications to artificially introduce bugs into the corresponding specs. 

In the following table is the list of all specs. For the open-source specs, the link corresponds to the initial commit that was used when testing the spec with FSVT. 

| Spec Name | Source | 
| :---------------- | :------: |
| Max | `./specs` | 
| Sort | `./specs` | 
| Binary Search | `./specs` | 
| Key-Value Store SM | `./specs` | 
| token-wre | `./specs` |
| simpleAuction-wre | `./specs` | 
| div | `./specs` | 
| NthHarmonic | `./specs` |
| [QBFT](https://github.com/Consensys/qbft-formal-spec-and-verification/tree/1630128e7f5468c08983d08064230422d9337805) | external |
| [Distributed Validator](https://github.com/Consensys/distributed-validator-formal-specs-and-verification/tree/fed55f5ea4d0ebf1b7d21b2c6fc1ecb79b5c3dd3) | external |
| [daisy-nfsd](https://github.com/mit-pdos/daisy-nfsd/tree/98434db21451b76c700dcf5783b2bca00a63c132) | external |
| [TrueSat](https://github.com/andricicezar/truesat/tree/62f52fd82709b888fa604f20297f83572c8592ae) | external |
| [Eth2.0](https://github.com/Consensys/eth2.0-dafny/tree/4e41de2866c8d017ccf4aaf2154471ffa722b308) | external |
| [AWS ESDK](https://github.com/aws/aws-encryption-sdk-dafny/tree/eaa30b377be9c0c17aeae9fce11387b0fbccafba) | external |

# Running FSVT

To run all automatic experiments (Both Mutation Testing and Automatic Sanity Checking experiments) described in the OSDI paper, run: `./runAllExperiments.sh`

It is recommended to run this command in a detached shell session since it will take some time to complete. For example:

```
screen -S runExperiments
./runAllExperiments.sh
[Ctrl-a][Ctrl-d] #This detaches the screen "runExperiments"

#To reattach
screen -R runExperiments
```

> **_NOTE:_** Running `./runExperiments.sh` will automatically clone all external specs into the `./spec` directory to the originally tested commit hash. Additionally, small patches will be applied to external specs to accommodate for small syntactical changes between Dafny versions.

Output and logs for each experiment will be found in `./experimentOutput` split into different sub-directories for each spec and each experiment. The tail of each produced file's name ending in `..._output.txt` for each experiment will show the final results.

**After `./runExperiments.sh` finishes, check `./experimentOutput/finalResults.txt` for recreation of Tables 3 and 4 from the paper based on the results of running `./runExperiments.sh`** 

> **_NOTE:_** The original experimental setup consisted of 1 root node and 20 nodes running the dafny-GRPC servers to parallelize checking verification conditions. Using a different configuration will impact the total time to run each experiment. The time difference is approximately linear based on the number of nodes; for example, running with 21 nodes an experiment taking approximately 160 min will be 3x as long when running the same experiment with 7 nodes. Having too few nodes may cause verification timeouts that result from overloading the various nodes, to run the full experiments it is recommended to have at least 7 nodes. There is a minimum bound for the time to run each experiment, as even if all verification tasks can be parallelized, larger systems like qbft and DVT still will take 1-2 hours for a full end-to-end proof. 

> **_NOTE:_** Some results in appearing in`./experimentOutput/finalResults.txt` may differ slightly from Table 4 in the included pdf. 
> * The time taken for each experiment will vary based on the number of nodes, the CPU load at each node, and the black-box nature of z3, so there may be some variance in timing.
> * The number of alive mutations for the `Div`spec will appear higher (8 vs 3). This is due to a limitation in the final step of the mutation framework that is incompatible with this type of spec in classifying alive mutations into a DAG. The result of reducing the 8 alive mutations to 3 is calculated manually with the assistance of Dafny. Details on how to achieve this are included in comments in `./specs/div/div.dfy`.
> * The number of generated mutations for QBFT NetworkInit, QBFT AdversaryInit, and DVT AdversaryNext should be 44, 35, and 110 respectively. These are updated results due to an improvement in avoiding producing duplicate mutations and will be adjusted in the camera-ready version. The orignal numbers are from older experiments, and are typos. The difference in mutations is due to duplicate or logically equivalent mutations so this difference does not affect the final number of alive mutations or even the total set of mutations tested, just accounts for duplicated mutations.
> * The number of alive mutations for QBFT AdversaryNext should be 4. This is the result of an increased timeout allowance for the mutation classification step - the same information is included in the 4 alive mutations as in the previous 7, so this improvement over the original values in table 4 as it further minimizes the manual effort to inspect more mutations. As a consequence of this same timeout allowance increase, there is expected to be one more alive mutation for DVT AdversaryNext. This alive mutation gives the same hint as the others. 
 

To run either the mutation testing experiments or just the ASC experiments, utilize either `./runMutationTestingExperiments.sh` or `./runASCExperiments.sh`. If running either of these, make sure to also run `./cleanExperimentOutput.sh` beforehand.

##### Mutation Testing 

The general command to run mutation testing follows this form:

`./Binaries/Dafny [Standard Dafny Arguments] [FSVT Arguemnts] [full-path-to-the-file-containing-mutation-target] &> output.txt`

For example to test the specification for a Max method (`/specs/max/maxSpec.dfy`), run the following command from the root node:
```
./Binaries/Dafny /compile:0 /timeLimit:1520 /trace /arith:5 /noCheating:1 /mutationTarget:maxExample.maxSpec /proofName:maxTest.maxT /proofLocation:"$(pwd)/specs/max/lemmaMaxTestCorrect.dfy" /holeEvalServerIpPortList:ipPorts.txt $(pwd)/specs/max/maxSpec.dfy &> output.txt
```

After executing this command, see the output in a file named `output.txt` and the tail of the output will look like this:

```
...
...
--- END Mutation Classifications -- 

TOTAL Elapsed Time is 15749 ms
root = (0):(a > b ==> c == a) && (b + 1 > a ==> c == b)
7 :: 8 :: 42 :: 43 :: 44 :: 48 :: 50 :: 52 :: 60 :: 62 :: 63 
---
Total Alive Mutations = 1

```
This output indicates that there is a single alive mutation(with ID 0) to investigate - `(a > b ==> c == a) && (b + 1 > a ==> c == b)` 

Full experiment logs can be found in `./outputLogs` which will contain the logs for each mutation and the intermediary files generated for the different mutation testing passes. Any `.txt` file contains the output from the dafny verifier for each mutation. There are many `.dfy` files generated to test the different mutation passes (i.e. weaker, vac, full, classification)

##### Automatic Sanity Checking (ASC)

The general command to run mutation testing follows this form:

`./Binaries/Dafny` [Standard Dafny Arguments] [FSVT Arguemnts] `/checkInputAndOutputSpecified` [full path to the file containing mutation target] `&> output.txt`

```
./Binaries/Dafny /compile:0 /timeLimit:1520 /trace /arith:5 /noCheating:1 /holeEval:testWrapper.SATSolver.test /proofName:testWrapper.SATSolver.start /mutationRootName:testWrapper.SATSolver.test /holeEvalRunOnce /holeEvalDepth:2 /proofLocation:"$(pwd)/specs/truesat/truesat_src/solver/solver.dfy" /holeEvalServerIpPortList:ipPortOneNode.txt /checkInputAndOutputSpecified $(pwd)/specs/truesat/truesat_src/solver/solver.dfy &> output.txt
```

# SpecTesting Proof(STPs) Examples
A brief description and some examples are included in `./STPS` 

STPs are just Dafny proofs, so running STPS requires no additional instrumentation other than just running Dafny. 


# FSVT arguments

| Argument | Description |
| -------- | ------- |
| /mutationTarget:[name] | Full Name of predicate for mutation target. In Dafny, this includes the module name followed by the predicate name. For example, if the predicate's name is `spec` located in module `Example`, the full name is: `Example.spec` |
| /proofName:[name] | Name of lemma for full end-to-end proof. Also follows `Module.Name` format |
| /proofLocation:[name] | Full path to `.dfy` file containing lemma targeted for /proofName |
| /serverIpPortList:[name] | Name of `.txt` file containing the IP addresses and ports for other node end points. If configured using `./configureFSVT.sh` a file named `ipPorts.txt` will be created with the correct formatting. An example is included: `ipPortsExample.txt` |
|/inPlaceMutation | This flag indicates that the mutation target is for a method with pre/post conditions rather than a predicate. In this case /proofName is the name of the method to mutate the spec and the proof to check. | 
| /mutationRootName:[name] | If the mutation target is different from the high-level safety property of the system, make sure to specify the high-level safety property with this flag. | 
| /checkInputAndOutputSpecified | Argument to indicate the use of the ASC. When using this flag, /mutationTarget and /mutationRootName should be set to a predicate in the same file, the contents of the predicate are irrelevant, but these flags are still needed to run. /proofName should be set to the method that is being tested with the ASC | 
|/isRequires | Tests for mutations that are stronger than the original, rather than weaker|