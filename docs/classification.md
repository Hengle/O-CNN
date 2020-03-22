# Shape Classification

Change the working directory to `caffe/experiments/cls` for `Caffe` and
`tensorflow/scripts` for `TensorFlow`.


## O-CNN on Caffe

1. Download [ModelNet40](http://modelnet.cs.princeton.edu/ModelNet40.zip) dataset
and unzip it to the folder `dataset/ModelNet40`.
2. Convert triangle meshes (in `off` format) to point clouds (in `points` format)
with the [virtual_scanner](https://github.com/wang-ps/O-CNN/tree/master/virtual_scanner).
This process can be automatically executed by the following command.
Remember to provide actual `<The path of the virtual_scanner>` to run the command,
and replace the symbol `^` with `\` for multiple-line commands in shell.
We also provide the converted `point clouds` for convenience. Download the zip file
[here](https://www.dropbox.com/s/m233s9eza3acj2a/ModelNet40.points.zip?dl=0) and
unzip it to the folder `dataset/ModelNet40.points`.
    ```shell
    python prepare_dataset.py --run=m40_convert_mesh_to_points ^
                              --scanner=<The path of the virtual_scanner>
    ```
3. The generated point clouds are very dense, and if you would like to save disk
spaces, you can optionally run the following command to simplify the point cloud.
    ```shell
    python prepare_dataset.py --run=m40_simplify_points ^
                              --simplify_points=<The path of simplify_points>
    ```
4. Convert the point clouds to octrees, then build the `lmdb` database used by
`caffe` with the executive files [`octree`](Installation.md#Octree) and
[`convert_octree_data`](Installation.md#Caffe).
This process can be automatically executed by the following command.
Remember to provide actual `<The path of the octree>` and
`<The path of the convert_octree_data>`to run the command.
We also provide the converted `lmdb` for convenience. Download the zip file
[here](https://www.dropbox.com/s/t6d7z12ye3rpfit/ModelNet40.octree.lmdb.zip?dl=0)
and unzip it to the folder `dataset`.
    ```shell
    python prepare_dataset.py --run=m40_generate_ocnn_lmdb ^
                              --octree=<The path of the octree> ^
                              --converter=<The path of the convert_octree_data>
    ```
5. Run [`caffe`](Installation.md#Caffe) to train the model.
For detailed usage of [`caffe`](Installation.md#Caffe), please refer to the
official tutorial [Here](http://caffe.berkeleyvision.org/tutorial/interfaces.html).
We also provide our pre-trained Caffe model in `models/ocnn_M40_5.caffemodel`.
    ```shell
    caffe train  --solver=ocnn_m40_5_solver.prototxt  --gpu=0
    ```


## AO-CNN on Caffe

1. Follow the instructions [above](#o-cnn-on-caffe) untile the 3rd step.
The AO-CNN takes adaptive octrees for input, run the following command to prepare
the data automatically.
    ```shell
    python prepare_dataset.py --run=m40_generate_aocnn_lmdb ^
                              --octree=<The path of the octree> ^
                              --converter=<The path of the convert_octree_data>
    ```
2. Run [`caffe`](Installation.md#Caffe) to train the model.
    ```shell
    caffe train  --solver=aocnn_m40_5_solver.prototxt  --gpu=0
    ```


## O-CNN on TensorFlow

1. Follow the instructions [above](#o-cnn-on-caffe) untile the 3rd step.
The `Tensorflow` takes `TFRecords` as input, run the following command to
convert the point clouds to octrees, then build the `TFRecords` database
with the executive files [`octree`](Installation.md#Octree) and
[`convert_tfrecords.py`](../tensorflow/util/convert_tfrecords.py).
    ```shell
    python prepare_dataset.py --run=m40_generate_ocnn_octree_tfrecords \
                              --octree=<The path of the octree> \
                              --converter="../util/convert_tfrecords.py"
    ```

2. Run the following command to train the network:
    ```shell
    python run_cls.py configs/cls_octree.yaml
    ```

3. With `Tensorflow`, the network can also directly consume the points
as input and build octrees at runtime.
Run the following command to store the `points` into one `TFRecords` database.
    ```shell
    python prepare_dataset.py --run=m40_generate_ocnn_points_tfrecords \
                              --converter="../util/convert_tfrecords.py"
    ```
4. Run the following command to train the network which directly takes points.
    ```shell
    python run_cls.py configs/cls_points.yaml
    ```