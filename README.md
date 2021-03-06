[drvq](http://image.ntua.gr/iva/tools/drvq)
====

*dimensionality-recursive vector quantization*

`drvq` is a C++ library implementation of [dimensionality-recursive vector quantization](http://image.ntua.gr/iva/research/drvq), a fast vector quantization method in high-dimensional Euclidean spaces under arbitrary data distributions. Usage of the software is only described here; for more information on the methods used please refer to the [DRVQ software page](http://image.ntua.gr/iva/tools/drvq), the [research project](http://image.ntua.gr/iva/research/drvq), or the original publication ([abstract](http://image.ntua.gr/iva/publications/qc), [PDF](http://image.ntua.gr/iva/files/qc.pdf)). The [latest stable release](https://sourceforge.net/projects/drvq/files/) is available for download at [SourceForge](https://sourceforge.net/projects/drvq/), and the latest development version is at [github](https://github.com/iavr/drvq). This README file is better viewed online [here](https://github.com/iavr/drvq/blob/master/README.md).

Licence
-------

`drvq` has a 2-clause BSD license. See file [LICENSE](/LICENSE) for the complete license text.

Directory structure
-------------------

`drvq` library constists primarily of template C++ header-only (`.hpp`) code, but also includes a number of command-line tools to test the library, provided as `.cpp` files. The directory structure is:

	/data      sample data files for testing
	/matlab    binary file input/output tools for matlab
	/out       output folder for drvq tools
	/run       simple scripts to compile and execute
	/src       drvq library and tools source code

Requirements
------------

`drvq` implementation is based on C++ template library [ivl](http://image.ntua.gr/ivl/), which may be downloaded from [sourceforge.net](http://sourceforge.net/projects/ivl/files/) and [installed](http://image.ntua.gr/ivl/download.php) on a number of platforms.

To avoid future instabilities, prefer [version 0.9.2]( http://sourceforge.net/projects/ivl/files/src/ivl-0.9.2-src.tar.gz) that was used during `drvq` development.

Because most of `ivl` consists of template header-only code and no modules are used in `drvq`, you don't even need to install `ivl`; just download and adjust the `ivl` local folder in the scripts described below.

Installation, building
----------------------

`drvq` has only been tested on [clang 3.3](http://llvm.org/releases/download.html#3.3) and [g++ 4.8.1](http://gcc.gnu.org/gcc-4.8/) on Linux, but it should be straightfoward to use on other platforms.

No installation is needed or provided for `drvq`. Command-line tools under folder [/src/](/src/) illustrate how the library may be used and may be compiled with minimal options without a Makefile, and scripts under [/run/](/run/) show how a tool may be compiled and executed.

Script [/run/run](/run/run) is for GCC and [/run/lrun](/run/lrun) for Clang. It is best to copy them in a folder in your path, e.g. `/usr/local/bin`, after adjusting folders for your local copy of `ivl`. In this case, starting from the folder where `drvq` resides,

	cd src/
	run train

compiles file [/src/train.cpp](/src/train.cpp), produces binary `/src/train`, and executes it unless there are compiler or linker errors. If `ivl` is properly installed, compilation may be even simpler, without specifying the include folder with `-I`.

Extension `.cpp` is not necessary for the main source file. Additional source files or compiler options may be specified as additional command-line arguments, but extensions are needed for extra source files.

Binary file format
------------------

All input/output to all `drvq` tools is based on a binary file format, using a file extension `.bin`. With the exception of codebooks (file `codebook.bin`) produced by tool `train` and used by other tools, all other input/output files each contain an array of arrays of numbers of any type.

Such files may be read/written by Matlab scripts [load_double_array.m](/matlab/load_double_array.m) / [save_double_array.m](/matlab/save_double_array.m) under folder [/matlab/](/matlab/). An array of arrays is is represented by a two-dimensional matrix in Matlab, but any Matlab data type may be used. These scripts are very simple and may serve as a specification for input/output tools in other platforms. For C++, input/output is handled by template functions under [/src/lib](/src/lib).

Sample data
-----------

With the specific information provided per tool below, one should be able to generate appropriate input and read the output of each tool. For convenience, a number of sample data files are provided under folder [/data/](/data/) so that all `drvq` tools may run immediately.

These files are part of the actual experimental data that were used during development. In particular, under [/data/oxford/hesaff.sift/queries/descriptors/](/data/oxford/hesaff.sift/queries/descriptors/), SIFT descriptors are given for each of the 55 cropped query images of [Oxford buildings dataset](http://www.robots.ox.ac.uk/~vgg/data/oxbuildings/), as obtained by [Hessian-affine detector](https://github.com/perdoch/hesaff). File [/data/oxford/files/queries.txt](/data/oxford/files/queries.txt) contains a relevant list of filenames.

For each image, a binary file contains an array of descriptors, where each descriptor is represented by an array of `float`s. A file of the same format can be easily generated by [load_double_array.m](/matlab/load_double_array.m) given a two-dimensional matrix of `float`s in Matlab, where rows (columns) correspond to local features (descriptor elements/dimensions).

Tools
-----

All tools are provided as very simple, script-like `.cpp` files of few lines of code each, with all low-level implementation hidden in corresponding `.hpp` or other included files. Each `.cpp` file has a `main()` function and generates a command-line executable that can be used as a stand-alone tool for a particular job, using a rich set of command-line options.

On the other hand, the source code illustrates how the library can be used, e.g. to integrate in other programs. One is welcome to tune the code beyond the options offered by the tools, including the data types used. This is straightforward because the implementation is using templates or type definitions where necessary. E.g. the type of all vector elements is defined as `float` at the very beginning of the `.cpp` file of each tool, and is template parameter `T` anywhere else in the library; and the type of different integer labels is defined as `lab` and `pos` in [/src/search+/tree.hpp](/src/search+/tree.hpp).

The usage of each tool can be seen by option `-h` or `--help`, including a short description of each command-line option. Some additional documentation is provided below. In all cases, hard-coded defaults enable a "demo" operation on the sample data with no options given at all.

### `train`

Specified by [train.cpp](/src/train.cpp). Reads a set of input training data like those provided under [/data/](/data/), representing a set of points (vectors) in a Euclidean space; trains one or more codebooks, and saves the codebooks in a custom binary format that is to be read by other tools. The codebooks are internally structured as binary trees over the dimensions of the space and contain additional data like lookup tables to be used for fast encoding (quantization) of new data.

The first few options refer to input/output files, paths, etc. and are rather self-explanatory. Option `--dataset` acts like a preset for other such options. All "presets" are hard-coded in [/src/options/data.hpp](/src/options/data.hpp) are can be freely changed in the source code, but each option can also be overriden. Option `--unit` is only used when a large dataset is split into individual units (chunks) of files under different folders.

Because all experiments carried out refer to descriptors of images, all data are assumed to reside in individual files, one for each image. In a different application, all data might be in a single file; in this case, the input file list provided by option `--list` would only contain a single filename.

For the same reason, although the method supports arbitrary dimensions, only two alternatives are provided here, controlled by `--descriptor`: 64 dimensions for SURF descriptors, or 128 dimensions for SIFT descriptors. Each vector may be used as a whole, producing a single codebook, or split into sub-vectors, producing more codebooks. This is controlled by `--books`, supporting 1, 2, or 4 codebooks. E.g. SIFT vectors of 128 dimensions and 4 codebooks result in 4 sub-vectors of 32 dimensions each.

Because the method is hierarchical in the number of dimensions, there is no single parameter for the target codebook size (number of centroids), but rather one such target for each level of the hierarchy. This is controlled by "preset" codebook sizes (capacities) specified in [/src/options/train.hpp](/src/options/train.hpp) and controlled by option `--capacity`. E.g. the default setting `2` for SIFT and 4 codebooks corresponds to entry

	5, 6, 7, 8, 9, 11

which represents the codebook size per number of dimensions, with both expressed as powers of two. E.g. `2^5 = 32` centroids for `2^0 = 1` dimension, `2^6 = 64` centroids for `2^1 = 2` dimensions and so on; finally, `2^11 = 2048` centroids for `2^6 = 32` dimensions. One may freely manipulate these presets for a different application, but codebook size should generally increase with dimension, and should not increase too much beyond `2^12` or training will take too much space and time.

Termination of training takes into account both progress towards convergence and the actual number of iterations so far. It is controlled by a single parameter `--theta`. The value should be positive; a lower value results in longer training.

### `flat`

Specified by [flat.cpp](/src/flat.cpp). Reads a codebook file generated by `train`; "flattens" and exports centroids in a format that can be read e.g. by [load_double_array.m](/matlab/load_double_array.m) as a two-dimensional matrix in Matlab. This is useful because codebooks produced by `train` can otherwise only be read by the `drvq` library and tools, which is due to the fact that a custom binary file format is used to represent the hierarchical codebook structure and additional data. `flat` only exports the codebook centroids, which can then be used in any application but without the fast encoding capabilities of `drvq`.

If a single codebook is used, an array of centroids is generated, with each centroid represented as an array of `float`s. In the case of multiple codebooks, the format and size of the generated file is identical, but different parts of the data refer to different sub-codebooks. E.g. with SIFT vectors of length 128 and 4 codebooks, there is again an array of vectors, but now each vector of length 128 represents 4 concatenated centroids of length 32 each; each centroid belongs to a different codebook.

There are no options for `flat` apart from those specifying the input/output files.

### `label`

Specified by [label.cpp](/src/label.cpp). Reads a codebook file obtained by `train` and a a set of input data like those provided under [/data/](/data/), representing a set of points (vectors) in a Euclidean space exactly as for `train`; encodes each given point according to the codebook(s) and saves the resulting integer label for each point.

The first few options refer to input/output files, paths, etc. exactly as for `train` and are rather self-explanatory. The "demo" operation with no options actually runs on the same data as for `train`; in a real application, a different set of data would be provided.

A single output file is generated, containing an array of vectors; each vector corresponds to one input file (e.g. to one image for our samples in [/data/](/data/)) and each vector element is a long unsigned integer (`size_t`) that represents one encoded data point.

Given a new data point (vector) and a single codebook, encoding amounts to finding the nearest centroid and using the integer id of this centroid as a label. In the case of multiple codebooks, the vector is split into sub-vectors and for each sub-vector the nearest sub-centroid is found from the corresponding sub-codebook. The resulting labels are then encoded into a single integer. E.g. with SIFT vectors of length 128 and 4 codebooks, there are 4 labels, say l0, l1, l2, l3. These are encoded as

	k^3 * l3 + k^2 *l2 + k * l1 + l0

where k is the size of the sub-codebooks. On a 64-bit machine, k can be up to `2^16` for this to fit into a single integer.

Finding the nearest centroid is a nearest-neighbor search problem. Given the data provided in the codebook, three methods are supported, as controlled by `--method`: `fast`, `approx`, and `exact`. `approx` is only experimental and does not really offer any benefit over `exact`, because it takes roughly the same time. `fast` is again approximate, based on lookup operations, and really fast but not very precise. It is the method that is used during training. `exact` is a lot slower but it is preferable in a real applications where performance matters.

### `nn`

Specified by [nn.cpp](/src/nn.cpp). Reads a codebook file obtained by `train` and a a set of input data like those provided under [/data/](/data/), representing a set of points (vectors) in a Euclidean space exactly as for `train`; performs an evaluation of the different nearest neighbor search methods of tool `label` and prints a set of measurements.

It carries out search by naive, exhaustive computation, which is the slowest, verifies  correctness of method `exact`, and measures the precision of approximate methods `fast`, `approx` in a number of ways. Timings are provided for all methods. A set of input files is used for evaluation by default, and all measurements are averaged over all given query points (vectors).

With the default options and provided data, `exact` is roughly `10x` faster than naive, exhaustive computation, and `fast` is roughly `1000x` faster than `exact`.

The first few options refer to input/output files, paths, etc. exactly as for `train` and `label`. Option `--query` can be specified to choose a single input file by its id from the given list of files; otherwise, all input files are considered. The remaining options are specific to the provided measurements and are rather self-explanatory.

Citation
--------

Please cite the following paper if you use this software.

Yannis Avrithis. [Quantize and Conquer: A dimensionality-recursive solution to clustering, vector quantization, and image retrieval](http://image.ntua.gr/iva/publications/qc). In Proceedings of International Conference on Computer Vision ([ICCV 2013](http://www.iccv2013.org/)), December 2013.
