# tflite-micro-lib
TensorFlow Lite micro C/C++ library that can be easily integrated in your embedded project:  
just clone the library (`git clone https://github.com/biagiom/tflite-micro-lib`) and include it in your project.

This library is based on TensorFlow v2.2 and has been successfully tested on STM32 boards.  
For more information about TensorFlow Lite for microcontrollers project see the [official documentation](https://www.tensorflow.org/lite/microcontrollers).  
Moreover, for more information about how to build from scratch this library and integrate it in a
STM32CubeIDE project see the instructions described in the follow.

## Build the library and include it in your STM32CubeIDE project
* Clone TensorFlow source:
  ```
  git clone https://github.com/tensorflow/tensorflow
  cd tensorflow
  ```
  If you have already cloned tensorflow source, just update it:
  ```
  git pull
  ```
  
* Build the `hello_world` example for `mbed` with support of CMSIS-NN:
  ```
  make -f tensorflow/lite/micro/tools/make/Makefile TARGET=mbed TAGS=cmsis-nn generate_hello_world_mbed_project
  ```
  In order to clean your workspace, just run the following commands before building the `hello_world` example:
  ```
  make -f tensorflow/lite/micro/tools/make/Makefile clean
  make -f tensorflow/lite/micro/tools/make/Makefile clean_downloads
  ```
  
* Create a new directory named `TensorFlow` that will hold the library source files (it will be created in your home directory):
  ```
  mkdir ~/TensorFlow
  ```

* Move into output build directory:
  ```
  cd tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/hello_world/mbed/
  ```

* Make sure you have installed mbed-cli (if not, run `sudo -H pip3 install -U mbed-cli`) and set up the Mbed project by downloading mbed-os:
  ```
  mbed config root .
  mbed deploy
  ```

* Copy the `tensorflow` and `third_party` folders into the library base directory created previously:
  ```
  cp -R tensorflow third_party ~/TensorFlow/
  ```

* Create a new directory for the CMSIS Core header files and move the CMSIS core files shipped with Mbed:
  ```
  mkdir -p ~/TensorFlow/tensorflow/lite/micro/micro/tools/make/downloads/cmsis/CMSIS/Core/Include
  cp ./mbed-os/cmsis/TARGET_CORTEX_M/*.h ~/TensorFlow/tensorflow/lite/micro/micro/tools/make/downloads/cmsis/CMSIS/Core/Include/
  ```
  
  Alternatively, you can also use the CMSIS Core files for Cortex-M that are shipped with a STM32CubeIDE project in `Drivers/CMSIS/Include`, 
  or the CMSIS Core files that are included in the CMSIS source folder that is downloaded during the build of the hello_world project in 
  `tensorflow/lite/micro/tools/make/downloads/cmsis/CMSIS/Core/Include`.

* Move into the library root directory:
  ```
  cd ~/TensorFlow
  ```

* Remove some unneeded directories:
  ```
  rm -r tensorflow/lite/micro/examples
  rm -r tensorflow/lite/micro/mbed
  ```

* Update the definition of `__patched_SXTB16_RORn` in `tensorflow/lite/micro/tools/make/downloads/cmsis/CMSIS/NN/Source/NNSupportFunctions/arm_nn_mat_mult_nt_t_s8.c`:

  Comment the definition of __patched_SXTB16_RORn and add below the following line:  
  `#define __patched_SXTB16_RORn(ARG1, ARG2)   __SXTB16(__ROR(ARG1, ARG2))`  
  This step is needed otherwise the Assembler of the GCC toolchain provided by STM32CubeMX will generate an error.

* Create a new source directory in your STM32CubeIDE project (e.g. TensorFlow) and move the library files 
  (assuming that your STM32CubeIDE workspace folder is `~/STM32CubeIDE_Workspace` and your project is called `TFLite_Hello_World`:
  ```
  cp -R tensorflow ~/STM32CubeIDE_Workspace/TFLite_Hello_World/TensorFlow
  cp -R third_party ~/STM32CubeIDE_Workspace/TFLite_Hello_World/TensorFlow
  ```

* Make sure to include the following directories in the list of paths and symbols (Properties → C/C++ General → Paths and Symbols → Includes) 
  in addition to the existing ones (both for C and C++ languages):
  ```
	${ProjDirPath}/TensorFlow
	${ProjDirPath}/TensorFlow/tensorflow/lite/micro/tools/make/downloads
	${ProjDirPath}/TensorFlow/third_party/flatbuffers/include
	${ProjDirPath}/TensorFlow/third_party/gemmlowp
	${ProjDirPath}/TensorFlow/third_party/ruy
  ```
  
  Moreover, if you plan to use the CMSIS Core included in the library, delete the original path `Drivers/CMSIS/Include` and add the following new path:
  ```
	${ProjDirPath}/TensorFlow/tensorflow/lite/micro/tools/make/downloads/cmsis/CMSIS/Core/Include
  ```

* Finally, in order to make use of CMSIS-NN kernels add the `__ARM_FEATURE_DSP` macro and initialize it to `1 `
  (Properties → C/C++ General → Paths and Symbols → Symbols).
