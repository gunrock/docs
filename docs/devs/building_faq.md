# Building FAQ

When building Gunrock, you can encounter a variety of issues stemming from your system configuration. These issues can commonly be solved by referencing [this table](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) from NVIDIA's installation guide. While some configurations besides those in the table work quite well, _we will close tickets for build issues whose systems do not meet NVIDIA's installation requirements_. We're happy to discuss potential solutions with you, but it's not our top priority. 

Please take a look at some of these common problems before submitting your compilation error as an issue. 

> I get a CMake Error `Cannot find source file: XXX`.

Ensure you have either cloned gunrock recursively with `git clone --recursive` or initialized the submodules after cloning with `git submodule update --init` in the gunrock root directory. 

> I receive an `undefined reference to main` error when compiling gunrock applications. 

You are likely using an improper compiler version. Inspect the CMake output for the compiler versions that are being used.

```
-- The C compiler identification is GNU XXX
-- The CXX compiler identification is GNU XXX
```

An undefined reference to main typically indicates that the GCC version is too low, so ensure that the $CC and $CXX environment variable are set to a proper compiler using NVIDIA's installation guide [table](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) as a reference.

> My CUDA version autodetection fails with CMake saying `-- GPU architectures auto-detect failed. Will build for all possible architectures.` _or_ I receive a `nvcc fatal   : Unsupported gpu architecture 'compute_30'` error during compilation.

If there's any issue with your system configuration, CMake will fail to detect your compute capability and use all of them for compilation. You can try debugging this autodetection process by running the same file that CMake uses:

```
nvcc --run -ccbin $YOUR_CCBIN_PATH build/autodetect_cuda_archs.cu
```

and examining the error output you receive. Proper output are the `-gencode=arch=compute_XX` strings. 

You can also skip this autodetection process entirely by specifying 

```
cmake .. -DCUDA_AUTODETECT_GENCODE=OFF -DGUNROCK_GENCODE_SM75=ON
```

which may make it easier to debug your system configuration.

> I encounter a `/usr/include/c++/10.1.0/type_traits(1396): error: type name is not allowed` or a similar error in `/usr/include/c++`. 

Any error in `/usr/include/c++` is almost always an issue with your compiler version. Look at the CMake output. Make sure you've set the $CXX environment variable as well as $CC. Issues like `type name is not allowed` often mean your compiler version is too new. See NVIDIA's [installation guide](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) for more information.