# deepcluster

Python3 implementation of deepcluster by Facebook Research as described in the paper 
Deep Clustering for Unsupervised Learning of Visual Features 
[[paper](https://arxiv.org/abs/2006.09882), 
[code](https://github.com/facebookresearch/deepcluster)].

## Local Install

Setup virtual environment:
```
$ mkdir venv
$ python -m virtualenv venv
$ source venv/bin/activate
$ pip install faiss-cpu==1.6.3 --no-cache
$ pip install -r requirements.txt
```

## Docker

Download image from `591746741767.dkr.ecr.us-east-2.amazonaws.com/deepcluster:latest`


Before creating docker image, download the model checkpoint ([here](https://drive.google.com/file/d/1uOgJ6KXHbeg2c-MlzHk6TPp8SSqYaolv/view?usp=sharing)) 
 and save it to repo like `.../deepcluster/checkpoint.pth.tar`.

Create docker image:
```
$ bash docker_build.sh
```
Run scripts through docker (more on that bellow):
Note: mkdir `/path/to/save/experiment` before running any of these.
```
$ bash docker_scripts/test.sh /path/to/dataset [optional: /path/to/save/data]
$ bash docker_scripts/train.sh /path/to/dataset /path/to/save/experiment
$ bash docker_scripts/predict.sh /path/to/dataset /path/to/save/class_predictions
$ bash docker_scripts/train_fine_tune.sh /path/to/dataset /path/to/save/experiment
$ bash docker_scripts/train_top_softmax.sh /path/to/dataset /path/to/save/experiment
```

## Requirements

- Python3.6 +
- GPU
- Cuda 10.1

## Pretrained models

Download model from [here](https://drive.google.com/file/d/1uOgJ6KXHbeg2c-MlzHk6TPp8SSqYaolv/view?usp=sharing) and place it in side the repo like so `.../deepcluster/checkpoint.pth.tar` before building docker image.

For prediction classes and index are needed.

Download classes from [here](https://drive.google.com/file/d/1iDRRwjkNC1Pf4zD7OswGkFr163cjXo0l/view?usp=sharing). Already inside the repo, no need to download.

Download index from [here](https://drive.google.com/file/d/1EJtOQBbkroq43TvmQA2IHIPAluceFn0w/view?usp=sharing). Already inside the repo, no need to download.

## Dataset

Data used in development of this repo is Flickr25K and Flickr15K. You can download them from [here](https://cvssp.org/data/Flickr25K/cag17.html).

## Run

There are five scripts that you can run. Test, train, predict, fine_tune and softmax. Every script is described bellow.
To call any of the docker scripts, build docker image first.
```
$ bash docker_build.sh
```

For every script there are three ways of running them.
- docker call (ex. `$ bash docker_scripts/test.sh`)
- scripts dir call (ex. `$ bash scripts/test.py`)
- direct call (ex. `$ python test.py`)

All three ways are described for every script.

### Test

Test script takes pretrained model, computes features from given dataset, and runs KMeans clustering algorithm on those features.
If `save` parameter is set, it also saves images grouped by KMeans.

Note: `data` parameter needs to point to one level above directory with images. For example, data='/path/to/dataset', where images are in subdir like '/path/to/dataset/car/image_N.jpg'

Note: if `save` parameter is set, script will create one dir for every centroid. Only the centroids with dominating classes ( > 30% of the same class ) will have class name in dir name.
It is considered that all other centroids can't be assigned with a single class.

Note: If seeds are not set, you can expect slightly different results from call to call.
This is due to random initialization of centroids start position of KMeans algorithm.
With the given model you should expect around 77% accuracy on Flickr.

For docker run (build image first)
```
$ bash docker_scripts/test.sh /path/to/dataset [optional: /path/to/save/data]
```

For local run, setup `data`, `save` and `resume` parameters in scripts/test.sh script and run
```
$ bash scripts/test.sh
```
or call python script directly
```
$ python test.py [-h] [--data DATA] [--arch {alexnet,vgg16}] [--sobel]
                 [--nmb_cluster NMB_CLUSTER] [--cluster_alg {KMeans,PIC}]
                 [--batch BATCH] [--resume PATH]

arguments:
  -h, --help            show this help message and exit
  --data                Path to dataset
  --save                Path to dir where data will be saved, if empty nothing will be saved
  --arch                CNN architecture. [alexnet or vgg16], only vgg16 is supported for now
  --sobel               Sobel filtering
  --nmb_cluster (--k)   number of cluster for k-means (default: 1000)
  --cluster_alg         clustering algorithm. KMeans or PIC (default: KMeans)
  --batch               mini-batch size (default: 256)
  --resume              path to checkpoint (default: None)
```
### Train

Train script is really not used for this project. It's here and it works. Fine tune script is used instead.

For docker run (build image first)
```
$ bash docker_scripts/train.sh /path/to/dataset /path/to/save/experiment
```

For local run, setup `data`, `exp` and `resume` parameters in scripts/train.sh and run
```
$ bash scripts/train.sh
```
or call python script directly
```
$ python train.py [-h] [--data DATA] [--arch {alexnet,vgg16}] [--sobel]
                  [--nmb_cluster NMB_CLUSTER] [--cluster_alg {KMeans,PIC}]
                  [--batch BATCH] [--resume PATH] [--experiment PATH]
                  [--learning_rate LEARNING_RATE] [--weight_decay WEIGHT_DECAY]
                  [--workers WORKERS] [--start_epoch START_EPOCH]
                  [--epochs EPOCHS] [--momentum MOMENTUM]
                  [--checkpoints CHECKPOINTS] [--reassign REASSIGN] [--verbose]
                  [--dropout DROPOUT] [--seed SEED]

arguments:
  -h, --help            show this help message and exit
  --data                Path to dataset.
  --arch                CNN architecture. alexnet or vgg16
  --sobel               Sobel filtering
  --nmb_cluster, --k    number of cluster for k-means (default: 10000)
  --cluster_alg         clustering algorithm. KMeans or PIC (default: KMeans)
  --batch               mini-batch size (default: 256)
  --resume              path to checkpoint (default: None)
  --experiment, --exp   path to dir where train will be saved
  --learning_rate, --lr learning rate
  --weight_decay, --wd  weight decay
  --workers             number of data loading workers (default: 4)
  --start_epoch         manual epoch number (useful on restarts) (default: 0)
  --epochs              number of total epochs to run (default: 200)
  --momentum            momentum (default: 0.9)
  --checkpoints         how many iterations between two checkpoints (default: 25000)
  --reassign            how many epochs of training between two consecutive reassignments of clusters (default: 1)
  --verbose             chatty
  --dropout             dropout percentage in Dropout layers (default: 0.5
  --seed                random seed (default: None)

```

### Fine tune


Fine tune script is used for adjusting model parameters to Flickr25K dataset.

For docker run (build image first)
```
$ bash docker_scripts/train_fine_tune.sh /path/to/dataset /path/to/save/experiment
```

For local run, setup `data`, `exp` and `resume` parameters in scripts/train_fine_tune.sh and run
```
$ bash scripts/train_fine_tune.sh
```

or call python script directly
```
$ python train_fine_tune.py [-h] [--data DATA] [--arch {alexnet,vgg16}]
                            [--sobel] [--nmb_cluster NMB_CLUSTER]
                            [--cluster_alg {KMeans,PIC}] [--batch BATCH]
                            [--resume PATH] [--experiment PATH]
                            [--learning_rate LEARNING_RATE]
                            [--weight_decay WEIGHT_DECAY] [--workers WORKERS]
                            [--start_epoch START_EPOCH] [--epochs EPOCHS]
                            [--momentum MOMENTUM] [--checkpoints CHECKPOINTS]
                            [--reassign REASSIGN] [--verbose]
                            [--dropout DROPOUT] [--seed SEED]

arguments:
  -h, --help            show this help message and exit
  --data DATA           Path to dataset.
  --arch                CNN architecture. alexnet or vgg16
  --sobel               Sobel filtering
  --nmb_cluster, --k    number of cluster for k-means (default: 1000)
  --cluster_alg         clustering algorithm (default: Kmeans)
  --batch               mini-batch size (default: 256)
  --resume              path to checkpoint (default: None)
  --experiment, --exp   path to dir where train will be saved
  --learning_rate, --lr learning rate
  --weight_decay, --wd  weight decay
  --workers             number of data loading workers (default: 4)
  --start_epoch         manual epoch number (useful on restarts) (default: 0)
  --epochs              number of total epochs to run (default: 200)
  --momentum            momentum (default: 0.9)
  --checkpoints         how many iterations between two checkpoints (default: 25000)
  --reassign            how many epochs of training between two consecutive reassignments of clusters (default: 1)
  --verbose             chatty
  --dropout             dropout percentage in Dropout layers (default: 0.5
  --seed                random seed (default: None)
```


### Prediction

For docker run (build image first)
```
$ bash docker_scripts/predict.sh /path/to/dataset /path/to/save/class_predictions
```

For local run, setup `data`, `cluster_index`, `classes`, `checkpoint` and `save` parameters in scripts/predict.sh and run
```
$ bash scripts/prediction.sh
```
or call prediction.py like
```
$ python predict.py [-h] [--data DATA] [--cluster_index CLUSTER_INDEX]
                    [--checkpoint PATH] [--classes CLASSES] [--save SAVE]
                    [--arch {alexnet,vgg16}] [--sobel]
                    [--cluster_alg {KMeans,PIC}] [--batch BATCH] [--top_n TOP_N]

arguments:
  -h, --help            show this help message and exit
  --data                Path to directory to predict
  --cluster_index       path to clustering index file (default: cluster_index)
  --checkpoint          path to checkpoint (default: None)
  --classes             path to json file with classes description (default: classes.json)
  --save                path to dir to save results (default: <don't save>)
  --arch                CNN architecture. alexnet or vgg16 (default: vgg16)
  --sobel               Sobel filtering
  --cluster_alg         clustering algorithm (default: Kmeans)
  --batch               mini-batch size (default: 256)
  --top_n               top N for accuracy (default: 1)
```

### Train top layer (with Softmax)

For docker run (build image first)
```
$ bash docker_scripts/train_top_softmax.sh /path/to/dataset /path/to/save/experiment
```
For local run, setup `data`, `resume` and `experiment` parameters in scripts/train_top_softmax.sh and run
```
$ bash scripts/train_top_softmax.sh
```
or call train_top_softmax.py like
```
$ python train_top_softmax.py [-h] [--data DATA] [--arch {alexnet,vgg16}]
                              [--sobel] [--batch BATCH] [--resume PATH]
                              [--experiment PATH]
                              [--learning_rate LEARNING_RATE]
                              [--weight_decay WEIGHT_DECAY] [--workers WORKERS]
                              [--start_epoch START_EPOCH] [--epochs EPOCHS]
                              [--checkpoints CHECKPOINTS] [--verbose]
                              [--dropout DROPOUT] [--seed SEED]

arguments:
  -h, --help            show this help message and exit
  --data                Path to dataset.
  --experiment, --exp   path to dir where train will be saved
  --resume              path to checkpoint (default: None)
  --arch                CNN architecture
  --sobel               Sobel filtering
  --batch               mini-batch size (default: 256)
  --learning_rate, --lr learning rate
  --weight_decay, --wd  weight decay
  --workers             number of data loading workers (default: 4)
  --start_epoch         manual epoch number (useful on restarts) (default: 0)
  --epochs              number of total epochs to run (default: 200)
  --checkpoints         how many iterations between two checkpoints (default: 25000)
  --verbose             chatty
  --dropout             dropout percentage in Dropout layers (default: 0.5)
  --seed                random seed (default: None)
```
