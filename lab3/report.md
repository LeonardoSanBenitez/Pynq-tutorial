
# Question 1
**Explain what is the purpose and the contents of the following files at the first (BOOT) partition of the SD card: arch.json, BOOT.BIN, boot.scr, dpu.xclbin and image.ub**
arch.json: identifies the hardware architecture of the DPU on the Ultra96-V2. It helps the Vitis-AI runtime understand how to talk to the DPU for running AI models efficiently. Think of it as identifying a "cheat sheet" for the system to know what hardware features are available.
The content of `/media/sd-mmcblk0p1/arch.json` is just `{"fingerprint":"0x1000020F6014405"}`, which is the same fingerprint obtained at `xdputil query`

BOOT.BIN: it's the first file loaded by the board and contains the FPGA bitstream (hardware configuration) and the ARM processor's bootloader (like U-Boot). It's like the "power-on" combo that wakes up both the FPGA and CPU, getting them ready to work together.

boot.scr: U-Boot script that sets up environment variables and boot commands before Linux starts. It's like a quick to-do list the board checks to know how to load the OS (e.g., where to find the kernel, what parameters to use, etc).

dpu.xclbin: compiled FPGA design for the DPU, which configures the programmable logic to run NNs. This is the "brain" file that turns the generic FPGA into a custom AI chip when the system boots.

image.ub: bundled file containing the Linux kernel (the OS core), device tree (hardware mapping of what is connected where), and initramfs (temporary root filesystem). It's like a "starter pack" for Linux—everything needed to boot before the full rootfs takes over

Additional explanations can be found at [hackster]([url](https://www.hackster.io/AlbertaBeef/vitis-ai-1-4-flow-for-avnet-vitis-platforms-5ebc3f))

# Question 2
**Explain what is the purpose and the contents of the second partition of the SD card**

It’s the "operating system" part – Contains all the Linux system files (like /bin, /usr, /etc) that make the board a working computer.

Pre-installed AI stuff – Comes with ready-to-use AI software (Vitis-AI runtime) and demo apps (like image recognition) so you can run AI models right away.

We can see that the content of `/media/sd-mmcblk0p2` is:
```bash
bin  boot  dev  etc  home  lib  log_lock.pid  lost+found  media  mnt  proc  run  sbin  sys  tmp  usr  var
```

just like a normal linux operating system

# Question 3

**Output of xdputil query**
```bash
{
    "DPU IP Spec":{
        "DPU Core Count":1,
        "DPU Target Version":"v1.4.1",
        "IP version":"v3.3.0",
        "generation timestamp":"2021-06-07 19-15-00",
        "git commit id":"df4d0c7",
        "git commit time":2106071910,
        "regmap":"1to1 version"
    },
    "VAI Version":{
        "libvart-runner.so":"Xilinx vart-runner Version: 1.4.0-fa49b842f283242091476cf8e1ae4d242a2a838e 585 2021-07-13-18:50:52 ",
        "libvitis_ai_library-dpu_task.so":"Xilinx vitis_ai_library dpu_task Version: 1.4.0-93d2e0097889ccebb5f1256e0afe8409de81f482 585 2021-07-13 10:54:30 [UTC] ",
        "libxir.so":"Xilinx xir Version: xir-ff89b11dcabb00eef6d148fcf660c8e6d02eb184 2021-07-13-18:49:08",
        "target_factory":"target-factory.1.4.0 ce1b39e329cc06cb7545e8aa39174fb8b9969f0b"
    },
    "kernels":[
        {
            "DPU Arch":"DPUCZDX8G_ISA0_B2304_MAX_BG2",
            "DPU Frequency (MHz)":200,
            "IP Type":"DPU",
            "Load Parallel":2,
            "Load augmentation":"enable",
            "Load minus mean":"disable",
            "Save Parallel":2,
            "XRT Frequency (MHz)":200,
            "cu_addr":"0xb0000000",
            "cu_handle":"0xaaaaeaf50c90",
            "cu_idx":0,
            "cu_mask":1,
            "cu_name":"DPUCZDX8G:DPUCZDX8G_1",
            "device_id":0,
            "fingerprint":"0x1000020f6014405",
            "name":"DPU Core 0"
        }
    ]
}
```

This output from xdputil query is basically a spec sheet and status report for our DPU. It tells us there’s one DPU core running at 200 MHz with a Vitis-AI 1.4.1-compatible design, and lists the software versions (like libvart-runner.so) to make sure everything matches up. The "kernels" section dives into hardware details—like the DPU’s architecture (DPUCZDX8G_ISA0_B2304_MAX_BG2), its memory address (0xb0000000), and parallelism settings (2x load/save ops at once). If your AI models act weird, check here first—mismatched versions or wrong arch settings could be the culprit!

After this, the device is ready for use, as can also be seen by the command `xbutil scan`:
```bash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
System Configuration
OS name:        Linux
Release:        5.10.0-xilinx-v2021.1
Version:        #1 SMP Fri Jun 4 15:57:16 UTC 2021
Machine:        aarch64
Glibc:          2.32
Distribution:   N/A
Now:            Mon Feb 28 06:58:46 2022 GMT
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
XRT Information
Version:        2.11.0
Git Hash:       0dc9f505a3a910dea96166db7b5df8530b9ae38e
Git Branch:     2021.1
Build Date:     2021-06-05 12:31:54
ZOCL:           2.11.0,0dc9f505a3a910dea96166db7b5df8530b9ae38e
 [0]:edge
```

# Question 4
**Run the examples**
We can run the examples provided by Xilinx, with the pretrained models and predownloaded data:
```bash
cd ~/Vitis-AI/demo/VART/adas_detection && ./adas_detection ./video/adas.avi /usr/share/vitis_ai_library/models/yolov3_adas_pruned_0_9/yolov3_adas_pruned_0_9.xmodel

cd ~/Vitis-AI/demo/VART/pose_detection && ./pose_detection ./video/pose.mp4 /usr/share/vitis_ai_library/models/sp_net/sp_net.xmodel /usr/share/vitis_ai_library/models/ssd_pedestrian_pruned_0_97/ssd_pedestrian_pruned_0_97.xmodel

cd ~/Vitis-AI/demo/VART/segmentation && ./segmentation ./video/traffic.mp4 /usr/share/vitis_ai_library/models/fpn/fpn.xmodel

# Specially for this one, copy all outputs (because we will compare with part 3)
cd ~/Vitis-AI/demo/VART/resnet50_mt_py && python3 ./resnet50.py 8 /usr/share/vitis_ai_library/models/resnet50/resnet50.xmodel

cd ~/Vitis-AI/demo/VART/inception_v1_mt_py && python3 ./inception_v1.py 8 /usr/share/vitis_ai_library/models/inception_v1/inception_v1.xmodel

cd ~/Vitis-AI/demo/Vitis-AI-Library/samples/facedetect &&  ./test_performance_facedetect densebox_640_360 ./test_performance_facedetect.list

cd ~/xilinx_developer/ppdemo1216 && ./demo ./ppd/vlist.txt ./ppd/ 1

cd ~/xilinx_developer/ppdemo1216 && ./demo ./ppd/vlist.txt ./ppd/ 2

cd ~/xilinx_developer/ppdemo1216 && ./demo ./ppd/vlist.txt ./ppd/ 4
```
And check if their output matches the output in the PDF.


# Question 5
**Compare the results obtained from both caffe and tensorflow with ones obtained at section 3.6e.1**

![Screenshot from 2025-04-10 06-54-56](https://github.com/user-attachments/assets/8bdf0318-cab8-4b87-8b1f-5dcf6d002928)

When running `cd ~/Vitis-AI/demo/VART/resnet50_mt_py && python3 ./resnet50.py 8 /usr/share/vitis_ai_library/models/resnet50/resnet50.xmodel`, we obtained the output:
```bash
FPS=27.65, total frames = 2880.00 , time=104.172245 seconds
```

## Compilation

The models were compiled manually using the **Vitis AI Compiler**.  
The original models (from **Caffe** and **TensorFlow**) were first downloaded from the **Xilinx AI Model Zoo** and extracted.  

For our Ultra96v2 board, the DPU configuration was set up using the corresponding `arch.json` file with the target architecture **DPUCZDX8G_ISA0_B2304_MAX_BG2**.  

The compilation process included:
- **Quantization** of the models (8-bit integer precision),
- **Operator fusion** to optimize model graph execution,
- **Generation** of `.xmodel` files for FPGA deployment.

The compiled models were:
- `resnet50` from **Caffe**
- `resnet_v1_50_tf` from **TensorFlow**

The compilation scripts used were `compile_cf_model.sh` for Caffe models and `compile_tf_model.sh` for TensorFlow models.  
The resulting `.xmodel` files were transferred to the Ultra96v2 board for testing.

---

## Results

When executing the compiled ResNet-50 model using VART with the command:

```bash
cd ~/Vitis-AI/demo/VART/resnet50_mt_py
python3 ./resnet50.py 8 /usr/share/vitis_ai_library/models/resnet50/resnet50.xmodel
```

**TODO**: Compare the results obtained from both caffe and tensorflow with ones obtained at section 3.6e.1

# Question 6

**Test the models on the board**

The model files from the last part were uploaded to git, as well as the test images. They are available in [Pynq-tutorial/lab3](https://github.com/LeonardoSanBenitez/Pynq-tutorial/tree/main/lab3) and [Pynq-tutorial/lab3/images](https://github.com/LeonardoSanBenitez/Pynq-tutorial/tree/main/lab3/images), respectively.

After cloning the repository, we modified the file `~/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py` to point to the correct dataset (since the dataset path was hardcoded in the script). In a nutshell, this script does a performance test with ResNet-50 on the on a Xilinx DPU, including loading the model, preprocessing the images, running the DPU inference, and measuring FPS. The modified version of the script is available in [Pynq-tutorial/lab3/resnet_50.py](https://github.com/LeonardoSanBenitez/Pynq-tutorial/blob/main/lab3/resnet_50.py).

For each of the 3 model, inference should be performed with a command like like the following:

```bash
python3 ~/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py 8 Pynq-tutorial/lab3/densebox_640_360/densebox_640_360.xmodel
```

However, we face the following error:

```python
Exception in thread Thread-1:
Traceback (most recent call last):
Exception in thread Thread-2:
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
Exception in thread Thread-3:
Traceback (most recent call last):
Exception in thread Thread-4:
Traceback (most recent call last):
Exception in thread Thread-5:
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
Exception in thread Thread-6:
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
Traceback (most recent call last):
Traceback (most recent call last):
    self.run()
    self.run()
  File "/usr/lib/python3.8/threading.py", line 870, in run
Traceback (most recent call last):
Exception in thread Thread-7:
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
  File "/usr/lib/python3.8/threading.py", line 870, in run
    self.run()
    self._target(*self._args, **self._kwargs)
    self.run()
Exception in thread Thread-8:
    self.run()
  File "/usr/lib/python3.8/threading.py", line 870, in run
  File "/usr/lib/python3.8/threading.py", line 870, in run
Traceback (most recent call last):
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
    self._target(*self._args, **self._kwargs)
Traceback (most recent call last):
  File "/usr/lib/python3.8/threading.py", line 870, in run
    self.run()
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
    self._target(*self._args, **self._kwargs)
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
  File "/usr/lib/python3.8/threading.py", line 932, in _bootstrap_inner
  File "/usr/lib/python3.8/threading.py", line 870, in run
    self._target(*self._args, **self._kwargs)
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
    self.run()
    self._target(*self._args, **self._kwargs)
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
    self.run()
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
  File "/usr/lib/python3.8/threading.py", line 870, in run
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
    self._target(*self._args, **self._kwargs)
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
  File "/usr/lib/python3.8/threading.py", line 870, in run
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
    self._target(*self._args, **self._kwargs)
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
    self._target(*self._args, **self._kwargs)
  File "/home/root/Vitis-AI/demo/VART/resnet50_mt_py/resnet50.py", line 133, in runResnet50
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
    imageRun[j, ...] = img[(count + j) % n_of_images].reshape(input_ndim[1:])
ValueError: cannot reshape array of size 150528 into shape (360,640,3)
FPS=59175.41, total frames = 2880.00 , time=0.048669 seconds
```

