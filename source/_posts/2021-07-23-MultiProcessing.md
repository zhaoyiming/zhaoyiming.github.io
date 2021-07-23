---
title: MultiProcessing of DL in PyTorch  
date: 2021-07-23 15:13:20
tags:
- NeuralNetwork
categories: 
- NeuralNetwork
---
# MultiProcessing of DL in PyTorch  

```
This article helps to handle DL tasks(e.g., extract and save embedding) base on multiprocess and multi-GPU in PyTorch. 

Thought：Difference to the ddp。
```

### Include libs

```
import GPUtil
from torch.multiprocessing import Process
```

### Initial Parameters

```
use_gpu=True // use GPU or not
world_size=1 // the number of allocated GPU and Process
```

### Structured Functions

```
def extract(self):
        if self.parallel_extract_list:
            self.parallel_extract()
        if self.sequential_extract_list:
            self.sequential_extract()
```

### parallel_extract()

```
def parallel_extract(self):
        t0 = time.time()
        for extract_file, save_to_dir in self.parallel_extract_list:

       		# spilt the huge file to several parts according to world_size
            df = pd.read_csv(extract_file)
            dfs = np.array_split(df, self.world_size)
            
            # get max gpu nums
            gpu_ids = GPUtil.getAvailable(maxMemory=0.02,
                                          limit=self.world_size)
            processes = []
            for rank, gpu_id in enumerate(gpu_ids):
                p = Process(target=self._parallel_extract,
                            args=(f'{save_to_dir}/res_{rank}.h5', dfs[rank], gpu_id, rank))
                p.start()
                print(f'process {rank} has started')
                processes.append(p)

            for p in processes:
                p.join()
        print(f'total time is {(time.time() - t0) / 60}')

        
    def _parallel_extract(self, save_to_dir, csv_file, gpu_id, rank):
    
    	# allocate gpu for every child_process
        self.device = torch.device(gpu_id)
        
  		# load model
        self.params["embedding_model"].to(self.device)
    
        # prep dataloader
        test_dataloader = self.dataio_prep(csv_file)
        
        # In general, _extraction extracts embeeding and return specific column.
        with h5py.File(save_to_dir, self.mode) as fw:
            fw['X'], fw['n_frames'], fw['spk_ids'], fw['spk_path'] = self._extraction(save_to_dir, test_dataloader)
        print(f'saving embedding')
```

### sequential_extract()

```
 def sequential_extract(self, gpu_id=None):
 /*allocate device according to parameters*/
        if not gpu_id:
            if self.use_gpu:
                gpu_id = GPUtil.getAvailable(maxMemory=0.02,
                                             order='last',
                                             limit=self.world_size)[0]
                self.device = torch.device(gpu_id)
            else:
                self.device = torch.device('cpu')
        else:
            self.device = torch.device(gpu_id)

        self._sequential_extract()
   def _sequential_extract(self):
        t0 = time.time()
        for csv_file, save_to_dir in self.sequential_extract_list:
            # prep dataloader
            test_dataloader = self.dataio_prep(csv_file)

            # load model and allocate device
            self.params["embedding_model"].to(self.device)
            
            # In general, _extraction extracts embeeding and return specific column.
            with h5py.File(save_to_dir, self.mode) as fw:
                fw['X'], fw['n_frames'], fw['spk_ids'], fw['spk_path'] = self._extraction(save_to_dir, test_dataloader)
            print(f'saving embedding')
            print(f'save_to_dir to {save_to_dir}')
        print(f'total time is {(time.time() - t0) / 60}')
```



