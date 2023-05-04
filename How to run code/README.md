## What is Copycat CNN?
Copycat CNN is a method by which to copy the learned parameters of a Deep Learning model knowing only the number of possible classifications. The method operates on the principle that Deep Learning models are trained on a specific set of data known as their problem domain. Deep learning models are then able to make predictions on data that they have not explicitly seen during training meaning they can draw conclusions on data outside of their problem domain. The method follows that by querying a Deep Learning model where only output labels are given and no access to the model or training data set one can create a fake dataset by feeding input and macthing labels from the target model being stolen. The learned parameters of this model can be stolen by then training another CNN on the fake dataset.

### Dependency set up   
__Ubuntu:__ 22.04>=, Python >=3.10   
__Jetson Nano:__ Jetpack version >= 5.1 w/ CUDA toolkit, Python >=3.9   
   
1. Create main folder to work out of
```sh
$ mkdir $HOME/Copycat
```
2. Download the Framework folder in this Github and extract to the previously made directory $HOME/Copycat
   
3. Set up virtual enviornment and install needed packages
```sh
$ python3 -m venv $HOME/Copycat/venv 
$ source $HOME/Copycat/venv/bin/activate
$ pip3 install matplotlib numpy scikit-learn torch torchvision tqdm
```
4. Verify CUDA toolkit install
```
$ python3
>>> import torch
>>> print(torch.cuda.is_avaliable())
>>> exit()
```
[If the above returns True, the calculations will run through an enabled Nvidia graphics module, if not calculations will run on the CPU where a core dump can occur if the dependency set up versions are not compatible]

On this example, we are using the [CIFAR10](https://www.cs.toronto.edu/~kriz/cifar.html) as [dataset](./oracle/cifar_data.py) and a simple [model](./oracle/model.py). <br>
But feel free to change the code and test other dataset and model. <br>
The same model structure is being used to [Oracle](./oracle/model.py) and for [Copycat](./copycat/model.py), but it is not necessary. <br>
But to avoid constraints caused by the model structure (for example, a smaller model may not be able to copy a larger one), you must use the same model structure for both networks.

### Training of the target model
This model will be trained and established as a benchmark. It will be the target model we intend to copy by querying it and assembling a fake dataset
```sh
$ # Training:
$ python oracle/train.py cifar_model.pth
$ # Testing:
$ python oracle/test.py cifar_model.pth
```
The file `cifar_model.pth` will be created and will be our _target model_.

Now, it is time to extract the labels from our _target model_.<br>
Download the [ImageNet](http://www.image-net.org/) images and, if you want, [Microsoft COCO](https://cocodataset.org) images.<br>
Our tests used 3M images, but feel free to test with more or less images.

After download the images and place them on a directory, you have to create a _txt_ file with their location.<br>
It is easy to make by using this command:
```sh
$ cd $HOME/Copycat/Framework
$ find DIRECTORY_WHERE_YOU_DOWNLOADED_THE_IMAGES -type f | grep -i 'jpg\|jpeg\|png' > images.txt
```

With the images, we are ready to extract the labels from _target model_:
```sh
$ python copycat/label_data.py cifar_model.pth images.txt stolen_labels.txt
$ # If you need to change the batch size, inform it as an additional parameter.
$ # Example for batch_size=64:
$ python copycat/label_data.py cifar_model.pth images.txt stolen_labels.txt 64
```
The file stolen_labels.txt will be created and now we can train our Copycat network.<br>
If the process provided some error processing some image, remove the image filename from images.txt and try again.

Finally, we can create our Copycat model and test it:
```sh
$ # Training:
$ python copycat/train.py copycat.pth stolen_labels.txt
$ # Testing:
$ python oracle/test.py copycat.pth
```

The Macro Average can be used to compare the quality of the attack.
