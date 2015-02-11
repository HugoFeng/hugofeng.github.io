title: Optimizing Pattern Inference of Convolutional Neural Network with OpenCL
date: 2015-01-08 09:31:15
tags: OpenCL CNN
---

### 1. Introduction

As a supervised approach of Deep Learning, Convolutional Neural Network is famous for its noise robustness and translation invariances. It has been widely used to recognize all kinds of patterns, such as hand written digits and human faces. Although Convolutional Neural Network has greatly reduced the computation, comparing with a traditional neural network with the same amount of connections, there still remains a lot of space for parallel optimizations.

Here we introduce an implementation of parallelized LeNet-5 inference with OpenCL. The code for this project is hosted on https://github.com/HugoFeng/convnet .

### 2. Pattern Inference of Convolutional Neural Network

We are using LeNet-5, the most famous CNN architecture introduced by Yann LeCun, for pattern inference and training. The test dataset is the MNIST dataset, which contains a total of 70000 hand written digit images. Since we only consider optimizing the inference (feed forward stage) of the network, the training stage will not be mentioned.

![LeNet-5 (modified)](/img/cnn_overview.png)

Above is an illustration of LeNet-5 (has been slightly modified). It has 3 convolution layers (C1, C3, C5), 2 subsampling layers (S2, S4) , 1 fully connected layer (F6) and 1 output layer.

In traditional neural networks, nodes of every layer are presented as a “flat” 1-D array, and nodes from the adjacent layers are fully connected. But in convolutional networks, nodes in Convolution Layer and Subsampling Layer are given “2D+1D” representations (although stored in 1-D array as well). That means each of these layers, has several “feature maps”, in each “feature map”, nodes are organized in a 2D images fashion, in other words, each node in a layer is identified by (column index, row index, feature map index). 

Connections between a Convolution Layer and Subsampling Layer do not cross over, for example, each feature map in S2 only connects to its corresponding feature map in C1, and it sub-samples the feature maps in C1 with 2*2 to 1 with a “max” function. In the process of sub-sampling, the same 2*2 weight array is used over the input with respect to each feature map. 

![Activation](/img/cnn_activation.png)

Connections between a Subsampling Layer and a following Convolution Layer is more complicated.  As is shown above, each feature map in Convolution Layer is related to inputs from the areas of the same position in several feature maps. In the original definition of LeNet-5, the corresponding input feature map is not all of the maps but only some of them which may refer to a table given by Yann LeCun, but here we choose to connect all the input feature maps for simplicity.

The Fully connected Layer is flat as traditional neural networks are, and it has fully connection with the input Convolution Network.

The Output layer has 10 output nodes, and each node represents a label, which is one of the 10 digits. And the inference is computed as the node label with the largest output value.

### 3. Performance Analysis of serial implementation

#### 3.1 Hardware Specs

CPU Name: Intel Core i5-4250U Processor
Number of Cores: 2
Number of Threads: 4
Processor Base Frequency:  1.3 GHz
Max Turbo Frequency:  2.6 GHz
Number of Memory Channels: 2
Max Memory Bandwidth: 25.6 GB/s


#### 3.2 Profiling

Test with inference 100 samples:

![Test1](/img/cnn_test1.png)

It is very obvious that the `forward` method of Class `ConvolutionalLayer` is consuming 49562/50169 = 98.79% of `test` function call’s time. So working on optimizing the method `forward` in class `ConvolutionalLayer` is the best choice.

### 4. Performance Analysis of parallel optimization

#### 4.1 Hardware Specs

The first device is called Intel(R) HD Graphics 5000
There is a read-write cache
The global mem cache size is 2097152
The local memory size is 65536
The maximum work group size is: 512
The maximum number of dimensions is: 3
The maximum number of items in dimension 0 is 512
The maximum number of items in dimension 1 is 512
The maximum number of items in dimension 2 is 512

Number of compute units:       40
Clock frequency in MHz :       200(lowest), 1000(highest)
Max amount of memory in bytes: 373712486
Memory Type:        DDR3
Bus Width:  128 Bit
Memory Bandwidth: 25.6 GB/s
Floating-point performance:     704 GFLOPS

### 5. Strategies of Optimizations 

As has been discussed above, the most time consuming function is `forward` in class `ConvolutionalLayer`. So our goal is to minimize the time cost by this function.

In my project, I have implemented 3 different approaches for parallelizing the `forward` method in class `ConvolutionalLayer`. 

* forward_parallel: accepts one input samples at a time, which is the same as the CPU serial version. It hides all OpenCL implementations inside this function, so that codes in all the classes don’t have to make any change to be compatible with is parallelized function. User can use a macro `GPU` to switch between the original CPU `forward` implementation and the GPU “forward_parallel”.

    * Lack of parallel data access: The number and topology of work items is defined by the problem, which means we cannot increase work items.

![Topology of Work items1](/img/cnn_topo1.png)

* forward_batch: accepts a batch of input samples at once, and codes in other classes are modified slightly to work with the batch operation. 

    * The problem size of each kernel can be increased by changing the batch size.

![Topology of Work items2](/img/cnn_topo2.png)

* forward_batch_more: acts like `forward_batch`, but each thread works for output node in several samples at the same location. As is shown below, each work item works for 2 nodes at the same location but within 2 different samples. Since nodes at the same location belonging to different samples of a batch are sharing the same set of weights, so each item will only need to load one set of weights for the inputs and applied them on the inputs of several batches. By doing so, it saves bandwidth and increases the Computational Intensity for a certain amount. Furthermore, the convolution procedure is re-coded together with the iteration of loading the input sub-matrix into private registers, so that the convolution is computed during the loading, which saves one inner loop per input per sample per output node. And the number of samples one item should deal with is defined as a macro `THREAD_TASKS`, which can be changed.

![Topology of Work items3](/img/cnn_topo3.png)

#### 5.1 Roofline model Analysis

1.  “forward_parallel”: 

    a) CI = 2.625

    b) Peak performance = 2.625 * 25.6 = 67.2 GFLOPS

2.  “forward_batch”: 

    a) CI = 3.2

    b) Peak performance = 3.2 * 25.6 = 81.92 GFLOPS

3.  “forward_batch_more”: 

    a) {% math CI = \frac{9⁄T+15.8}{1⁄T+1⁄(indepth*25)+1} * \frac{1}{4} < 3.95 %}, (T=THREAD_TASKS, as the node/batch numbers a work item should deal with)
    
    b) Peak performance = 3.95 * 25.6 = 101.12 GFLOPS 

![Roofline model](/img/cnn_roofline.png)

Since the first implementation has a low computational intensity and fell into memory bandwidth limitation, I designed the second and the third implementation trying to raise the computational intensity, but a CI of 3.7 is so far the best I can get while a CI of 3.95 is theoretically reachable with an infinitely large T (THREAD_TASKS). It’s still far from CI of 27.5, which will get rid of memory bandwidth limitations.

### 6. Profiling the kernels

We choose the second Convolution Layer to profile, which has the following parameters:

* Input depth: 6, height: 14, width: 14

* Output depth: 16, height: 10, width: 10

![Profile result](/img/cnn_table.png)

![Time & speed up](/img/cnn_best.png)

The first implementation, `forward_parallel`, operates on one sample per kernel, and it maps each work item to each output pixel, so the number of work items the kernel launches is limited by the size of the output feature maps.

In order to raise the number of threads each kernel launches and make the kernel scalable, the second implementation is designed. It groups a certain amount of samples together as a batch. Therefore, multiple samples can be computed with a single kernel launch, and according to the batch size, the thread number can be greatly increased. By doing so, the kernel has raised the speed up from 47.61 to 63.3, which is not dramatic since the actual work of each work item remains the same.

In the third implementation, the work of each work item is modified so that each work item can work on multiple samples. And since each work item works on the pixel of the same location of each sample, the pixels share the same set of weight matrix. So each work item only need to load the weight matrix once and it is loaded to the private memory. 

![kernel time](/img/cnn_time_kernel.png)

By saving bandwidth loading weights, `forward_batch_more` kernel can gain a speed up of approximately 2 times comparing to `forward_batch` kernel.

![kernel time w.r.t. THREAD_TASKS](/img/cnn_time_kernel_t.png)

However, by increasing the THREAD_TASKS, which is the number of samples each work item works on, the work load of each work item is also increased. Due to the pool performance of the cores on GPU, adding too much serial work load on each thread is not a good idea. Experiments shows that with `THREAD_TASKS` set to 3, which means each work item works on 3 samples, the kernel can reach the best performance on the problem set of the second Convolutional Layer.

Besides, whether using `forward_batch` or `forward_batch_more` kernel, there doesn’t seem to be much difference when considering the whole task, say testing the inference with 1000 samples with the feed forward convolutional neural network. Instead, batch size seems to have more influence than the kernels themselves.

![time and sample](/img/cnn_time_sample.png)

Profiling the whole program shows why this would happen. Using “Performance and Diagnose” tool in Visual Studio 2013, it tells how much time is spent on each function, but it is worth to point out that using this tool requires target to be built with `Debug` schema, instead of `Release` schema with which the previous analysis is tested. So the exact time reported here might be not accurate and trustworthy, but the percentage data is useful. 

![test2](/img/cnn_test2.png)

By profiling the task of testing 1000 samples, it shows that with the power of GPU, the function that calls OpenCL kernel only counts a very small part of the whole time consumption, while the `forward_batch` functions in class `FullyConnectedLayer` and class `MaxpoolingLayer` is taking most of the time. This explains why there doesn’t seem to be a difference changing the kernel. On the other hand, this indicates that on top of the speed up we have achieved, if the `forward_batch` function of the other 2 layers are optimized, more speed up is still possible.

![time all](/img/cnn_time_all.png)

Back to the whole picture, built with `Release` schema, code with OpenCL implementation runs a lot faster than the serial-only code, which gains a speed up of 34215/293.4 = 116.6 times.

### 7. Conclusion

This report introduced three implementations of optimizing the inference part of a Convolutional Neural Network with OpenCL technology. The speed up of the specific function (function `forward`) reached 151.78 times while the speed up of the whole inference process reached 116.6 times. 

--------------

### Appendix

#### Setting up the project

Requirements: 

* Visual Studio 2013, Xcode, or Visual Studio 2012 ( need to add some extra code ). VS2010 is not supported.
* Git
* Boost library ( only need header files, not binaries)
* MNIST dataset ( will be attached with the code)

1)  Clone the `opencl_support` branch and `profile-parallel` of the repo on https://github.com/HugoFeng/convnet, or just unzip my attachment zip file.

* git clone https://github.com/HugoFeng/convnet.git
* git checkout profile-parallel origin/profile-parallel

2)  In the `convnet/convnet/` directory, if file `settings.h` doesn’t exits, you need to create it manually, and add the following snippet to it. (Change the directories to your own path)

~~~ C
#define GPU
//#define PROFILING
//#define SIMPLE_PARALLEL
//#define BATCH_MORE
#define THREAD_TASKS 3
//#define CHECK_RESULT
#define DATA_PATH "E:/code/data/mnist/"
#define KERNEL_PATH "E:/code/cpp/convnet/convnet/kernels.ocl"
~~~ C
If you are using VS2012, it’s possible later the compiler complains that the type `float_t` is not defined, then the following code is also needed to add into `settings.h`

~~~ C
namespace std {
    typedef float float_t;
}
typedef float float_t;
~~~

3) Create folder `convnet/build`, and create a blank VS C++ project inside the folder. Then drag all source files inside `convnet/convnet` except `kernels.ocl` into the project. 

4) In property setting of the project, add the directory of boost header files, and the headers directory for OpenCL to C/C++ “include directories”.

5) Add OpenCL lib directory (with the specific platform folder) to the “additional lib search path”, and add `OpenCL.lib` to the linking libs explicitly.

6) For better performance, please use `Release` scheme to build.

#### Macro explanations

~~~ C
#define GPU
//#define PROFILING
//#define SIMPLE_PARALLEL
//#define BATCH_MORE
#define THREAD_TASKS 3
//#define CHECK_RESULT
~~~

Above are the macros defined inside `settings.h` file. The `settings.h` files is added to the `.gitignore` list so that each commit is platform and settings independent.

* If `GPU` is NOT defined, the project will only use CPU code, and the rest of the macros will not take effect on anything, while the last parameter (batch size) of the `n.test` function call should be `1` since CPU operations don’t take batch inputs.
* If `PROFILING` is defined, the kernels in each `forward_batch` or `forward_batch_more` functions of class `ConvlutionalLayer` will be iterated multiple times (100 times) and print out profiling info about the kernels. Before doing so, inside `main.cpp`, the `test_sample_count` and the last parameter of the `n.test` function call should be consistent, so that the program will only go forward through the network once in the testing process.
* `SIMPLE_PARALLEL` means using `forward_gpu` for testing process, and the batch size should set to `1`. It only takes effect in branch `profile-parallel`.
* If `BATCH_MORE` is defined, then `forward_batch_more` is used, otherwise, `forward_batch` will be used. If `BATCH_MORE` is defined, one MUST make sure that the `THREAD_TASKS` macro defined in `kernels.ocl` file on line 103 is consistent with the macro defined. They must have the same value to compute correctly.
* If `CHECK_RESULT` is defined, the result will be checked after each forward feeding process is finished. Notice that this only support checking result for batch operation outputs, which means when `GPU` is undefined or `SIMPLE_PARALLEL` is defined, this macro MUST be commented out.


















