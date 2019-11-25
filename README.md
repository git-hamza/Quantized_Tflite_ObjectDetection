# Quantized_Tflite_ObjectDetection
8-bit Quantized tflite format model, ready to be compiled for Edge TPU (Coral Accelerator)

Directory structure for Quantized_Tflite_ObjectDetection.
Please keep all the folder names as it is mentioned. e.g. annotation, images etc. It will save you a lot of time.

## Installation:
Install the Tensorflow Object Detection API dependencies from the following link https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md.
## Directories Before Training:
###### config directory : 
Copy the .config file of the model you are about to use in this directory.
###### data directory : 
You need to copy you data images and annotations to this directory. Data directory contain two *.py* files. *xml_to_csv.py* converts the annotations to csv file named as *train.csv*. *generate_tfrecord.py* file convert csv files to tf record file named as *train.record*. Same needs to be done for test/val data.
###### inputs directory : 
inputs directory must have three things before starting the training i.e. *label_map.pbtxt* (labels of the classes). *train.record* and *val.record* files.
###### Softlink object_detection directory from models directory:
Clone the models from the tensorflow git repository to your home directory through `git clone https://github.com/tensorflow/models`. From there we will make softlink of object_detection to our repository directory(where our directories and executable file lies). Use the following command to make softlink 
`ln -s /home/hamza/models/research/object_detection /home/hamza/Quantized_Tflite_ObjectDetection`
It actually work as ` ln -s file_address address_where_to_link`.
###### Pre-trained_model: 
Training model for the first time, you need the checkpoints of a pretrained model that you are using. You can download it from here https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md . What I did here is, I made a directory in home by the name of pre-trained model where I keep all my pre-trained model directories. From there I load their checkpoint into the configuration file. That way it is easy and more useful. As you are about to train a quantize model, either download *ssd_mobilenet_v1_quantized_coco* or *ssd_mobilenet_v2_quantized_coco*.
## Executable files:
###### run_train : 
it is an executable file that uses train.py to train the model and take configuration file as an input please set the input of configuration file as *config/FILE_NAME.config*. it also make a directory named as train where it saves all the checkpoints. It is designed in such a way through confiuration file, if training process is stopped in midway and few checkpoints are saved in the train directory. If you start the training again, it will continue from the recent checkpoints.
###### run_eval : 
it is an executable file that uses eval.py to evaluate the model from recent checkpoint. The recent checkpoint is loaded from train directory in it. How the evaluation will be performed are set at the end of the configuration file. It saves the output in a directory name eval.
###### run_exp_tflite_graph : 
it is an executable file that uses export_tftlite_ssd_graph.py script and trained checkpoints from the train directory (You have to specify which checkpoint). It outputs frozen model tflite_graph.pb that can be converted to tflite and tflite_graph.pbtxt in the *tflite* directory.
###### run_convert_tflite : 
it is an executable file that takes tflite_graph.pb as an input and outputs a quantized 8-bit tflite format file i.e. model.tflite which is ready for inference.
## Things to look for before training:
###### configuration file: 
Check the number of classes in the configuration file to make sure you are training for the amount that is written in this file. Check the checkpoint address, *train.record* and *label_map.pbtxt* addresses in training and 
evaluation part. Check the graph_rewriter function at the end of config file that is responsible for quantization.
###### generate_tfrecord.py: 
Please check *class_text_to_int(row_labels)* function in the file. It is the first function in the file and it assign the labels to the classes. Please make sure you have correctly label for every class that you have given in *label_map.pbtxt* file.
###### label_map.pbtxt : 
This file contain dictionary where labels are assigned to each class. Please check the label with those you provided in *generate_tfrecord.py*.
Reference: https://gist.github.com/apivovarov/eff80275d0f72e4582c105921919b852
## Running process :
After setting the directory structure with the names mentioned in this repository, you can run the following commands step by step.
Note: if the *run_train*, *run_eval* or *run_export* is not running through those command, it mean they are not executable yet. to make them exeuctable try this command `chmod +x FILE_NAME`.
```
1. python3 xml_to_csv.py
2. python3 generate_tfrecord.py --csv_input=train.csv  --output_path=../inputs/train.record
3. python3 generate_tfrecord.py --csv_input=val.csv  --output_path=../inputs/val.record
4. ./run_train
5. ./run_eval
6. ./run_exp_tflite_graph
7. ./run_convert_tflite
```
## Directories After Training:
###### train: 
After you run the executable *./run_train* successfully. A directory by the name of train will be created in you main directory that will contain all you saved checkpoints.
###### eval : 
After training, you can run the executable *./run_eval* which will create a directory by the name of eval and it will have the evaluted images from the *test/val*.
###### tflite : 
After training, when you run the executable *./run_exp_tflite_graph*. It will create a directory by the name of tflite and output *tflite_graph.pb and *tflite_graph.pbtxt* in the directory. After running the executable *./run_convert_tflite* it will output a *model.tflite* in this directory.
