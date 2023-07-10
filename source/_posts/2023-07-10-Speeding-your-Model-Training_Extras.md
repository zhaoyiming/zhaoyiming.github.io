---
title: Speeding Your Model Training Extras
date: 2023-07-10 20:45:39
tags:
- NeuralNetwork
---

# INTRODUCTION

Although some methods of model training acceleration have been described, over the past year I have come across a few more techniques in model training that can improve training speed. In this article, we continue to add some tips to accelerate model training.

There are primarily three stages during model training that consume significant time:

- Loading data from disk
- Processing data in datasets and dataloaders
- Actual model training

# Data Loading

Primarily, to reduce time consumption, it is essential to minimize the first two stages without hardware improvements or changing the training strategy. We should aim to avoid time-consuming operations in the dataset or dataloader stage. The model training stage has to wait for the data to load before a batch of data can be processed. Hence, it's advisable to pre-process data offline using multiprocessing, followed by loading the data without any time-consuming processing.

## One method is to use datasets like HDF5

Even with offline pre-processing of data, loading large files still takes time. PyTorch datasets involve loading files from the disk, which inevitably takes some time. However, this issue can be mitigated by using the HDF5 dataset. HDF5 offers a way to store your pre-processed features in its data format. Here's a simple example of how to store your data into an HDF5 file using Python:

```python
import h5py
import numpy as np

with h5py.File("mytestfile.hdf5", "w") as f:
	dset = f.create_dataset("mydataset", (100,), dtype='i')
```

However, if you need to process hundreds of gigabytes of data, this method can be slow. In this case, multiprocessing can be used to split all data into several parts and convert them into multiple h5 files. Below is a code example that demonstrates how to split and merge data:

## Splitting File

```python
def split_scp(*from_path*, *target_path*, *file_num*):
  f = open(from_path, "r")
  lines = f.readlines()  
  n = math.ceil(len(lines)/file_num)
  output = [lines[i:i + n] for i in range(0, len(lines), n)]
  for small in range(len(output)):
      print("train"+*str*(small)+".scp: ",len(output[small]))
      with open(target_path + "/train"+*str*(small)+".scp", 'w') as f:
          for i in output[small]:
              f.write(i)
  print("-----split_scp down-----")
```

## Multiprocesss

```python
split_scp("scp/train_temp.scp", scp_path, process_num)

for i in range(*int*(process_num/one_process)):
    processes = []
    for i in range(i*one_process, (i+1)*one_process):
        p = Process(*target*=process_ming,
                    *args*=(scp_path+"/train" + *str*(i) + ".scp", chunk_path+"/train" + *str*(i) + ".h5" ,True))
        p.start()
        print(f'process '+ *str*(i) +' has started')
        processes.append(p)
    for p in processes:
        p.join()
```

## Cell_Process

```python
def process_ming(*scp*, *to_file*, *training*):
	Multiple proecess to parallel extract feature
```

## Merge

```python
def h5list2dict(*from_path*, *to_file*):
  if not os.path.exists(os.path.dirname(to_file)):
      os.mkdir(os.path.dirname(to_file))
  for root, dirs, files in os.walk(from_path):
      with h5py.File(os.path.join(root, files[0]), 'r') as f1:
          attributs = *list*(f1.keys())
          f1.close()
      for attribut in attributs:
          temp=[]
          for file in sorted(files):
              with h5py.File(os.path.join(root, file), 'r') as f2:
                  temp.extend(f2[attribut][...])
              f2.close()
          with h5py.File(to_file, "a") as fw:
              fw[attribut] = temp
          fw.close()
          del temp
          gc.collect()
  print("-----combining h5 files down-----")
```

## Virtual Dataset

Unfortunately, merging multiple h5 files can cause issues when memory size is scarce. In such cases, you can use h5py's virtual dataset to merge all files.

```python
def h5list2virtual(from_path, to_file, file_num):
  if not os.path.exists(os.path.dirname(to_file)):
      os.mkdir(os.path.dirname(to_file))
  total_num=0

  for root, dirs, files in os.walk(from_path):
      nowtype=h5py.File(root+"/"+files[0])['mix'].dtype
      print(nowtype)
      for i in range(len(files)):
          total_num+=int(files[i].split("_")[1].split(".")[0])
  layout1 = h5py.VirtualLayout(shape=(total_num, 4, 64000), dtype=nowtype)
  layout2 = h5py.VirtualLayout(shape=(total_num, 64000), dtype=nowtype)
  start_num=0

  for root, dirs, files in os.walk(from_path):
      for i in range(len(files)):
          now_num = int(files[i].split("_")[1].split(".")[0])
          now_file = root+"/"+files[i]
          print("file is: start_number is:",now_file, start_num)
          layout1[start_num:start_num+now_num, :,:] = h5py.VirtualSource(now_file, 'mix', shape=(now_num, 4,64000))
          layout2[start_num:start_num+now_num, :] = h5py.VirtualSource(now_file, 'zone0', shape=(now_num, 64000))
          start_num=start_num+now_num

  with h5py.File(to_file, 'w', libver='latest') as f:
      f.create_virtual_dataset('mix', layout1)
      f.create_virtual_dataset('zone0', layout2)
      print("-----combining h5 files down-----")
```

## Dataset

```python
class myDataset(*Dataset*):
  def __init__(*self*, *file*):
      *self*.fw=h5py.File(file, 'r')
  def __len__(*self*):
      return len(*self*.fw["mix"])
  def __getitem__(*self*, *idx*):
      return [torch.tensor(*self*.fw["mix"][idx], *dtype*=torch.float32), torch.tensor(*self*.fw["zone0"][idx], *dtype*=torch.float32)]
```

# A Proper DataLoader Worker When Data Processing is Necessary

If you must pre-process input data online, there are ways to alleviate loading congestion. Setting the correct parameters for "num_worker" and "batch_size" can allow you to load data using multiprocessing. Please note that setting a very high "num_worker" requires substantial memory. Generally, "num_worker" should be set to the number of GPUs times a certain factor, and increasing this number can allow more data to be loaded at a time. 

However, increasing this number too much can slow down the process. For example, as the number of "num_workers" increases, more data will be loaded from the dataloader at once. If a time-consuming operation on a file blocks the process, the entire operation is blocked. Therefore, setting the number to two or three times the number of GPUs may be a better choice. 

It's important to note that this number should be determined experimentally.

# Model Training

For the model training phase, just utilize PyTorch's official Distributed Data Parallel (DDP) method like the previous artcile.
