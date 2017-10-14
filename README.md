# Tensorflow-MKL-Mac

Tensorflow (Intel MKL-DNN 2018) for Mac
====================================

Intel Math Kernel Library (MKL) for Intel based systems accelerate math processing routines, increase application performance, and reduce development time. This ready-to-use math library includes:

Linear Algebra | Fast Fourier Transforms (FFT) | Deep Neural Networks | Vector Statistics & Data Fitting | Vector Math & Miscellaneous Solvers

This is my first successful build of Tensorflow which has integrated with MKL-DNN 2018 Initial Release. You can download the compiled pip wheel file **(.whl)** through **Release** section. This build has enabled AVX, SSE4 features on Intel CPU.

*Please remind that this build is **NOT** for **CUDA GPU**. The intention to utilize **Intel MKL** is to accelerate **Intel Core i5 (Haswell) or above CPU (not GPU)** on Mac computer which has well-known limit of OpenCL support on its integrated Graphic processor (not even has **SYCL/ComputeCPP** support for Mac at the moment).*

For OpenCL 1.2 support release of Tensorflow, please see https://github.com/hughperkins/tf-coriander.

**Here's the build instruction:**
---------------------------------

System: Mac OS X 10.12.6

Intel MKL-DNN library: mklml_mac_2018.0.20170908.tgz (from https://github.com/01org/mkl-dnn/releases)

Tensorflow release: 1.3.1 (https://github.com/tensorflow/tensorflow/releases)

Bazel release: 0.5.4 (https://github.com/bazelbuild/bazel/releases)

Installed MKL to /opt/intel/mklml

`$ tar -xvzf mklml_mac_2018.0.20170908.tgz`

`$ mv  mklml_mac_2018.0.20170908 /opt/intel/mklml`

Symlink the two .dylib files (**lib/libmklml.dylib, lib/libiomp5.dylib**) to /usr/local/lib

`$ ln -sf /opt/intel/mklml/lib/libmklml.dylib /usr/local/lib/libmklml.dylib`

`$ ln -sf /opt/intel/mklml/lib/libiomp5.dylib /usr/local/lib/libiomp5.dylib`

Remember to assign proper file ownership of the .dylib links in order to avoid errors at the later stage:

`$ chown $(whoami):staff /usr/local/lib/libmklml.dylib`

`$ chown $(whoami):staff /usr/local/lib/libiomp5.dylib`

Assign correct **'install name'** to the two .dylib symlinks in order to avoid any compiling error:

`$ install_name_tool -id "/opt/intel/mklml/lib/libmklml.dylib" /usr/local/lib/libmklml.dylib`

`$ install_name_tool -id "/opt/intel/mklml/lib/libiomp5.dylib" /usr/local/lib/libiomp5 .dylib`

Download and extract Tensorflow:

`$ wget https://github.com/tensorflow/tensorflow/archive/v1.3.1.tar.gz`

`$ tar -xzvf v1.3.1.tar.gz`

Supposing new folder **tensorflow-1.3.1** has been extracted:

`$ cd tensorflow-1.3.1`

Key changes to edit the file **tensorflow-1.3.1/configure**:
* Change **MKL_ML_LIB_PATH** to **lib/libmklml.dylib**
* Change **MKL_ML_OMP_LIB_PATH** to **lib/libiomp5.dylib**
* Comment out the parts that say "if linux:"
* Comment out the parts that say "Darwin is not supported" and "exit" clause
* Comment out the parts that deal with **libdl.so.2**

Changes to **tensorflow-1.3.1/third_party/mkl/BUILD**:
Change the list of three .so files to the two .dylib files

Start configuring the source: 

`$ ./configure`

Type **Yes** to **use MKL**, **No** to **download MKL**. 
MKL installed separately. Path to MKL: /opt/intel/mklml

Apple builtin LLVM (clang) doesn't support compiler option like **"-fopenmp"** so let's skip checking it by modifying tensorflow/tensorflow.bzl:

1. Find the line:

`]) + if_cuda(["-DGOOGLE_CUDA=1"]) + if_mkl(["-DINTEL_MKL=1", "-fopenmp",]) + if_android_arm(`

2. Then remove the value "-fopenmp" from the statement:

`]) + if_cuda(["-DGOOGLE_CUDA=1"]) + if_mkl(["-DINTEL_MKL=1"]) + if_android_arm(`

Build command: 

`$ cd tensorflow-1.3.1`

`$ bazel build -c opt --config=opt --config=mkl --copt="-DEIGEN_USE_VML" --copt=-mavx --copt=-mavx2 --copt=-mfma --copt=-msse4.1 --copt=-msse4.2 --linkopt="-Wl,-rpath,/opt/intel/mklml/lib" --linkopt="-L/opt/intel/mklml/lib" --linkopt="-lmklml" --linkopt="-iomp5"  //tensorflow/tools/pip_package:build_pip_package`

This build process lasted for more than half hour on my Mac.

Once the build process is completed, let's check the wheel out:

`$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg`

Double check if the file is there:

`$ ls -l /tmp/tensorflow_pkg/tensorflow-1.3.1-cp27-cp27m-macosx_10_12_intel.whl`

Finally, use pip to install the wheel:

`$ pip install --ignore-installed --upgrade /tmp/tensorflow_pkg/tensorflow-1.3.1-cp27-cp27m-macosx_10_12_intel.whl`

After the installation, let's have a fast (10x-50x) implementation of protobuf to boost the performance:

(*This is also the solution to **KeyError: "Couldn't find enum google.protobuf.MethodOptions.IdempotencyLevel"** while running command like "import tensorflow" within python)

For Python 2.7:

`pip install --upgrade \
https://storage.googleapis.com/tensorflow/mac/cpu/protobuf-3.1.0-cp27-none-macosx_10_11_x86_64.whl`

For Python 3.n:

`$ pip3 install --upgrade \
https://storage.googleapis.com/tensorflow/mac/cpu/protobuf-3.1.0-cp35-none-macosx_10_11_x86_64.whl`

To check if the Tensorflow is running properly, try the followings:

`$ python`

```python
>>> import tensorflow as tf
>>>
```

If no error message appears, then Tensorflow is imported successfully in Python.

**Time to start your spiritual work in Deep Learning!**



