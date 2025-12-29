# Example for yaml

## Example for configuration YAML file for PTQ quantization using hb_mappe

for this cmd:  

```bash
hb_mapper makertbin --config config.yaml --model-type onnx
```

For details, please refer to the official documentation(X3 Toolchain> 3.2.3.1 YAML config file).

```yaml
# config.yaml
model_parameters:
    # file name to the ONNX model
    onnx_model: 'your_model.onnx'
    # BPU architecture type, e.g., X3 BPU: 'bernoulli2', X5 BPU: 'bayes2', s100 (why you do not use AGX??)...
    march: "bernoulli2"
    # Specify whether to output intermediate results of each layer during model conversion. If True, output all intermediate results of each layer.
    layer_out_dump: False
    # Log file output control parameters,
    # debug outputs detailed information of model conversion
    # info outputs only key information
    # warn outputs warnings and errors and above
    log_level: 'debug'
    # Directory to store the results of model conversion
    working_dir: 'model_output'
    # Prefix for output bin files
    output_model_file_prefix: 'x3_yolo11n_cls'

# model input related parameters, if multiple input nodes, separate with ';', use None for default settings
input_parameters:
    # (optional) The name of the model input node, this name should be consistent with the name in the model file, otherwise an error will be reported. If not filled, the node name in the model file will be used
    # input_name: "if it is optional, why fill here, right? right?"
    input_name: ""
    # The actual data format input to the network during execution, including nv12/rgb/bgr/yuv444/gray/featuremap,
    # If the input data is yuv444 and the model training uses bgr(NCHW), hb_mapper will automatically insert a YUV to BGR(NCHW) conversion operation
    input_type_rt: 'nv12'
    # The input data layout that the mixed heterogeneous model needs to adapt to after conversion, can be set to: NHWC/NCHW
    # If input_type_rt is configured as nv12, this parameter does not need to be configured
    input_layout_rt: 'NHWC'
    # The data format of the network during training, optional values are rgb/bgr/gray/featuremap/yuv444
    input_type_train: 'rgb'
    # The data layout of the network during training, optional values are NHWC/NCHW
    input_layout_train: 'NCHW'
    # The input size of the model network, separated by 'x'. If not filled, the input size in the model file will be used, otherwise it will override the input size in the model file
    # input_shape: '1x3x416x416'
    input_shape: ''
    # The preprocessing method for network input, mainly the following:
    # no_preprocess no operation
    # data_mean subtract channel mean_value
    # data_scale multiply image pixels by data_scale coefficient
    # data_mean_and_scale subtract channel mean and then multiply by scale coefficient
    norm_type: 'data_mean_and_scale'
    # The mean value to subtract from the image, if it is a channel mean, values must be separated by spaces
    mean_value: 123.68 116.28 103.53
    # The scaling factor for image preprocessing, if it is a channel scaling factor, values must be separated by spaces
    scale_value: 0.0171 0.0175 0.0174

calibration_parameters:
    # The directory where reference images for model quantization are stored. Supported image formats include JPEG, BMP, etc. The input images
    # should represent typical scenarios, usually selecting 20~100 images from the test set. Additionally, the input
    # images should cover typical scenarios and avoid rare cases such as overexposure, saturation, blur, pure black, pure white, etc.
    # If there are multiple input nodes, they should be separated by ';'
    cal_data_dir: './calibration_images'
    # If the size of the input image files is inconsistent with the size used during model training, and preprocess_on is true,
    # default preprocessing method (skimage resize) will be used,
    # to scale or crop the input images to the specified size, otherwise, the user needs to preprocess the images to the training size in advance
    preprocess_on: False
    # The algorithm type for model quantization, supports kl, max, default, load. Usually, default is sufficient. If the model is exported from QAT, load should be selected.
    calibration_type: 'default'

    # This parameter is for the 'max' calibration method and is used to adjust the cutoff point for 'max' calibration. This parameter is only effective when calibration_type is 'max'.
    # The value range for this parameter is 0.5 ~ 1.0. Common configuration options include: 0.99999/0.99995/0.99990/0.99950/0.99900.
    max_percentile: 0.9999

# Compiler related parameters
compiler_parameters:
    # c compilation strategy, supports bandwidth and latency optimization modes ;
    # bandwidth aims to optimize DDR access bandwidth;
    # latency aims to optimize inference time
    compile_mode: 'latency'
    # Setting debug to True will enable the compiler's debug mode, which can output performance simulation related information such as frame rate, DDR bandwidth usage, etc.
    debug: False
    # Specify the number of cores for model compilation. If not specified, a single-core model is compiled by default. To compile a dual-core model, uncomment the following line.
    core_num: 2
    # Optimization level range is O0~O3
    # O0 does no optimization, fastest compilation speed, lowest optimization level,
    # O1-O3 as the optimization level increases, the expected execution speed of the compiled model will be faster, but the required compilation time will also be longer.
    # O2 is recommended for the fastest verification
    optimize_level: 'O3'
```

## Exmple for yolo11n_cls config yaml

yolo11n_cls_config.yaml

```yaml
model_parameters:
  onnx_model: 'yolo11n.onnx'
  march: 'bernoulli2'
  working_dir: 'model_output'
  output_model_file_prefix: 'yolo11n_detect_bernoulli2_nv12'
  node_info: {"/model.10/m/m.0/attn/Softmax": {'ON': 'BPU','InputType': 'int16','OutputType': 'int16'}}

input_parameters:
  input_type_rt: 'nv12'
  input_type_train: 'rgb'
  input_layout_train: 'NCHW'
  norm_type: 'data_scale'
  # this scale value is 1/255
  scale_value: 0.003921568627451

calibration_parameters:
  cal_data_dir: './calibration_images'
  cal_data_type: 'float32'
  calibration_type: 'default'

compiler_parameters:
  compile_mode: 'latency'
  optimize_level: 'O3'
```

## Reference

- [X3 Toolchain User Guide(Chinese)](http://10.33.20.238:8080/oe_mapper/source/faststart/quickstart.html#id2)
- [X5: Yolo11n Deloyment(Chinese)](https://forum.d-robotics.cc/t/topic/33301)