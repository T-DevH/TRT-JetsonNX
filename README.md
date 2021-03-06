# TensorRt_demos

TensorRT(TRT) is an SDK for optimizing trained deep learning models to enable high-performance inference. TRT contains a deep learning inference **optimizer** for trained deep learning models and an optimized **runtime** for execution. Once you have trained your deep learning network in any specific framnework, TRT enables you to inference it with higher throughtput and lower latency.

![What is TRT](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/whatistrt2.png)

TRT has two parts:
* **Conversion paths**: The different paths users can follow to convert models to optimized TRT engines
* **Runtime options**: Different runtimes users can target with TRT when deploying the optimized TRT engines.

5 key technologies TRT implements to output an optimized inference server:
* Layer and tensor fusion
* Kernel auto-tuning
* Multi-stream execution
* Dynamic tensor memory
* Precision calibration

![TRT technologies](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/TRTtech.png)


Examples demonstrating how to optimize caffe/tensorflow/darknet models with TensorRT and run inferencing on NVIDIA Jetson NX.

* Run an optimized "yolov4-416" object detector on Jetson NX.
* Run an optimized "yolov3-416" object detector on Jetson NX.
* Run an optimized "ssd_mobilenet_v1_coco" object detector ("trt_ssd_async.py") on Jetson NX.
* Run a very accurate optimized "MTCNN" face detector on Jetson NX.
* Run an optimized "GoogLeNet" image classifier at "~16 ms per image (inference only)" on Jetson Nano.
* yolo3/yolo4 and SSD are evaluated mAP and FPS
* Using INT8 and DLA core
* **To test:** Demos should work on x86_64 platform with NVIDIA GPU(s) as well. Please refer to README_x86.md.


Table of contents
-----------------

* [Prerequisite](#prerequisite)
* [Demo #1: GoogLeNet](#googlenet)
* [Demo #2: MTCNN](#mtcnn)
* [Demo #3: SSD](#ssd)
* [Demo #4: YOLOv3](#yolov3)
* [Demo #5: YOLOv4](#yolov4)
* [Demo #6: Using INT8 and DLA core](#int8_and_dla)

<a name="prerequisite"></a>
Prerequisite
------------

The code in this repository was tested on Xavier NX DevKits.  In order to run the demos below, in my experiments I used Jetpack 4.4: [Setting up Jetson Xavier NX](https://github.com/T-DevH/Jetson-NX).

Demo programs in this repository require "cv2" (OpenCV) module for python3.  "cv2" module comeme pre-installed in the JetPack.  

Running Demo #3 (SSD) requires to have "tensorflow-1.x" installed. You could probably use the [official tensorflow wheels provided by NVIDIA](https://docs.nvidia.com/deeplearning/frameworks/pdf/Install-TensorFlow-Jetson-Platform.pdf)

Demo #4 and Demo #5, you'd need to have "protobuf" installed.  I recommend installing "protobuf-3.8.0",[Setting up Jetson Xavier NX](https://github.com/T-DevH/Jetson-NX).  


<a name="googlenet"></a>
Demo #1: GoogLeNet
------------------

This demo illustrates how to convert a prototxt file and a caffemodel file into a TensorRT engine file, and to classify images with the optimized TensorRT engine.

Step-by-step:

1. Clone this repository.

   ```shell
   $ cd ${HOME}/project
   $ git clone https://github.com/jkjung-avt/tensorrt_demos.git
   $ cd tensorrt_demos
   ```

2. Build the TensorRT engine from the pre-trained googlenet (ILSVRC2012) model.  Note that I downloaded the pre-trained model files from [BVLC caffe](https://github.com/BVLC/caffe/tree/master/models/bvlc_googlenet) and have put a copy of all necessary files in this repository.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/googlenet
   $ make
   $ ./create_engine
   ```

3. Build the Cython code. Install Cython if not previously installed.

   ```shell
   $ sudo pip3 install Cython
   $ cd ${HOME}/project/tensorrt_demos
   $ make
   ```

4. Run the "trt_googlenet.py" demo program.  For example, run the demo using a USB webcam (/dev/video0) as the input.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
   $ python3 trt_googlenet.py --usb 0 --width 1280 --height 720
   ```

   Here's a screenshot of the demo.

   ![GoogleNet demo](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/GoogleNet.png)

5. The demo program supports 5 different image/video inputs.  You could do `python3 trt_googlenet.py --help` to read the help messages.  Or more specifically, the following inputs could be specified:

   * `--image test_image.jpg`: an image file, e.g. jpg or png.
   * `--video test_video.mp4`: a video file, e.g. mp4 or ts.  An optional `--video_looping` flag could be enabled if needed.
   * `--usb 0`: USB webcam (/dev/video0).
   * `--rtsp rtsp://admin:123456@192.168.1.1/live.sdp`: RTSP source, e.g. an IP cam.  An optional `--rtsp_latency` argument could be used to adjust the latency setting in this case.
   * `--onboard 0`: Jetson onboard camera.

   In additional, you could use `--width` and `--height` to specify the desired input image size, and use `--do_resize` to force resizing of image/video file source.

   The `--usb`, `--rtsp` and `--onboard` video sources usually produce image frames at 30 FPS.  If the TensorRT engine inference code runs faster than that (which happens easily on a x86_64 PC with a good GPU), one particular image could be inferenced multiple times before the next image frame becomes available.  This causes problem in the object detector demos, since the original image could have been altered (bounding boxes drawn) and the altered image is taken for inference again.  To cope with this problem, use the optional `--copy_frame` flag to force copying/cloning image frames internally.

<a name="mtcnn"></a>
Demo #2: MTCNN
--------------

This demo builds upon the previous one.  It converts 3 sets of prototxt and caffemodel files into 3 TensorRT engines, namely the PNet, RNet and ONet.  Then it combines the 3 engine files to implement MTCNN, a very good face detector.

Assuming this repository has been cloned at "${HOME}/project/TRT-JetsonNX", follow these steps:

1. Build the TensorRT engines from the pre-trained MTCNN model.  (Refer to [mtcnn/README.md](https://github.com/T-DevH/TRT-JetsonNX/blob/master/mtcnn/README.md) for more information about the prototxt and caffemodel files.)

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/mtcnn
   $ make
   $ ./create_engines
   ```

2. Build the Cython code if it has not been done yet.  Refer to step 3 in Demo #1.

3. Run the "trt_mtcnn.py" demo program.  For example, I grabbed from the internet a poster of The Avengers for testing.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
   $ python3 trt_mtcnn.py --image ${HOME}/Pictures/avengers.jpg
   ```

   Here's the result:

   ![My face detect](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/MTCNN.png)

4. The "trt_mtcnn.py" demo program could also take various image inputs.  Refer to step 5 in Demo #1 for details.


<a name="ssd"></a>
Demo #3: SSD
------------

This demo shows how to convert pre-trained tensorflow Single-Shot Multibox Detector (SSD) models through UFF to TensorRT engines, and to do real-time object detection with the TensorRT engines.

Assuming this repository has been cloned at "${HOME}/project/TRT-JetsonNX", follow these steps:

1. Install requirements (pycuda, etc.) and build TensorRT engines from the pre-trained SSD models.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/ssd
   $ ./install.sh
   $ ./build_engines.sh
   ```

2. Run the "trt_ssd.py" demo program.  The demo supports 4 models: "ssd_mobilenet_v1_coco", "ssd_mobilenet_v1_egohands", "ssd_mobilenet_v2_coco", or "ssd_mobilenet_v2_egohands".  

   ```shell
   $ cd ${HOME}/project/TRT-JetsonNX
   $ python3 trt_ssd_async.py --usb 0 --width 1280 --height 720 --model ssd_mobilenet_v1_coco
   ```

   Here's the result of the "async" version of SSD:

   ![T-DevH detected](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/SSD.png)


3. The "trt_ssd.py" demo program could also take various image inputs.  Refer to step 5 in Demo #1 again.

4. To verify accuracy (mAP) of the optimized TensorRT engines and make sure they do not degrade too much (due to reduced floating-point precision of "FP16") from the original TensorFlow frozen inference graphs, you could prepare validation data and run "eval_ssd.py".  Refer to [README_mAP.md](README_mAP.md) for details.

   I compared mAP of the TensorRT engine and the original tensorflow model for both "ssd_mobilenet_v1_coco" and "ssd_mobilenet_v2_coco" using COCO "val2017" data.  The results were good.  In both cases, mAP of the optimized TensorRT engine matched the original tensorflow model.  The FPS (frames per second) numbers in the table were measured using "trt_ssd_async.py" on my Jetson NX DevKit with JetPack-4.4.

   | TensorRT engine         | mAP @<br>IoU=0.5:0.95 |  mAP @<br>IoU=0.5  | FPS on NX |
   |:------------------------|:---------------------:|:------------------:|:-----------:|
   | mobilenet_v1 TF         |          0.232        |        0.351       |      --     |
   | mobilenet_v1 TRT (FP16) |          0.232        |        0.351       |     70.8    |
   | mobilenet_v2 TF         |          0.248        |        0.375       |      --     |
   | mobilenet_v2 TRT (FP16) |          0.248        |        0.375       |     60.8    |


<a name="YOLOv3"></a>
Demo #4: YOLOv3
---------------

(Merged with Demo #5: YOLOv4...)

<a name="YOLOv4"></a>
Demo #5: YOLOv4
---------------

These 2 demos showcase how to convert pre-trained yolov3 and yolov4 models through ONNX to TensorRT engines. Part of the implementation consists of a "yolo_layer" plugin to speed up inference time of the 2 (yolov3/yolov4) models.

The "yolo_layer" plugin implementation is based on TensorRT's [IPluginV2IOExt](https://docs.nvidia.com/deeplearning/tensorrt/api/c_api/classnvinfer1_1_1_i_plugin_v2_i_o_ext.html).  

Similar plugin code by [wang-xinyu](https://github.com/wang-xinyu/tensorrtx/tree/master/yolov4) and [dongfangduoshou123](https://github.com/dongfangduoshou123/YoloV3-TensorRT/blob/master/seralizeEngineFromPythonAPI.py).  So big thanks to both of them.

Assuming this repository has been cloned at "${HOME}/project/TRT-JetsonNX", follow these steps:

1. Install "pycuda" in case you haven't done so in Demo #3.  Note that the installation script resides in the "ssd" folder.

   ```shell
   $ cd ${HOME}/project/TRT-JetsonNX/ssd
   $ ./install_pycuda.sh
   ```

2. Install **version "1.4.1" (not the latest version)** of python3 **"onnx"** module.  Note that the "onnx" module would depend on "protobuf" as stated in the [Prerequisite](#prerequisite) section.  Reference: [information provided by NVIDIA](https://devtalk.nvidia.com/default/topic/1052153/jetson-nano/tensorrt-backend-for-onnx-on-jetson-nano/post/5347666/#5347666).

   ```shell
   $ sudo pip3 install onnx==1.4.1
   ```

3. Go to the "plugins/" subdirectory and build the "yolo_layer" plugin.  When done, a "libyolo_layer.so" would be generated.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/plugins
   $ make
   ```

4. Download the pre-trained yolov3/yolov4 COCO models and convert the targeted model to ONNX and then to TensorRT engine.  I use "yolov4-416" as example below.  (Supported models: "yolov3-tiny-288", "yolov3-tiny-416", "yolov3-288", "yolov3-416", "yolov3-608", "yolov3-spp-288", "yolov3-spp-416", "yolov3-spp-608", "yolov4-tiny-288", "yolov4-tiny-416", "yolov4-288", "yolov4-416", "yolov4-608".

   ```shell
   $ cd ${HOME}/project/TRT-JetsonNX/yolo
   $ ./download_yolo.sh
   $ python3 yolo_to_onnx.py -m yolov4-416
   $ python3 onnx_to_tensorrt.py -m yolov4-416
   ```

   The next step ("onnx_to_tensorrt.py"). When done, the optimized TensorRT engine would be saved as "yolov4-416.trt".

5. Test the TensorRT "yolov4-416" engine with the "usb camera" image.

   ```shell
   $ cd ${HOME}/project/TRT-JetsonNX
   $ python3 trt_yolo.py --usb 0 -m yolov4-416
   ```

   This is a screenshot of the demo.

   !["yolov4-416" detection](https://github.com/T-DevH/TRT-JetsonNX/blob/master/doc/yolo4-1.png)

6. The "trt_yolo.py" demo program could also take various image inputs.  Refer to step 5 in Demo #1 again.

 
7. Other models than "yolov4-416"can be tested.

8. (Optional) I didn't test this, streaming TensorRT YOLO detection output over a network and view the results on a remote host, 
    
    ```shell
    $ python3 trt_yolo_mjpeg.py --usb 0 --copy_frame -m yolov4-416
    ```
    Then check out the output in a web browser (http://<ip addr>:8080)

9. Similar to step 5 of Demo #3, "eval_yolo.py" for evaluating mAP of the TensorRT yolov3/yolov4 engines.  Refer to [README_mAP.md](README_mAP.md) for details.

   ```shell
   $ python3 eval_yolo.py -m yolov4-416
   ```
   
   ```shell
   Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.459
   Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.700
   Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.496
   Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.256
   Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.517
   Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.627
   Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.342
   Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.531
   Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.554
   Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.331
   Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.617
   Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.744
   None
   ```

   Performance of different TensorRT yolov3/yolov4 engines with COCO "val2017" data  including FPS (frames per second) on the Jetson NX DevKit with JetPack-4.4 (TensorRT 7).

   | TensorRT engine        | mAP @<br>IoU=0.5:0.95 |  mAP @<br>IoU=0.5  | FPS on NX |
   |:-----------------------|:---------------------:|:------------------:|:-----------:|
   | yolov3-tiny-288 (FP16) |        0.077          |      0.158         |             |
   | yolov3-tiny-416 (FP16) |        0.096          |      0.202         |             |
   | yolov3-288 (FP16)      |        0.330          |      0.601         |             |
   | yolov3-416 (FP16)      |        0.373          |      0.664         |             |
   | yolov3-608 (FP16)      |        0.376          |      0.665         |             |
   | yolov3-spp-288 (FP16)  |        0.339          |      0.594         |             |
   | yolov3-spp-416 (FP16)  |        0.391          |      0.664         |             |
   | yolov3-spp-608 (FP16)  |        0.410          |      0.685         |             |
   | yolov4-tiny-288 (FP16) |        0.179          |      0.344         |             |
   | yolov4-tiny-416 (FP16) |        0.196          |      0.387         |             |
   | yolov4-288 (FP16)      |        0.376          |      0.591         |             |
   | yolov4-416 (FP16)      |        0.459          |      0.700         |             |
   | yolov4-608 (FP16)      |        0.488          |      0.736         |             |



<a name="int8_and_dla"></a>
Demo #6: Using INT8 and DLA core
--------------------------------

NVIDIA introduced [INT8 TensorRT inferencing](https://on-demand.gputechconf.com/gtc/2017/presentation/s7310-8-bit-inference-with-tensorrt.pdf) since CUDA compute 6.1+.  For the embedded Jetson product line, INT8 is available on Jetson AGX Xavier and Xavier NX.  In addition, NVIDIA further introduced [Deep Learning Accelerator (NVDLA)](http://nvdla.org/) on Jetson Xavier NX.  I tested both features on my Jetson Xavier NX DevKit, and shared the source code in this repo.

Please make sure you have gone through the steps of [Demo #5](#yolov4) and are able to run TensorRT yolov3/yolov4 engines successfully, before following along:

1. In order to use INT8 TensorRT, you'll first have to prepare some images for "calibration".  These images for calibration should cover all distributions of possible image inputs at inference time.  According to [official documentation](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#optimizing_int8_c), 500 of such images are suggested by NVIDIA.  As an example, I used 1,000 images from the COCO "val2017" dataset for that purpose.  Note that I've previously downloaded the "val2017" images for [mAP evaluation](README_mAP.md).

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/yolo
   $ mkdir calib_images
   ### randomly pick and copy over 1,000 images from "val207"
   find ${HOME}/data/coco/images/val2017/ -type f -name "*.jpg" -print0 | xargs -0 shuf -e -n 1000 -z | xargs -0 cp -vt ${HOME}/project/TRT-JetsonNX/yolo/calib_images
   ```

   When this is done, the 1,000 images for calibration should be present in the "${HOME}/project/tensorrt_demos/yolo/calib_images/" directory.

2. Build the INT8 TensorRT engine.  I use the "yolov3-608" model in the example commands below.  (I've also created a "build_int8_engines.sh" script to facilitate building multiple INT8 engines at once.)  Note that building the INT8 TensorRT engine on Jetson Xavier NX takes quite long.  By enabling verbose logging ("-v"), you would be able to monitor the progress more closely.

   ```
   $ ln -s yolov3-608.cfg yolov3-int8-608.cfg
   $ ln -s yolov3-608.onnx yolov3-int8-608.onnx
   $ python3 onnx_to_tensorrt.py -v --int8 -m yolov3-int8-608
   ```

3. (Optional) Build the TensorRT engines for the DLA cores.  I use the "yolov3-608" model as example again.  (I've also created a "build_dla_engines.sh" script for building multiple DLA engines at once.)

   ```
   $ ln -s yolov3-608.cfg yolov3-dla0-608.cfg
   $ ln -s yolov3-608.onnx yolov3-dla0-608.onnx
   $ python3 onnx_to_tensorrt.py -v --int8 --dla_core 0 -m yolov3-dla0-608
   $ ln -s yolov3-608.cfg yolov3-dla1-608.cfg
   $ ln -s yolov3-608.onnx yolov3-dla1-608.onnx
   $ python3 onnx_to_tensorrt.py -v --int8 --dla_core 1 -m yolov3-dla1-608
   ```

4. Test the INT8 TensorRT engine with the "dog.jpg" image.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
    python3 trt_yolo.py --usb 0 -m yolov3-int8-608
   ```

   (Optional) Also test the DLA0 and DLA1 TensorRT engines.

   ```shell
   $ python3 trt_yolo.py --usb 0 -m yolov3-dla0-608
   $ python3 trt_yolo.py --usb 0 -m yolov3-dla1-608
   ```

5. Evaluate mAP of the INT8 and DLA TensorRT engines.

   ```shell
   $ python3 eval_yolo.py -m yolov3-int8-608
   $ python3 eval_yolo.py -m yolov3-dla0-608
   $ python3 eval_yolo.py -m yolov3-dla1-608
   ```

6. I tested the 5 original yolov3/yolov4 models on my Jetson Xavier NX DevKit with JetPack-4.4 (TensorRT 7.1.3.4).  Here are the results.

   The following **FPS numbers** were measured under "15W 6CORE" mode, with CPU/GPU clocks set to maximum value (`sudo jetson_clocks`).

   | TensorRT engine  |   FP16   |   INT8   |   DLA0   |   DLA1   |
   |:-----------------|:--------:|:--------:|:--------:|:--------:|
   | yolov3-tiny-416  |    58    |    65    |    42    |    42    |
   | yolov3-608       |   15.2   |   23.1   |   14.9   |   14.9   |
   | yolov3-spp-608   |   15.0   |   22.7   |   14.7   |   14.7   |
   | yolov4-tiny-416  |    57    |    60    |     X    |     X    |
   | yolov4-608       |   13.8   |   20.5   |   8.97   |   8.97   |

   And the following are **"mAP@IoU=0.5:0.95" / "mAP@IoU=0.5"** of those TensorRT engines.

   | TensorRT engine  |       FP16      |       INT8      |       DLA0      |       DLA1      |
   |:-----------------|:---------------:|:---------------:|:---------------:|:---------------:|
   | yolov3-tiny-416  |  0.096 / 0.202  |  0.094 / 0.198  |  0.096 / 0.199  |  0.096 / 0.199  |
   | yolov3-608       |  0.376 / 0.665  |  0.378 / 0.670  |  0.378 / 0.670  |  0.378 / 0.670  |
   | yolov3-spp-608   |  0.410 / 0.685  |  0.407 / 0.681  |  0.404 / 0.676  |  0.404 / 0.676  |
   | yolov4-tiny-416  |  0.196 / 0.387  |  0.190 / 0.376  |        X        |        X        |
   | yolov4-608       |  0.488 / 0.736  | *0.317 / 0.507* |  0.474 / 0.727  |  0.473 / 0.726  |

7. Refferences:
Big thanks to xxxxxxxxx who provided most of the scripts in this repo
  

Licenses
--------

I referenced source code of [NVIDIA/TensorRT](https://github.com/NVIDIA/TensorRT) samples to develop most of the demos in this repository.  Those NVIDIA samples are under [Apache License 2.0](https://github.com/NVIDIA/TensorRT/blob/master/LICENSE).
