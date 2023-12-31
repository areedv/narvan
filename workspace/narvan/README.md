# Manual notebook

## Set-up
Basic set up on a plain host (this might be simpler when using one of the docker containers provided by tensorflow?)

### Anaconda
Download from [this site](https://www.anaconda.com/products/individual) and run the script.

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

If GPU is available now is the time to set it up, see [here](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html#tf-install).

Install protobuf by first downloading the [appropriate zipfile ](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html#tf-install), unpack it and define the `protobuf/bin` directory in the PATH environment variable.

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
From within `TensorFlow/models/research/`:

```bash
cp object_detection/packages/tf2/setup.py .
python -m pip install .
```

Test the installation
```bash
python object_detection/builders/model_builder_tf2_test.py
```

## Process annotated data
Annotated data may be exported from roboflow as Pascal VOC format. If split already, just collect images and xml files under the image directory, delete train/test/valid foleder and use below script to re-split accordingly.
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
python generate_tfrecord.py -x ../../workspace/narvan/images/train -l ../../workspace/narvan/annotations/label_map.pbtxt -o ../../workspace/narvan/annotations/train.record
```

## Configure training job
Download a pre-trained model, for instance [this one](http://download.tensorflow.org/models/object_detection/tf2/20200711/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8.tar.gz).
and extract the archive content under `worspace/narvan/trained_models`.

Prepare a new model under the directory `workspace/narvan/models/narvan_ssd_mobilenet_v2_fpnlite_320x320` by copying the pipeline used for the pretrained model:
```bash
cp pre-trained-models/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8/pipeline.config models/narvan_ssd_mobilenet_v2_fpnlite_320x320/.
```

Adjust the pipeline accordingly.

Copy tf object detection training python script to this project (`workspace/narvan`):
```bash
cp ../../TensorFlow/models/research/object_detection/model_main_tf2.py .
```

and star the build from the `workspace/narvan` directory:
```bash
python model_main_tf2.py --model_dir=models/narvan_ssd_mobilenet_v2_fpnlite_320x320 --pipeline_config_path=models/narvan_ssd_mobilenet_v2_fpnlite_320x320/pipeline.config
```

Monitor progress with tensorboard:
```bash
tensorboard --logdir=models/narvan_ssd_mobilenet_v2_fpnlite_320x320
```
and open [localhost:6006](http://localhost:6006/)

## Export the model
Copy the python exporter script to `narvan/`:
```bash
cp ../../TensorFlow/models/research/object_detection/exporter_main_v2.py .
```
and extract the trained inference graph:
```bash
python ./exporter_main_v2.py --input_type image_tensor --pipeline_config_path ./models/narvan_ssd_mobilenet_v2_fpnlite_320x320/pipeline.config --trained_checkpoint_dir ./models/narvan_ssd_mobilenet_v2_fpnlite_320x320/ --output_directory ./exported-models/narvan_ssd_mobilenet_v2_fpnlite_320x320
```
