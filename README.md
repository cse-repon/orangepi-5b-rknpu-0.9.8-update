# Updating RKNPU Driver to Version 0.9.8 on Orange Pi 5B (Jammy 1.0.8 with Linux 6.1.43-rockchip-rk3588)


This guide provides detailed instructions to update the RKNPU driver to version 0.9.8 on an Orange Pi 5B running Jammy 1.0.8 with the Linux 6.1.43-rockchip-rk3588 kernel. Upgrading the RKNPU driver is essential for successfully running RKLLM multimodal models.


**You can download the prebuilt [linux-image-current-rockchip-rk3588_1.0.8_arm64.deb](https://github.com/cse-repon/orangepi-5b-rknpu-0.9.8-update/blob/830e587c5bc9ae39f2655caad8d7e837f42d3935/linux-image-current-rockchip-rk3588_1.0.8_arm64.deb) and jump to step 9 directly** or continue building the kernel on your board.


You can download the working rkllm models from below HF links for running the **[examples of RKNN LLM repository(https://github.com/airockchip/rknn-llm/tree/main/examples)]**:
1. https://huggingface.co/3ib0n/Qwen2-VL-2B-rkllm
2. https://huggingface.co/VRxiaojie/DeepSeek-R1-Distill-Qwen-7B-RK3588S-RKLLM1.1.4




# Build Linux 6.1.43 rk3588 Kernel with RKNPU Driver 0.9.8
## Prerequisites

- **Operating System**: [Ubuntu 22.04 Jammy 6.1.43 (https://drive.google.com/drive/folders/1xhP1KeW_hL5Ka4nDuwBa8N40U8BN0AC9)] 
- **Kernel Version**: Linux 6.1.43-rockchip-rk3588
- **Disk Space**: At least 20 GB of free space
- **Network Access**: Ensure the development board has internet access to clone repositories and download necessary packages.


## Step 1: Verify Current NPU Driver Version

Before proceeding, check the current version of the RKNPU driver:

```bash
sudo cat /sys/kernel/debug/rknpu/version
```

If the output indicates a version lower than 0.9.8, proceed with the following steps to upgrade the driver.

## Step 2: Install Required Dependencies

Ensure that your system has the necessary packages installed:

```bash
sudo apt-get update
sudo apt-get install -y git build-essential libssl-dev
```

## Step 3: Clone the Orange Pi Build Repository

The Orange Pi Build repository is based on the Armbian build framework and is used to compile the Linux kernel for Orange Pi boards. Clone the repository:

```bash
cd ~
git clone https://github.com/orangepi-xunlong/orangepi-build.git -b next
```


## Step 4: Download the Linux 6.1 Kernel Source

Create a directory to store the kernel source and navigate into it:

```bash
cd orangepi-build
mkdir kernel && cd kernel
```

Clone the kernel source code:

```bash
git clone https://github.com/orangepi-xunlong/linux-orangepi.git -b orange-pi-6.1-rk35xx
```

Rename the directory for consistency:

```bash
mv linux-orangepi/ orange-pi-6.1-rk35xx
```

## Step 5: Download and Extract the RKNPU Driver

Obtain the RKNPU driver version 0.9.8 from the official repository:

```bash
cd ~/orangepi-build
git clone https://github.com/airockchip/rknn-llm.git
```

Unzip the driver:

```bash
tar -xvf /rknn-llm/rknpu-driver/rknpu_driver_0.9.8_20241009.tar.bz2
```

Copy the extracted driver files to the kernel source:

```bash
cp -r drivers/ kernel/orange-pi-6.1-rk35xx/
```

## Step 6: Modify Kernel Source Files

To ensure compatibility and avoid compilation errors, make the following modifications:

1. **Modify kernel/include/linux/mm.h file**:

   Edit the file:

   ```bash
   sudo nano kernel/orange-pi-6.1-rk35xx/include/linux/mm.h
   ```

   Add the following code at an appropriate location:

   ```c
   static inline void vm_flags_set(struct vm_area_struct *vma, vm_flags_t flags)
   {
       vma->vm_flags |= flags;
   }
   static inline void vm_flags_clear(struct vm_area_struct *vma, vm_flags_t flags)
   {
       vma->vm_flags &= ~flags;
   }
   ```

![image](https://github.com/user-attachments/assets/de47418d-aa0c-4494-a5fe-86162f5aa5d5)

1. **Modify rknpu\_devfreq.c file:**

   Edit the file:

   ```bash
   sudo nano kernel/orange-pi-6.1-rk35xx/drivers/rknpu/rknpu_devfreq.c
   ```

   Comment out this sentence on line 242 .set\_soc\_info = rockchip\_opp\_set\_low\_length,

   ```c
   //.set_soc_info = rockchip_opp_set_low_length,
   ```
![image](https://github.com/user-attachments/assets/26e01e59-d2b1-4f29-b997-f171b998ec8f)

## Step 7: Disable Source Synchronization

Because we manually overwrote drivers to the kernel/orange-pi-6.10-rk35xx directory before, if we run the compilation directly now, the script will check the inconsistency with the cloud source code, resulting in the problem of re-pulling the code to overwrite. Therefore, the source code synchronization function should be disabled in the configuration file.



First, run the build.sh script once to initialize.:

1. Run the build script to initialize:
   ```bash
   sudo ./build.sh
   ```
   Wait for a moment, and when you see the interface that asks us to make a selection, use the → arrow key and the Enter key on the keyboard to exit the menu.

   Check the current directory again and find an additional userpatches folder, which contains configuration files.&#x20;
2. GNORE\_UPDATES="yes"

   Save and vi.the

   &#x20;configuration file:
   ```bash
   sudo nano userpatches/config-default.conf
   ```
  Find and set IGNORE_UPDATES to yes:
   ```bash
   IGNORE_UPDATES="yes"
   ```

## Step 8: Compile the Linux Kernel

Start the build process:

```bash
sudo ./build.sh
```

Select the appropriate options for your board and kernel version when prompted.

![image](https://github.com/user-attachments/assets/c7730fc3-59ce-404b-ad18-e95c2e2812e7)

![image](https://github.com/user-attachments/assets/87a5024e-0561-41df-9f16-231dda9f56db)

![image](https://github.com/user-attachments/assets/fb142587-3888-4964-bdd3-e5a3b051e725)

![image](https://github.com/user-attachments/assets/f078d22f-886a-40b0-9bc6-8b8773c8ac00)

After successful build, you will see like below messages (Note: the first time build might require nearly 40 minutes!)
![image](https://github.com/user-attachments/assets/addcd887-8dda-4b7c-a2fb-f4ddd6c011f5)

Check the file 
![image](https://github.com/user-attachments/assets/f32ef375-33cd-4c30-8af0-e86cdefe0e13)

## Step 9: Install the New Kernel

Need to install only the **linux-image-current-rockchip-rk3588_1.0.8_arm64.deb** package:

```bash
sudo apt purge -y linux-image-current-rockchip-rk3588
sudo dpkg -i output/debs/linux-image-current-rockchip-rk3588_1.0.8_arm64.deb
```

## Step 10: Verify the Updated NPU Driver Version

Reboot the system:

```bash
sudo reboot
```

After rebooting, verify the NPU driver version:

```bash
sudo cat /sys/kernel/debug/rknpu/version
```
The output should now indicate version 0.9.8.
![image](https://github.com/user-attachments/assets/80a35e0a-8389-4800-bf2d-c27547155f0c)

## References

- [RKLLM Deployment Guide](https://wiki.vrxiaojie.top/Deepseek-R1-RK3588-OrangePi5/RKLLM部署语言大模型教程/（可选）升级RKNPU驱动.html)
- [Orange Pi Bu](https://github.com/airockchip/rknn-llm)[ild ](https://github.com/orangepi-xunlong/orangepi-build)[Repository](https://wiki.vrxiaojie.top/Deepseek-R1-RK3588-OrangePi5/RKLLM部署语言大模型教程/（可选）升级RKNPU驱动.html)
- [RKN LLM](https://github.com/airockchip/rknn-llm)
- [Easy installation of RKNN and RKLLM ](https://github.com/Pelochus/ezrknpu) Please use the latest RKLLM 1.1.4 version from RKNN LLM git repository

By following these steps, you should have successfully updated the RKNPU driver to version 0.9.8, enabling the deployment of RKLLM multimodal models on your Orange Pi 5B.

