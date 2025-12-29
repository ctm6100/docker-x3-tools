# X3 OE Toolchain Docker Compose Usage

This project provides a Docker Compose setup for running the OpenExplorer AI Toolchain for X3 on Ubuntu 20 (CPU version).

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed
- OpenExplorer (OE) package for X3 (download from [D-Robotics Forum](https://forum.d-robotics.cc/t/topic/28035))

> It seems that the docker image is the lastest on the d-robotics forum, so you may choose to download from there if docker hub version is outdated.

- Docker Imager for OpenExplorer AI Toolchain (CPU version) is available on [Docker Hub](https://hub.docker.com/r/openexplorer/ai_toolchain_ubuntu_20_xj3_cpu) or [D-Robotics Forum](https://forum.d-robotics.cc/t/topic/28035)

## Directory Structure

```
X3_Model/
├── docker-compose.yml
├── open_explorer/
│   └── horizon_xj3_open_explorer_{your_version}/
└── dataset/
```

- Place your OE package in the `open_explorer/` directory (e.g., `open_explorer/horizon_xj3_open_explorer_v2.6.6-py38_20240717`).
- Place your dataset in the `dataset/` directory.

## Docker Usage

### 1. Edit Compose File (if needed)

Update the `docker-compose.yml` file to match your OE package and dataset paths:

```yaml
    volumes:
      - ./open_explorer/horizon_xj3_open_explorer_{your_version}:/open_explorer
      - ./dataset:/data/horizon_xj3/data
```

If you downloaded the docker image from D-Robotics forum, please change the image version accordingly in the `docker-compose.yml` file.

```yaml
    image: openexplorer/ai_toolchain_ubuntu_20_xj3_cpu:{your_version}
``` 

To load the docker image from a tar file, use the command:

```bash
docker load -i <path_to_your_docker_image_tar_file>
```

### 2. Start the Container


Run the following command to start the container interactively and remove it after exit:

> if you wish keep the container after exit, `docker compose up` can be used instead.

```bash
docker compose run --rm x3_ai_toolchain
```

- This will launch a shell inside the container with the OE package and dataset mounted.

### 3. Stopping the Container

Simply exit the shell (Ctrl+D or `exit`) to stop and remove the container.

## Notes

- You can change the OE package version by updating the path in the `volumes` section.
- The container uses the latest version of `openexplorer/ai_toolchain_ubuntu_20_xj3_cpu` by default.
- For persistent changes, edit the `docker-compose.yml` file as needed.

## OE Package Setup

Attach to the docker container

> noramlly you should be already in the /open_explorer directory after entering the container, if not please `cd /open_explorer`

```bash
cd ddk/package/host
bash resolve.sh
bash install.sh
```

## Common Commands

### Check Onnx Model Command

Check model for Bernoulli2 architecture(your X3 accelerator):

```bash
hb_mapper checker --model-type onnx --march bernoulli2 --model <your model>.onnx
```

### PTQ Quantization Command

Run model conversion.  

```bash
# you should create a config.yaml file according to your model and dataset, refer to the official documentation for details.
hb_mapper makertbin --config config.yaml --model-type onnx
```

## Troubleshooting

- Ensure the OE package and dataset directories exist and have the correct permissions.
- If you encounter permission issues, try running Docker with `sudo` or adjust directory permissions.

### ONNX Opset Version Error

```bash
ERROR *** ERROR-OCCUR-DURING {horizon_nn.build_onnx} ***, error message: The opset version of the model is xx, the maximum supported version is 11
```

your ONNX model opset version is higher than the supported version (11). Please export your model to opset version 11 or lower using the following command:

> here you, yolo CLI people `yolo export model=yolo11n.pt format=onnx opset=11`

```python
torch.onnx.export(model, inputs, "model.onnx", opset_version=11)
```

Or convert the model(but this fail 90% time, better to re-export the model):

```python
import onnx
from onnx import version_converter
# Load your model
model = onnx.load("path/to/your_model.onnx")
# Convert the model to opset version 11
converted_model = version_converter.convert_version(model, 11)
# Save the converted model
onnx.save(converted_model, "your_model_opset11.onnx")
```

---

## Useful Links

- [Docker Hub - OpenExplorer AI Toolchain Docker Image(CPU Version)](https://hub.docker.com/r/openexplorer/ai_toolchain_ubuntu_20_xj3_cpu)

- [D-Robotics - OpenExplorer OE Packager and Docker Image Download Page](https://forum.d-robotics.cc/t/topic/28035)  
