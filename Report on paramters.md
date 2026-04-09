# Report on parameters of VGG16 and depthwise separable convolution

## Parameters Formulas
* **Parameters_conv2d** = (kernel_height x kernel_width x input_channels) × output_channels + output_channels (biases) 
* **Parameters_depth_conv2d** = (kernel_height x kernel_width) x input_channels + input_channels (biases) 
* **Parameters_dense** = input_units × output_units + output_units (biases)
* **Parameters_batchnorm** = 4 x channels
* **Parameters** = 0 (for pooling layer as it is not considered)

---

## 1. VGG16 (Frozen + Trainable)

| Layer | Formula | Calculation | Params |
| :--- | :--- | :--- | :--- |
| Block1 Conv1 | $3\times3\times3\times64+64$ | $1,728+64$ | [cite_start]1,792 |
| Block1 Conv2 | $3\times3\times64\times64+64$ | $36,864+64$ | [cite_start]36,928 |
| Block2 Conv1 | $3\times3\times64\times128+128$ | $73,728+128$ | [cite_start]73,856 |
| Block2 Conv2 | $3\times3\times128\times128+128$ | $147,456+128$ | [cite_start]147,584 |
| Block3 Conv1 | $3\times3\times128\times256+256$ | $294,912+256$ | [cite_start]295,168  |
| Block3 Conv2 | $3\times3\times256\times256+256$ | $589,824+256$ | [cite_start]590,080  |
| Block3 Conv3 | $3\times3\times256\times256+256$ | $589,824+256$ | [cite_start]590,080  |
| Block4 Conv1 | $3\times3\times256\times512+512$ | $1,179,648+512$ | [cite_start]1,180,160  |
| Block4 Conv2 | $3\times3\times512\times512+512$ | $2,359,296+512$ | [cite_start]2,359,808  |
| Block4 Conv3 | $3\times3\times512\times512+512$ | $2,359,296+512$ | [cite_start]2,359,808  |
| Block5 Conv1 | $3\times3\times512\times512+512$ | $2,359,296+512$ | [cite_start]2,359,808  |
| Block5 Conv2 | $3\times3\times512\times512+512$ | $2,359,296+512$ | [cite_start]2,359,808  |
| Block5 Conv3 | $3\times3\times512\times512+512$ | $2,359,296+512$ | [cite_start]2,359,808  |
| **VGG16 Total** | | | **14,714,688** 
### Custom Trainable Layers


| Layer | Formula | Calculation | Params |
| :--- | :--- | :--- | :--- |
| GlobalAvgPool2D | - | - | 0  |
| Dense 1 | $512 \times 512 + 512$ | $262,144 + 512$ |262,656 |
| Dropout | - | - | 0  |
| Dense 2 | $512 \times 256 + 256$ | $131,072 + 256$ | 131,328  |
| Dropout | - | - | 0  |
| Dense 3 (output) | $256 \times 3 + 3$ | $768 + 3$ | 771  |
| **Top Layers Total** | | | **394,755**  |

### VGG16 Model Totals
* **Non-trainable (frozen VGG16 layers)** = 7,635,264 
* **Trainable (last 4 conv layers)** = 7,079,424 
* **Trainable (top Dense layers)** = 394,755 
* **TOTAL** = 7,635,264 + 7,079,424 + 394,755 = 15,109,443 
* **Optimizer params** = 2 × 7,474,179 = 14,948,358 
* **Total with optimizer** = 7,635,264 + 7,474,179 + 14,948,358 = 30,057,801 

**Final stats from code:**
* **Total params:** 30,057,803 (114.66 MB) 
* **Trainable params:** 7,474,179 (28.51 MB)
* **Non-trainable params:** 7,635,264 (29.13 MB)
* **Optimizer params:** 14,948,360 (57.02 MB) 

---

## 2. Depthwise separable convolution

| Layer | Input Shape | Kernel | Formula | Params |
| :--- | :--- | :--- | :--- | :--- |
| **Block 1** | | | | |
| DepthwiseConv2D | 224x224x3 | 3 x 3 | $(3\times3\times3)$ | 27 |
| Conv2D (Pointwise) | 224x224x3 | 1 x 1, 64 filters | $(1\times1\times3) \times 64 + 64$ | 192 + 64 = 256 |
| BatchNormalization | 224x224x64 | - | 4 x 64 | 256  |
| MaxPooling2D | 224x224x64 | 2 x 2 | - | 0  |
| **Block 2** | | | |  |
| DepthwiseConv2D | 112x112x64 | 3 x 3 | $(3\times3\times64)$ | 576 |
| Conv2D (Pointwise) | 112x112x64 | 1 x 1, 128 filters | $(1\times1\times64) \times 128 + 128$ | 8,192 + 128 = 8,320 |
| BatchNormalization | 112x112x128 | - | 4 x 128 | 512 |
| MaxPooling2D | 112x112x128 | 2 x 2 | - | 0 |
| **Block 3** | | | | |
| DepthwiseConv2D | 56x56x128 | 3 x 3 | $(3\times3\times128)$ | 1,152  |
| Conv2D (Pointwise) | 56x56x128 | 1 x 1, 256 filters | $(1\times1\times128) \times 256 + 256$ | 32,768 + 256 = 33,024  |
| BatchNormalization | 56x56x256 | - | 4 x 256 | 1,024  |
| MaxPooling2D | 56x56x256 | 2 x 2 | - | 0  |
| **Block 4** | | | | |
| DepthwiseConv2D | 28x28x256 | 3 x 3 | $(3\times3\times256)$ | 2,304 |
| Conv2D (Pointwise) | 28x28x256 | 1 x 1, 512 filters | $(1\times1\times256) \times 512 + 512$ | 131,072 + 512 = 131,584  |
| BatchNormalization | 28x28x512 | - | 4 x 512 | 2,048  |
| **Top Layers** | | | | |
| GlobalAveragePooling2D | 28x28x512 | - | - | 0  |
| Dense | 512 -> 256 | 256 units | $(512\times256) + 256$ | 131,072 + 256 = 131,328 |
| Dropout (0.5) | 256 | - | - | 0  |
| Dense | 256 -> 3 | 3 units | $(256\times3) + 3$ | 768 + 3 = 771  |

### Depthwise Model Summary
* **Optimizer parameters** = 2 x Trainable parameters = $2\times311,713$ = 623,426
* **Total Trainable** = 311,262 
* **Total Non-trainable** = 1,920 
* **Model Total** = 313,182 

**Final Stats From the code:**
* **Total params:** 937,061 (3.57 MB)
* **Trainable params:** 311,713 (1.19 MB) 
* **Non-trainable params:** 1,920 (7.58 KB) 
* **Optimizer params:** 623,428 (2.38 MB)
