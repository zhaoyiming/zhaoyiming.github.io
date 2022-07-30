---
title: Speeding Your Model Training
date: 2022-07-30 00:41:39
tags:
- NeuralNetwork
---

Time is very precious for deep learning engineers. A typical time cost for model training could be divided into three parts:

- **Data Loading**
- **Forward Propagation**
- **Backward Propagation**

It is essential to reduce the time cost of data reading when GPU resource is enough and fixed. The data loading time could be reduced to less than 0.001s for a batch(8 samples), and the latter two procedures could cost 0.16s and 0.23s. However, any extra operation for training data(i.e., chunk data) would directly bring more time costs for every batch. To reduce the whole training time cost, you cloud to improve the training process from the following two aspects. 

## Save Data to HDF5 

To alleviate the real-time communication expense between memory and disks, saving your fragment files to a whole file is better to reduce the retrieval overhead. HDF5 is an ideal saving file format for massive data. Following are some detailed procedures in saving data to an HDF5:

### Speeding data saving by multiprocessing

#### Obtaining data list

```python
def generate_list(from_path, target_path):
    """generate train data lists from data path.

    Args:
        from_path (string): data path
        target_path (sring): list path
    """

    files=os.listdir(from_path)
    files_list=[]
    for i in files:
        files_list.append(i)
    
    train_list=files_list[:70000]
    dev_list=files_list[70000:71000]

    with open(target_path+"train_temp.scp", "w") as f:
        for i in train_list:
            f.write(i+" "+from_path+"/"+i+"\n")
    with open(target_path+"dev_temp.scp", "w") as f:
        for i in dev_list:
            f.write(i+" "+from_path+"/"+i+"\n")
    print("-----generate_list down-----")
```

#### Splitting the training data list to several lists

```python
def split_scp(from_path, target_path, file_num):
    """Splitting big trainging list to multiple file to parallel extract features 

    Args:
        from_path (string): to be splited file path
        target_path (string): chunk files path
        file_num (int): number to be splited
    """

    f = open(from_path, "r")
    lines = f.readlines()  
    n = math.ceil(len(lines)/file_num)
    output = [lines[i:i + n] for i in range(0, len(lines), n)]

    for small in range(len(output)):
        print("train"+str(small)+".scp: ",len(output[small]))
        with open(target_path + "train"+str(small)+".scp", 'w') as f:
            for i in output[small]:
                f.write(i)
    print("-----split_scp down-----")
```

#### Saving your data to HDF5

Read data from your data list in the following code:

```python
def data_load():
       data_dict={i:[] for i in ["key1", "key2"]}
       for data in datalist:
           eg=data_read(data) # for read data
           chunks = self.splitter.split(eg) # for chunk 
           for i in chunks:
               for cell in ["key1", "key2"]:
               data_dict[cell].append(i[cell])

        batch = {
                key: np.stack(data_dict[key], axis=0).astype(np.float32)
                for key in ["key1", "key2"]
        }
       
        with h5py.File(to_file, 'a') as fw:
            fw["key1"], fw["key2"] = batch["key1"], batch["key2"]
            print(f'saving ' + str(len(data_dict["key1"])) + ' data to '+to_file)
        fw.close()
```

#### Multiprocessing

Utilizing the multiprocessing method in python to extract data to multiple hdf5 files:

~~~python
def process_ming(scp, to_file, training):
    """Multiple proecess to parallel extract feature

    Args:
        scp (string): child process file
        training (bool): is training
    """
    dataload(xxx, xxx)

from multiprocessing import Process
process_num=10 # processes you want to paralell run
processes = []
for i in range(process_num):
  p = Process(target=process_ming,
              args=(xxx,xxx))
  p.start()
  print(f'process '+ str(i) +' has started')
  processes.append(p)

for p in processes:
    p.join()
~~~

#### Merge hdf5 files to one

```python
def h52binh5(from_path, to_file):
    """ Combining multiple small h5 files to a whole h5 file

    Args:
        path (string): path that store all small h5 files
    """
    if not os.path.exists(os.path.dirname(to_file)):
        os.mkdir(os.path.dirname(to_file))

    for root, dirs, files in os.walk(from_path):
        with h5py.File(os.path.join(root, files[0]), 'r') as f1:
            attributs = list(f1.keys())
        f1.close()

        data = {attribut: [] for attribut in attributs}
        for attribut in attributs:
            for file in sorted(files):
                with h5py.File(os.path.join(root, file), 'r') as f2:
                    data[attribut].append(f2[attribut][...])
                f2.close()
            temp=np.concatenate(data[attribut], axis=0)
            print(attribut+" : "+ str(temp.shape[0]))
            with h5py.File(to_file, "a") as fw:
                fw[attribut] = temp
            fw.close()
            del temp
    print("-----combining h5 files down-----")
```

### Customize your dataset for reading HDF5 

Customize your PyTorch dataset to extract data from HDF5:

```python
class myDataset(Dataset):
    def __init__(self, file):
        self.fw=h5py.File(file, 'r')

    def __len__(self):
        return len(self.fw["key1"])

    def __getitem__(self, idx):
        return torch.tensor(self.fw["key1"][idx], dtype=torch.float32), torch.tensor(self.fw["key2"][idx], dtype=torch.float32)

```

### Ultizing MultiPorcess to Improve Costing Operation

Avoid processing or chunking your data during the training procedure. Operation for training data would bring unmeaningful time cost, and this costing operation is challenging to parallel. Therefore, moving these operations to the data preparation stage could reduce training time. 

## DP & DDP

### Data Parallel

Speeding model training by DP as following:

```python
model = torch.nn.DataParallel(model) 
```

What's more? Put some data operation to be speeded by Cuda on your first GPU.

### Distributed Data Parallel

Speeding model training by DDP as following:

~~~python
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--local_rank",default=-1, type=int)
args = parser.parse_args()

torch.cuda.set_device(args.local_rank)
torch.distributed.init_process_group(backend='nccl')
nnet=model()

train_dataset=myDataset(args.train_data)
dev_dataset=myDataset(args.dev_data)

train_sampler = torch.utils.data.distributed.DistributedSampler(train_dataset, rank=args.local_rank, shuffle=True)
dev_sampler =torch.utils.data.distributed.DistributedSampler(dev_dataset, rank=args.local_rank, shuffle=False) 

train_loader = DataLoader(
    dataset=train_dataset,
    sampler=train_sampler,
    batch_size=args.batch_size,
    num_workers=16,
    pin_memory=True,
    drop_last=False
)

class Trainer(object):
    def __init__(self,
                 nnet,
                 rank,
                 ):
        self.rank=rank,
        self.device = th.device("cuda", self.rank[0])
        self.nnet=DDP(nnet.to(self.device), device_ids=[self.rank], output_device=self.rank)
    
def save_checkpoint(self, best=True):
        cpt = {
            "epoch": self.cur_epoch,
            "model_state_dict": self.nnet.module.state_dict(), # add .module for ddp
            "optim_state_dict": self.optimizer.state_dict()
        }
        th.save(
            cpt,
            os.path.join(self.checkpoint,"{0}.pt.tar".format("best" if best else "last")))


~~~

