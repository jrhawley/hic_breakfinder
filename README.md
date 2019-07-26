# Breakfinder

## Dependencies

For hic_breakfinder to run, you will need to have the `eigen` C++ library and the `bamtools` libraries installed.

Eigen can be found [here](http://eigen.tuxfamily.org/index.php?title=Main_Page) and bamtools can be found [here](https://github.com/pezmaster31/bamtools).

After installation, make sure to add the `bamtools` libraries to your `LD_LIBRARY_PATH`.
For instance, if you are running bash, you can modify the `.bashrc` file in your home directory to include the line:

```shell
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/path/to/bamtools/lib"
```

## Installation

To install, run:

```shell
./configure [options]
make
make install
```

If you want to install somewhere that isn't `/usr/local/bin/` run:

```shell
./configure --prefix=/path/to/install/directory
```

This will create the `hic_breakfinder` executable in `/path/to/install/directory/bin/`.

If your version of `eigen` is installed in a non-traditional location, you will need to run configure as follows:

```shell
./configure CPPFLAGS="-I /path/to/eigen"
```

If your version of `bamtools is installed in a non-traditional location, you will need to run configure as follows:

```shell
./configure CPPFLAGS="-I /path/to/bamtools/include/" LDFLAGS="-L/path/to/bamtools/lib/"
```

If both `eigen` and `bamtools` are installed in non-traditional locations, you will need to run the configure file as follows:

```shell
./configure CPPFLAGS="-I /path/to/bamtools/include -I /path/to/eigen" LDFLAGS="-L/path/to/bamtools/lib/"
```

After installation, the `hic_breakfinder` executable should be stored in the bin directory.

## Usage

```shell
hic_breakfinder --bam-file <BAM> --exp-file-inter <INTER> --exp-file-intra <INTRA> --name <PREFIX>
```

It will require 3 input files, a BAM file, an inter-chromosomal expectation file, and an intra-chromosomal expectation file.
We strongly recommend using the b38 build of the human genome for using Hi-C to identify structural variants.  

A note on the BAM file: `hic_breakfinder` will expect to find pair information for each read in the BAM file.
It will also expect that the data will have already been filtered to remove low quality alignments.
We have listed several scripts that can be used to align/post-process Hi-C data in the [`bwa_mem_hic_aligner` repository](https://github.com/dixonlab/bwa_mem_hic_aligner).

We have provided these expectation files in hg38 coordinates for users.
These are in the associated Box files link below.

`hic_breakfinder` has the option of being run at 1kb final resolution.
For this, add the option `--min-1kb` when executing the command.  
For 1kb resolution, the Hi-C experiment should be performed with frequent cutting enzymes like MboI and DpnII.
Using enzymes that cut less frequently than 1kb (i.e. HindIII or NcoI) will lead to problems if trying to identify breaks at 1kb.
Further, if the library has high sequence coverage (>100 million reads), using `--min-1kb` can substantially slow down the run time.

If you want to check that the software is running correctly, we have a BAM file from K562 and an associated `example_output.txt` file showing the results of running `hic_breakfinder` using the `--min-1kb` command on this BAM file in the associated files link.

For any questions/comments/concerns, please email jedixon@salk.edu

Associated files:

* [Box link](https://salkinstitute.box.com/s/m8oyv2ypf8o3kcdsybzcmrpg032xnrgx)

## Output

Hi-C breakfinder will produce lists of structural variant predictions at different resolutions (bin sizes).
The final output file will be named `$name.breaks.txt` (where `$name` is the parameter that is given using the `--name` argument).
It will also produce a series of intermediate calls at different resolutions (1Mb, 100kb, 10kb, optional 1kb for inter-chromosomal events, 100kb, 10kb, optional 1kb for interchromosomal events).
These files are labelled as `$name\_10kb.breaks.txt` or `$name\_10kb_intra.breaks.txt` (for the case of the 10kb calls).

The first line in the example_output.txt file looks like this:

```TSV
90.3714	chr13	93252000	93363000	-	chr9	131176000	131280000	+	1kb
```

Hi-C breakfinder aims to find sub-matrices of the original matrix that are most likely containing a structural rearrangement.
Therefore, each prediction represents a sub-matrix, reporting the positions of the column and row coordinates of the sub-matrix.

Here we will describe the meaning of column of the line:

1. Log-odds score of the rearrangement call. This is can be thought of as the "strength" of the call.
2. Column chromosome
3. Column start
4. Column end
5. Column strand
6. Row chromosome
7. Row start
8. Row end
9. Row strand
10. Resolution of the call (minimal bin size for which this call is made).

The strand predictions are mean to be estimates of whether the rearrangement breakpoint is at a given edge of the sub-matrix.
A `+` value means that we predict the `end` coordinate to be closests to the actually breakpoint, and a `-` value indicates we believe the `start` coordinate is closest to the breakpoint.
 