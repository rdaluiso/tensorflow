Tensorflow for PDG
==================

Here are miscellaneous notes for the usage of Tensorflow in PDG C++ projects.

Build Tensorflow
----------------

Merging suggestions from the sources:
- https://www.tensorflow.org/install/source_windows
- https://github.com/sitting-duck/stuff/tree/master/ai/tensorflow/build_tensorflow_1.14_source_for_Windows

The main steps to build on Windows follow:

1. Install the Visual Studio 2019 Redistributables and Build Tools from https://visualstudio.microsoft.com/downloads/.
2. Open in this folder an MSYS2 shell configured to use that version of Visual Studio,
   and configure it to disable conflicting expansions:
   ```
   export MSYS_NO_PATHCONV=1
   export MSYS2_ARG_CONV_EXCL="*"
   ```
3. Create a Python virtual environment and activate it:
   ```
   virtualenv venv
   . ./venv/Scripts/activate
   ```
4. Install the main Python dependencies:
   ```
   pip install six numpy wheel
   pip install keras_applications==1.0.6 --no-deps
   pip install keras_preprocessing==1.0.5 --no-deps
   ```
5. Download Bazel from https://bazel.build/ and put it with the name `bazel.exe` somewhere in the path:
   ```
   bazel version
   ```
6. Configure the build (all defaults should be OK):
   ```
   python ./configure.py
   ```
7. Launch a build of the dll and lib:
   ```
   bazel build --config=opt //tensorflow:tensorflow.dll
   bazel build --config=opt //tensorflow:tensorflow.lib
   ```
8. The output library files are in `bazel-bin/tensorflow`.

Link an executable
------------------

This is no easy business. Here are the main approaches tried with success:

1. Compile your executable within the Bazel build system; you will still have to list the dependencies as done e.g. in `tensorflow\examples\label_image\BUILD`.

2. Use another build system, add the include paths you need (including maybe some machine generated includes under `bazel-tensorflow`), and then link against `bazel-bin/tensorflow/tensorflow.lib`.
   You will almost surely get link errors, to be fixed by adding the macro `TF_EXPORT` from `tensorflow/core/platform/macros.h` to the needed symbols and rebuilding tensorflow.dll.
   Note that sometimes this will involve touching code generators, see e.g. my branch tf_export.
