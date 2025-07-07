## Getting Started with EDSOD



### Installation

The codebases are built on top of [Detectron2](https://github.com/facebookresearch/detectron2), [Sparse R-CNN](https://github.com/PeizeSun/SparseR-CNN), and [denoising-diffusion-pytorch](https://github.com/lucidrains/denoising-diffusion-pytorch).
Thanks very much.

#### Requirements
- Linux or macOS with Python ≥ 3.6
- PyTorch ≥ 1.9.0 and [torchvision](https://github.com/pytorch/vision/) that matches the PyTorch installation.
  You can install them together at [pytorch.org](https://pytorch.org) to make sure of this
- OpenCV is optional and needed by demo and visualization

#### Steps
1. Install Detectron2 following https://github.com/facebookresearch/detectron2/blob/main/INSTALL.md#installation.

2. Prepare datasets
```
mkdir -p datasets/coco

ln -s /path_to_coco_dataset/annotations datasets/coco/annotations
ln -s /path_to_coco_dataset/train2017 datasets/coco/train2017
ln -s /path_to_coco_dataset/val2017 datasets/coco/val2017
```

You need to download the VisDrone dataset from its [official website](https://aiskyeye.com/)

You need to download the UAVDT dataset from its [official website](https://sites.google.com/view/grli-uavdt/%E9%A6%96%E9%A1%B5/)

3. Prepare pretrain models

EDSOD uses three backbones including ResNet-50, ResNet-101 and Swin-Base. The pretrained ResNet-50 model can be
downloaded automatically by Detectron2. We also provide pretrained
[ResNet-101](https://github.com/ShoufaChen/DiffusionDet/releases/download/v0.1/torchvision-R-101.pkl) and
[Swin-Base](https://github.com/ShoufaChen/DiffusionDet/releases/download/v0.1/swin_base_patch4_window7_224_22k.pkl) which are compatible with
Detectron2. Please download them to `EDSOD_ROOT/models/` before training:

```bash
mkdir models
cd models
# ResNet-101
wget https://github.com/ShoufaChen/DiffusionDet/releases/download/v0.1/torchvision-R-101.pkl

# Swin-Base
wget https://github.com/ShoufaChen/DiffusionDet/releases/download/v0.1/swin_base_patch4_window7_224_22k.pkl

cd ..
```

Thanks for model conversion scripts of [ResNet-101](https://github.com/PeizeSun/SparseR-CNN/blob/main/tools/convert-torchvision-to-d2.py)
and [Swin-Base](https://github.com/facebookresearch/Detic/blob/main/tools/convert-thirdparty-pretrained-model-to-d2.py).

4. Train EDSOD
```
python train_visdrone.py --num-gpus 4 --config-file configs/diffdet.visdrone.swinbase.500boxes.yaml
```

5. Evaluate EDSOD
```
python train_visdrone.py --num-gpus 4 \
    --config-file configs/diffdet.visdrone.swinbase.500boxes.yaml \
    --eval-only MODEL.WEIGHTS path/to/model.pth
```

* Evaluate with arbitrary number (e.g 300) of boxes by setting `MODEL.DiffusionDet.NUM_PROPOSALS 300`.
* Evaluate with 4 refinement steps by setting `MODEL.DiffusionDet.SAMPLE_STEP 4`.


We also provide the [pretrained model of VisDrone](https://drive.google.com/file/d/1BKGA5oNSmmRclzZ8YXiQeo4Ewa40A3Nc/view?usp=drive_link) and [pretrained model of UAVDT](https://drive.google.com/file/d/1w47oyHjvptjZd1s4CqaYg-urIuIrZ5oV/view?usp=drive_link)
of [EDSOD-visdrone-swinbase-500boxes](configs/diffdet.visdrone.swinbase.500boxes.yaml) and [EDSOD-uavdt-swinbase-500boxes](configs/diffdet.uavdt.swinbase.500boxes.yaml) that are used for ablation study.

6. Deformable DETR + iterative refinement + two stage

python -m torch.distributed.launch --nproc_per_node=8 --use_env main.py --coco_path /YOUR_PATH/coco --resume ../r50_deformable_detr_plus_iterative_bbox_refinement_plus_plus_two_stage-checkpoint.pth --eval --with_box_refine --two_stage --num_queries 500

### Inference Demo with Pre-trained Models
We provide a command line tool to run a simple demo following [Detectron2](https://github.com/facebookresearch/detectron2/tree/main/demo#detectron2-demo).

```bash
python demo.py --config-file configs/diffdet.visdrone.swinbase.500boxes.yaml \
    --input image.jpg --opts MODEL.WEIGHTS edsod_visdrone_swinbase_500boxes.pth
```

We need to specify `MODEL.WEIGHTS` to a model from model zoo for evaluation.
This command will run the inference and show visualizations in an OpenCV window.

For details of the command line arguments, see `demo.py -h` or look at its source code
to understand its behavior. Some common arguments are:
* To run __on your webcam__, replace `--input files` with `--webcam`.
* To run __on a video__, replace `--input files` with `--video-input video.mp4`.
* To run __on cpu__, add `MODEL.DEVICE cpu` after `--opts`.
* To save outputs to a directory (for images) or a file (for webcam or video), use `--output`.
