# How to unfreeze and export tensorflow inference graph from object detection API using EC2 instance

#### Rui La	
####11/9/2018

The documentation is about training a TF object detection model on EC2 instance and export inference graph with and without frozne layers.

First create an EC2 instance
I created a ec2 instance using AWS deeplearning AMI and named it tf_od_installation
Then install different packages

Tensorflow object detection API depends on following libraries but most of libraries are already installed in EC2 deeplearning AMI

*   Protobuf 3.0.0
*   Python-tk
*   Pillow 1.0
*   lxml
*   tf Slim (which is included in the "tensorflow/models/research/" checkout)
*   Jupyter notebook
*   Matplotlib
*   Tensorflow (>=1.9.0)
*   Cython
*   contextlib2
*   cocoapi

## environment setup
Type these commands line by line or you can use bash file in this repo.

```bash
sudo su
yum update && yes | yum upgrade 
pip install Cython
yum install -y protobuf-compiler python-pil python-lxml
yum install python-tk
mkdir -p tensorflow
cd tensorflow
# Download tensorflow models 
git clone https://github.com/tensorflow/models.git
cd models/research
```


## Protobuf Compilation

The Tensorflow Object Detection API uses Protobufs to configure model and
training parameters. Before the framework can be used, the Protobuf libraries
must be compiled. This should be done by running the following command from
the tensorflow/models/research/ directory:

Download and install the 3.0 release of protoc, then unzip the file.

```bash
# From tensorflow/models/research/
wget -O protobuf.zip https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip
unzip protobuf.zip
```

Run the compilation process again, but use the downloaded version of protoc

```bash
# From tensorflow/models/research/
./bin/protoc object_detection/protos/*.proto --python_out=.
```

```bash
# From tensorflow/models/research/
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```

Now the environment is set up, we can exit sudo mode and then activate conda environment to run the testing code. 

```bash
# From tensorflow/models/research/
exit
source activate tensorflow_p27
```

Note: This command needs to run from every new terminal you start. If you wish
to avoid running this manually, you can add it as a new line to the end of your
~/.bashrc file, replacing \`pwd\` with the absolute path of
tensorflow/models/research on your system.


## Testing the Installation

You can test that you have correctly installed the Tensorflow Object Detection\
API by running the following command:


```bash
# From tensorflow/models/research/
sudo ./bin/protoc object_detection/protos/*.proto --python_out=.
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
python object_detection/builders/model_builder_test.py
```

if you see "OK" at the end of running, that means the environment installation for Tensorflow API is ready

## Export unfrozen inference graph

I have my trained checkpoint saved at [here](https://github.com/larui529/tensorflow-object-detection-docker)

To export unfrozen inference graph, we need to download it and replace the original "exporter.py" file.

```bash
# From tensorflow/models/research/object_detection/
sudo su
mkdir training
cd training
git clone https://github.com/larui529/tensorflow-object-detection-docker.git 
cp tensorflow-object-detection-docker/training/* .
cp tensorflow-object-detection-docker/exporter.py ../
exit
```

Finally export unfrozen inference graph to output_directory. 

```bash
# From tensorflow/models/research/
sudo chmod -R 777 ~/tensorflow/*
python export_inference_graph.py --input_type image_tensor --pipeline_config_path training/faster_rcnn_inception_v2_pets.config --trained_checkpoint_prefix training/model.ckpt-5736 --output_directory inference_graph
```


NOTE: We are configuring our exported model to ingest 4-D image tensors. We can
also configure the exported model to take encoded images or serialized
`tf.Example`s.

After export, you should see the directory inference_graph containing the following:

* saved_model/, a directory containing the saved model format of the exported model
* frozen_inference_graph.pb, the frozen graph format of the exported model
* model.ckpt.*, the model checkpoints used for exporting
* checkpoint, a file specifying to restore included checkpoint files
* pipeline.config, pipeline config file for the exported model

/inference_graph/
      |-- checkpoint
      |-- frozen_inference_graph.pb
      |-- model.ckpt.data-00000-of-00001
      |-- model.ckpt.index
      |-- model.ckpt.meta
      |-- pipeline.config
      |-- saved_model
            |-- saved_model.pb
            |-- variables
                  |-- variables.data-00000-of-00001 
                  |-- variables.index

folder of saved_model should be in same structure as export_savedmodel from tensorflow estimator which should be able used for inference serving. 