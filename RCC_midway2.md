## Deep-learning-practice using the RCC Midway2 cluster

### Grabbing a gpu2 node from midway2.rcc.uchicago.edu (after login)
Asking for 1 GPU, 1 CPU, 16GB RAM for 1 hour
```sh
sinteractive -p gpu2 --nodes=1 --gres=gpu:1 --ntasks-per-node=1 --mem-per-cpu=8000 --time=1:00:00
```

### Monitoring CPU, GPU, RAM usage
Once logged in to a gpu2 node, open two terminals.
```sh
xterm &
xterm &
```
In one terminal, run htop to monitor the CPU/RAM usage. Then press F4 for filter and enter the user id
```sh
htop
```

In the other terminal, run nvidia-smi to monitor the GPU usage
```sh
nvidia-smi -l 1
```

### Using pytorch + fastai on top of the CUDA library
1. To use the pytorch + fastai library, see https://github.com/fastai/fastai and install fastai

2. If installed, Load the necessary modules to use pytorch + fastai
```sh
module load cuda/9.0
module load Anaconda3/5.0.1
conda activate fastai
```

3. To see if the pytorch works, download the pytorch examples and run. This should run the MNIST classification.
```sh
git clone https://github.com/pytorch/examples.git
cd pytorch_examples/mnist
python main.py
```

### Using jupyter notebook on the gpu2 node
For jupyter notebook related instructions, see https://git.rcc.uchicago.edu/ivy2/Jupyter_on_compute_nodes

1. Load Anaconda3 module
2. Find out an ip address of the node:
```sh
(fastai) [kywch@midway2-gpu01 ~]$ ifconfig eth0 | grep 'inet '
        inet 10.50.221.191  netmask 255.255.252.0  broadcast 10.50.223.255
```
3. Start jupyter without launching a browser on the node (add & at the end to run in the background), using the ip address obtained above: 
```sh
(fastai) [kywch@midway2-gpu01 ~]$ jupyter notebook --no-browser --ip=10.50.221.191 &
```
4. By default, it would listen on port 8888. However, if the port is already taken by another user, it will complain. Try the next available port with --port=<port number> option.

5. Eventually, you'll get a URL with a token. For example: http://10.50.221.191:8888/?token=2af56958386caf78d5bdc2086b20b6ff18553b701c581cd3

6. Point the browser running on your laptop/desktop to this URL. Notice: compute nodes are only visible on internal uchicago network. Therefore, you either need to be on campus or use VPN to be able to use this method.

7. **To use GPU-accelerated libraries in jupyter notebook, use Python [conda env:DL_GPU] kernel.** (although, I'm not sure. The default kernel seems to use cuda. However, you can check -->) In the notebook, check whether it works by running.
```python
import torch
torch.cuda.is_available()       # Should be True
torch.backends.cudnn.enabled    # Should be True
```

8. To kill jupyter, ps to get the pid and use kill command.


### Performance benchmarks: CIFAR10 training time
Using a gpu2 node: 1 Tesla K80 gpu and 2 CPU cores

* Custom Wide Resnet + fastai + pytorch single gpu (https://github.com/fastai/imagenet-fast/tree/master/cifar10): took 1:21:03 vs. 0:06:45 on Paperspace Volta (V100)
```sh
(fastai) [kywch@midway2-gpu01 cifar10]$ python dawn_mod.py data --save-dir data/cf_train_save/wrn_v5 -a wrn_22 --fp16 --loss-scale 512 --epochs 1 --cycle-len 30 --lr 1.5 --wd 1e-4 --use-clr 20,20,0.95,0.85

Epoch:   0%|                                                                       | 0/30 [00:00<?, ?it/s]epoch      trn_loss   val_loss   accuracy                                                                 
    0      1.568049   2.731405   0.3089    
Epoch:   3%|##                                                          | 1/30 [03:19<1:36:12, 199.06s/it]    1      1.171877   1.276788   0.5774                                                                   
Epoch:   7%|####                                                        | 2/30 [06:06<1:25:27, 183.13s/it]    2      0.920082   0.970986   0.6679                                                                   
Epoch:  10%|######                                                      | 3/30 [08:52<1:19:54, 177.59s/it]    3      0.741179   0.753587   0.7507                                                                   
Epoch:  13%|########                                                    | 4/30 [11:36<1:15:24, 174.01s/it]    4      0.621733   0.833767   0.7193                                                                   
Epoch:  17%|##########                                                  | 5/30 [14:17<1:11:25, 171.44s/it]    5      0.544127   0.763003   0.7593                                                                   
Epoch:  20%|############                                                | 6/30 [16:58<1:07:54, 169.79s/it]    6      0.500345   0.760736   0.7552                                                                   
Epoch:  23%|##############                                              | 7/30 [19:41<1:04:41, 168.74s/it]    7      0.458589   0.537296   0.8162                                                                   
Epoch:  27%|################                                            | 8/30 [22:21<1:01:30, 167.73s/it]    8      0.426064   0.572941   0.8003                                                                   
Epoch:  30%|##################5                                           | 9/30 [25:03<58:28, 167.06s/it]    9      0.408095   0.703748   0.777                                                                    
Epoch:  33%|####################3                                        | 10/30 [27:45<55:31, 166.57s/it]    10     0.386634   0.740281   0.757                                                                    
Epoch:  37%|######################3                                      | 11/30 [30:26<52:35, 166.08s/it]    11     0.367969   0.540694   0.8254                                                                   
Epoch:  40%|########################4                                    | 12/30 [33:09<49:43, 165.75s/it]    12     0.340637   0.55378    0.8184                                                                   
Epoch:  43%|##########################4                                  | 13/30 [35:50<46:52, 165.44s/it]    13     0.327501   0.511305   0.8284                                                                   
Epoch:  47%|############################4                                | 14/30 [38:32<44:02, 165.16s/it]    14     0.307019   0.699895   0.7904                                                                   
Epoch:  50%|##############################5                              | 15/30 [41:11<41:11, 164.75s/it]    15     0.297837   0.659908   0.7998                                                                   
Epoch:  53%|################################5                            | 16/30 [43:49<38:20, 164.32s/it]    16     0.273685   0.395402   0.871                                                                    
Epoch:  57%|##################################5                          | 17/30 [46:28<35:32, 164.00s/it]    17     0.255043   0.679983   0.8083                                                                   
Epoch:  60%|####################################6                        | 18/30 [49:07<32:44, 163.75s/it]    18     0.239757   0.402062   0.8695                                                                   
Epoch:  63%|######################################6                      | 19/30 [51:46<29:58, 163.53s/it]    19     0.220999   0.404715   0.8703                                                                   
Epoch:  67%|########################################6                    | 20/30 [54:26<27:13, 163.34s/it]    20     0.191597   0.30171    0.9022                                                                   
Epoch:  70%|##########################################6                  | 21/30 [57:07<24:29, 163.23s/it]    21     0.172103   0.347882   0.892                                                                    
Epoch:  73%|############################################7                | 22/30 [59:45<21:43, 162.98s/it]    22     0.1352     0.325038   0.8984                                                                   
Epoch:  77%|#############################################2             | 23/30 [1:02:26<19:00, 162.89s/it]    23     0.09592    0.224998   0.9273                                                                   
Epoch:  80%|###############################################2           | 24/30 [1:05:07<16:16, 162.80s/it]    24     0.061392   0.212906   0.9383                                                                   
Epoch:  83%|#################################################1         | 25/30 [1:07:45<13:33, 162.60s/it]    25     0.044241   0.211943   0.9381                                                                   
Epoch:  87%|###################################################1       | 26/30 [1:10:23<10:49, 162.44s/it]    26     0.035331   0.217027   0.9378                                                                   
Epoch:  90%|#####################################################1     | 27/30 [1:13:04<08:07, 162.40s/it]    27     0.029121   0.214324   0.9391                                                                   
Epoch:  93%|#######################################################    | 28/30 [1:15:43<05:24, 162.26s/it]    28     0.024975   0.214972   0.9401                                                                   
Epoch:  97%|#########################################################  | 29/30 [1:18:21<02:42, 162.13s/it]    29     0.021141   0.215322   0.9395                                                                   
Epoch: 100%|###########################################################| 30/30 [1:21:03<00:00, 162.13s/it]
Finished!


```



