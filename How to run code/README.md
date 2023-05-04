## What is Copycat CNN?
Copycat CNN is a method by which to copy the learned parameters of a Deep Learning model knowing only the number of possible classifications. The method operates on the principle that Deep Learning models are trained on a specific set of data known as their problem domain. Deep learning models are then able to make predictions on data that they have not explicitly seen during training meaning they can draw conclusions on data outside of their problem domain. The method follows that by querying a Deep Learning model where only output labels are given and no access to the model or training data set one can create a fake dataset by feeding input and macthing labels from the target model being stolen. The learned parameters of this model can be stolen by then training another CNN on the fake dataset.

## Dependency set up   
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

## Target model
The target model is the model that is having its learned parameters stolen. We must first train a model to benchmark efficency of this method after theft occurs.   

Dataset utilized: [CIFAR10](https://www.cs.toronto.edu/~kriz/cifar.html)    
CNN model: $HOME/Copycat/Framework/oracle/model.py

### Training of the target model
This model will be trained and established as a benchmark. It will be the target model we intend to copy by querying it and assembling a fake dataset
      
5. Run scripts to train & test your target CNN Model 
```sh
$ # Training:
$ python3 oracle/train.py cifar_model.pth
$ # Testing:
$ python3 oracle/test.py cifar_model.pth
```
`cifar_model.pth`: Target model location

### Fake dataset input creation   

6. Create a directory to store input for the fake dataset
```
$ cd $HOME/Copycat
$ mkdir dimage
```   
7. Download and save to the $HOME/Copycat/dimage folder an assortment of images from your own source or:   
Download the [ImageNet](http://www.image-net.org/) images and, if you want, [Microsoft COCO](https://cocodataset.org) images.<br>     

8. Create a text file that has the file locations of each image for the purpose of querying our target model
```sh
$ cd $HOME/Copycat/
$ find $HOME/Copycat/dimage -type f | grep -i 'jpg\|jpeg\|png' > images.txt
```

9. Feed input images into the target model to query with our fake input the output being prediction labels from our target network we will use to create our fake dataset
```sh
$ python3 copycat/label_data.py cifar_model.pth images.txt stolen_labels.txt
$ python3 copycat/label_data.py cifar_model.pth images.txt stolen_labels.txt 64
```
10. Train and test our Copycat CNN with the fake dataset we just assembled
```sh
$ # Training:
$ python3 copycat/train.py copycat.pth stolen_labels.txt
$ # Testing:
$ python3 oracle/test.py copycat.pth
```

