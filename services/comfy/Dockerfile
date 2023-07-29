FROM alpine:3.17 as xformers
RUN apk add --no-cache aria2
RUN aria2c -x 5 --dir / --out wheel.whl 'https://github.com/AbdBarho/stable-diffusion-webui-docker/releases/download/6.0.0/xformers-0.0.21.dev544-cp310-cp310-manylinux2014_x86_64-pytorch201.whl'

FROM pytorch/pytorch:2.0.1-cuda11.7-cudnn8-runtime

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN apt-get update && apt-get install -y git && apt-get clean

ENV ROOT=/stable-diffusion
RUN --mount=type=cache,target=/root/.cache/pip \
  git clone https://github.com/comfyanonymous/ComfyUI.git ${ROOT} && \
  cd ${ROOT} && \
  git checkout master && \
  git reset --hard 884ea653c8d6fe19b3724f45a04a0d74cd881f2f && \
  pip install -r requirements.txt


RUN --mount=type=cache,target=/root/.cache/pip  \
  --mount=type=bind,from=xformers,source=/wheel.whl,target=/xformers-0.0.21-cp310-cp310-linux_x86_64.whl \
  pip install /xformers-0.0.21-cp310-cp310-linux_x86_64.whl

RUN apt-get update && apt-get install -y wget libgl1-mesa-dev libglib2.0-0

### install ComfyUI extensions ###
WORKDIR $ROOT/custom_nodes
## WAS Node Suite - ComfyUI
RUN git clone https://github.com/WASasquatch/was-node-suite-comfyui/ && \
  pip install -r $ROOT/custom_nodes/was-node-suite-comfyui/requirements.txt
## ComfyUI WD 1.4 Tagger (https://github.com/pythongosssss/ComfyUI-WD14-Tagger
RUN git clone https://github.com/pythongosssss/ComfyUI-WD14-Tagger ComfyUI-WD14-Tagger && \
  pip install -r $ROOT/custom_nodes/ComfyUI-WD14-Tagger/requirements.txt
## ControlNet Preprocessors for ComfyUI https://github.com/Fannovel16/comfy_controlnet_preprocessors
RUN git clone https://github.com/Fannovel16/comfy_controlnet_preprocessors && \
  cd comfy_controlnet_preprocessors && \
  python install.py --no_download_ckpts && \
  # mkdir -p $ROOT/stable-diffusion/controlnet_pre_ckpts/ckpts/ && \
  cd ..  
COPY controlnet_preprocessors_config.yaml $ROOT/custom_nodes/comfy_controlnet_preprocessors/config.yaml
## pythongosssss's ComfyUI-Custom-Scripts (https://github.com/pythongosssss/ComfyUI-Custom-Scripts)
RUN git clone https://github.com/pythongosssss/ComfyUI-Custom-Scripts.git ComfyUI-Custom-Scripts && \
  # current revision is fucked 
  cd ComfyUI-Custom-Scripts && \
  git reset --hard 5deac16327ea9684cfd1b89d14d9dc0bcbe0ff7c && \
  cd .. && \
  # Anime Segmentation
  cd ${ROOT}/comfy_extras && \
  git clone https://github.com/SkyTNT/anime-segmentation.git anime_segmentation && \
  mkdir ${ROOT}/comfy_extras/anime_segmentation/saved_models && \
  cd ${ROOT}/comfy_extras/anime_segmentation/saved_models && \
  wget https://huggingface.co/skytnt/anime-seg/resolve/main/isnetis.ckpt && \
  pip install -r ${ROOT}/comfy_extras/anime_segmentation/requirements.txt && \
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/anime-segmentation/anime_segmentation.py $ROOT/custom_nodes/ && \
  #Auto Arrange Graph
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/auto-arrange-graph/graphArrange.js $ROOT/web/extensions/ && \
  # Workflow SVG
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/export-workflow-svg/exportAsSvg.js $ROOT/web/extensions/ && \
  # Favicon Status
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/favicon-status/* $ROOT/web/extensions/ && \
  # Image Feed
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/image-feed/imageFeed.js $ROOT/web/extensions/ && \
  # Latent Upscale By
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/latent-upscale-by/latent_upscale_by.py $ROOT/custom_nodes/ && \
  # Lora Subfolders
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/lora-subfolders/loraSubfolders.js $ROOT/web/extensions/ && \
  # Node Finder
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/node-finder/nodeFinder.js $ROOT/web/extensions/ && \
  # Preset Text
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/preset-text/presetText.js $ROOT/web/extensions/ && \
  # Quick Nodes
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/quick-nodes/quickNodes.js $ROOT/web/extensions/ && \
  # Show Text
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/show-text/show_text.py $ROOT/custom_nodes/ && \
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/show-text/showText.js $ROOT/web/extensions/ && \
  # Widget Defaults
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/widget-defaults/widgetDefaults.js $ROOT/web/extensions/ && \
  # Workflows
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/workflows/workflows.py $ROOT/custom_nodes/ && \
  cp $ROOT/custom_nodes/ComfyUI-Custom-Scripts/workflows/index.js $ROOT/web/extensions/ && \
  rm -rf $ROOT/custom_nodes/ComfyUI-Custom-Scripts
#OpenPose Editor for ComfyUI https://github.com/space-nuko/ComfyUI-OpenPose-Editor
RUN git clone https://github.com/space-nuko/ComfyUI-OpenPose-Editor.git ComfyUI-OpenPose-Editor && \
  cp ComfyUI-OpenPose-Editor/js/* $ROOT/web/extensions/

WORKDIR ${ROOT}

ARG BRANCH=master SHA=8607c2d42d10b0108de02528e813cc703e58813f
RUN --mount=type=cache,target=/root/.cache/pip \
  git fetch && \
  git checkout ${BRANCH} && \
  git reset --hard ${SHA} && \
  pip install -r requirements.txt

# add info
COPY . /docker/
RUN cp /docker/extra_model_paths.yaml ${ROOT}
RUN chmod u+x /docker/entrypoint.sh
RUN cp /docker/extra_model_paths.yaml ${ROOT}

ENV NVIDIA_VISIBLE_DEVICES=all
ENV PYTHONPATH="${PYTHONPATH}:${PWD}" CLI_ARGS=""
EXPOSE 7860
ENTRYPOINT ["/docker/entrypoint.sh"]
CMD python -u main.py --listen --port 7860 ${CLI_ARGS}
