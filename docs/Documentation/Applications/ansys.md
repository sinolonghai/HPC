#Using ANSYS 

*ANSYS is a licensed commercial application for the Eagle system.* 

The NREL Computation Science Center (CSC) maintains an ANSYS computational fluid dynamics (CFD) license pool for general use, including two seats of CFD and four ANSYS HPC Packs for parallel solves.

!!! tip "Important"
     Network floating licenses are a shared resource. Whenever you open an ANSYS window, a license is pulled from the pool and becomes unavailable to other Eagle users. Please do NOT keep idle windows open—if you are not actively using the application, close it and return the associated licenses to the pool. Excessive retention of software licenses falls under the inappropriate use policy.

The main workflow that we support has two stages. The first is interactive graphical usage for *e.g.*, building meshes. For this case, ANSYS should be run on the DAV nodes in serial, and preferably without invoking simulation capability. The second stage is batch (*i.e.*, non-interactive) parallel processing, which should be run on compute nodes via a Slurm job script. Of course, if you have ANSYS input from another location ready to run in batch, the first stage is not needed. We unfortunately cannot support running parallel jobs on the DAV nodes, nor launching parallel jobs from interactive sessions on compute nodes.

License usage can be checked on Eagle with the command `lmstat.ansys`.

## Running ANSYS Interactively
GUI access is provided through FastX desktops. Open the Terminal application to issue the commands below.

1. To enable the ANSYS Fluent environment, use:

    ```
    module load ansys/<version>
    ```

1. To start Workbench, you can then issue the command `vglrun runwb2`.

## Licenses and Scaling
HPC Pack licenses are used to distribute ANSYS batch jobs to run in parallel across many compute cores.  The HPC Pack model is designed to enable exponentially more computational resources per each additional license.  A table summarizing this relationship is shown below.

|HPC Pack Licenses Used	| Maximum Cores Enabled|
|-----------------------|----------------------|
|1	                    |8                     |
|2	                    |32                    |
|3	                    |128                   |
|4	                    |512                   |

Additionally, a number of ANSYS products allow you to use up to four cores without consuming any of the HPC Pack licenses.  For example, a Mechanical or Fluent job can be run on four cores and consume only the underlying physics license.  When scaling these jobs to more than four cores, the four cores are added to the total amount made available by the HPC Pack licenses. For example, a Mechanical or Fluent batch job designed to completely utilize an Eagle compute node (36 cores) requires one physics license and two HPC Pack licenses (4 + 32 cores enabled).

## Running ANSYS in Parallel Batch Mode

To initiate an ANSYS run that uses the HPC Packs, it is necessary to create a command line that contains the hosts and number of processes on each in a format `host1:ppn_host1:host2:ppn_host2:....`. In order to do this as illustrated below, you must set `--ntasks-per-node` and `--nodes` in your Slurm header. A partial example submit script might look as follows.

???+ example "Example ANSYS Submission Script"
    ```bash
    #!/bin/bash -l
    ...
    #SBATCH --nodes=2
    #SBATCH --ntasks-per-node=36
    ...
    cd $SLURM_SUBMIT_DIR
    module purge  # purge everything else
    module load ansys/19.2
    module load intel-mpi/2018.0.3
    ...
    unset I_MPI_PMI_LIBRARY
    machines=$(srun hostname | sort | uniq -c | awk '{print $2 ":" $1}' | paste -s -d ":" -)
    ...
    ansys192 -dis -mpi intelmpi -machines $machines -i <input>.dat
    ```
## Running Fluent in Parallel Batch Mode
The procedure to launch ANSYS Fluent jobs in parallel batch mode follows a similar pattern to the ANSYS example shown above, but with a few critical differences.  An example submit script might look as follows.

???+ example "Example Fluent Submission Script"
    ```bash
    #!/bin/bash -l
    ...
    #SBATCH --nodes=2
    #SBATCH --ntasks-per-node=36
    ...
    cd $SLURM_SUBMIT_DIR
    module purge  # purge everything else
    module load ansys/19.2
    module load intel-mpi/2018.0.3
    ...
    unset I_MPI_PMI_LIBRARY
    srun hostname -s | sort -V > myhosts.txt
    ...
    fluent 2ddp -g -t $SLURM_NTASKS -cnf=myhosts.txt -mpi=intel -pinfiniband -i input_file.jou
    ```
where `2ddp` can be replaced with the version of FLUENT your job requires (`2d`, `3d`, `2ddp`, or `3ddp`), `-g` specifies that the job should run without the GUI, `-t` specifies the number of processors to use (in this example, 2 x 36 processors), `-cnf` specifies the hosts file (the list of nodes allocated to this job), `-mpi` and `-p<...>` specify the MPI implementation and interconnect, respectively, and`-i` is used to specify the job input file.  Note the alternative formatting of the host names when compared with the ANSYS example.

##Contact
For information about accessing licenses beyond CSC's base capability, please contact [Emily Cousineau.](mailto://Emily.Cousineau@nrel.gov)