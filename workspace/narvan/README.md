# Manual notebook

## Set-up
Basic set up on a plain host (this might be simpler when using a TF docker container?)

### Anaconda
Download from https://www.anaconda.com/products/individual and run the script

### Create a new conda environment and activate it
```bash
conda create -n tensorflow python=3.9
conda activate tensorflow
pip install --ignore-installed --upgrade tensorflow==2.5.0
```

Verify the installarion of tf:
```bash
python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

If GPU is available now is the time to set it up, see https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html#tf-install

Install protobuf from https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html#tf-install unpack and define protobuf/bin directory in PATH env var

In a new terminal, compile protobuf:
```bash
cd TensorFlow/models/research
protoc object_detection/protos/*.proto --python_out=.
```

### COCO API installation (not needed if the below step works as intendend)
To be added to TensorFlow/models/research:
```bash
cd ~/Downloads
git clone gh:cocodataset/cocoapi
cd cocoapi/PythonAPI
make
cp -r pycocotools ~/git/narvan/TensorFlow/models/research/
```

### Install Object Detection API
```bash
# From within TensorFlow/models/research/
cp object_detection/packages/tf2/setup.py .
python -m pip install .
```

Test the installation
```bash
python object_detection/builders/model_builder_tf2_test.py
```

## Process annotated data
Annotated data exporter from roboflow as Pascal VOC format. If split already, just collect images and xml files under the image directory, delete train/test/valid foleder and use below script to re-split accordingly.
```bash
cd scripts/preprocessing
python partition_dataset.py -x -i /home/are/git/narvan/workspace/narvan/images -r 0.1
```

## Create lable map
The file `label_map.pbtxt` can be obtained by exporting annotation data as tfrecord and copy this file into the `annotation` directory.

## Create TensorFlow records
For this, pandas is needed:
```bash
pip install pandas
```

Then, create TFRecord for both training and testing:
```bash
python generate_tfrecord.py -x ../../workspace/narvan/images/test -l ../../workspace/narvan/annotations/label_map.pbtxt -o ../../workspace/narvan/annotations/test.record
```
