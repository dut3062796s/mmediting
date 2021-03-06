## Prepare Datasets for Matting

### Adobe Composition-1k Dataset

It is recommended to symlink the Adobe Composition-1k (comp1k) dataset root, the [MS COCO dataset](http://cocodataset.org/#home) and the [PASCAL VOC dataset](http://host.robots.ox.ac.uk/pascal/VOC/) to `$MMEditing/data`:

```
mkdir data
ln -s $ADOBE_COMPOSITION_1K_ROOT data/adobe_composition-1k
ln -s $COCO_ROOT data/coco
ln -s $VOC_ROOT data/VOCdevkit
```

The result folder structure should look like:

```
mmediting
├── mmedit
├── tools
├── configs
├── data
│   ├── adobe_composition-1k
│   │   ├── Test_set
│   │   │   ├── Adobe-licensed images
│   │   │   │   ├── alpha
│   │   │   │   ├── fg
│   │   │   │   ├── trimaps
│   │   │   ├── merged  (generated by tools/preprocess_comp1k_dataset.py)
│   │   │   ├── bg      (generated by tools/preprocess_comp1k_dataset.py)
│   │   ├── Training_set
│   │   │   ├── Adobe-licensed images
│   │   │   │   ├── alpha
│   │   │   │   ├── fg
│   │   │   ├── Ohter
│   │   │   │   ├── alpha
│   │   │   │   ├── fg
│   │   │   ├── merged  (generated by tools/preprocess_comp1k_dataset.py)
│   │   │   ├── bg      (generated by tools/preprocess_comp1k_dataset.py)
│   ├── coco
│   │   ├── train2014   (or train2017)
│   ├── VOCdevkit
│   │   ├── VOC2012
```

If your folder structure is different, you may need to change the corresponding paths in config files.

The Adobe composition-1k dataset contains only `alpha` and `fg` (and `trimap` in test set). It is needed to merge `fg` with COCO data (training) or VOC data (test) before training or evaluation. A script is provided to perform image composition and generate annotation files for training or testing:

```shell
python tools/preprocess_comp1k_dataset.py data/adobe_composition-1k data/coco data/VOCdevkit --composite
```

The generated data is stored under `adobe_composition-1k/Training_set` and `adobe_composition-1k/Test_set` respectively. If you only want to composite test data (since compositing training data is time-consuming), you can remove the `--composite` option:

```shell
python tools/preprocess_comp1k_dataset.py data/adobe_composition-1k data/coco data/VOCdevkit
```

> Currently, only `GCA` supports online composition of training data. But you can modify the data pipeline of other models to perform online composition instead of loading composited images (we called it `merged` in our data pipeline).

### Note

We use the same function to save the generated images as the original script provided by Adobe which will generate png images with incorrect png profile. Thus, it's normal to observe the below warning when training model with single GPU:

```
libpng warning: iCCP: known incorrect sRGB profile
libpng warning: iCCP: profile 'ICC Profile': 'GRAY': Gray color space not permitted on RGB PNG
libpng warning: iCCP: profile 'ICC Profile': 1000000h: invalid rendering intent
libpng warning: iCCP: profile 'ICC Profile': 0h: PCS illuminant is not D50
libpng warning: iCCP: profile 'ICC Profile': 'desc': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'wtpt': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'bkpt': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'rXYZ': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'gXYZ': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'bXYZ': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'dmnd': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'dmdd': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'vued': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'view': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'lumi': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'meas': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'tech': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'rTRC': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'gTRC': ICC profile tag start not a multiple of 4
libpng warning: iCCP: profile 'ICC Profile': 'bTRC': ICC profile tag start not a multiple of 4
```

Note that we are also putting effort to remove these warnings for better training experience. Thus, any suggestions or contributions are welcomed.

If you find it hard to inspect the training process in CLI, it's recommended to use the `Tensorboard` to keep track on the training process. To do this, you can just simply uncomment the `TensorboardLoggerHook` in model config file.

```python
log_config = dict(
    interval=10,
    hooks=[
        dict(type='TextLoggerHook', by_epoch=False),
        dict(type='TensorboardLoggerHook'),
        # dict(type='PaviLoggerHook', init_kwargs=dict(project='dim'))
    ])
```

Optionally, you can also view all the loss log in `work_dir/${EXP_NAME}/${EXP_CREATE_TIME}.log.json`.
