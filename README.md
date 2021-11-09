## Double Dock For Chemprop Model-Training

 **Introduction：**

* The main script calls the vinalc and rdock programs to perform docking  computation of a given compound library respectively. 

* After DOCK_1 is over, take top_20% compounds according to the docking score to do DOCK_2 docking; 
* After DOCK_2 is over, it is ready for model training and prediction; 



## Requirements

vinalc;

rdock2013.1;

chemprop;

Open Babel 3.0.0;

cuda >= 8.0;

cuDNN;



### Installation

---

### Option 1: Conda

The easiest way to install the `chemprop` dependencies is via conda. Here are the steps:

1. `cd /path/to/chemprop`
2. `conda env create -f environment.yml`
3. `conda activate chemprop` (or `source activate chemprop` for older versions of conda)

Note that on machines with GPUs, you may need to manually install a GPU-enabled version of PyTorch by following the instructions [here](https://pytorch.org/get-started/locally/).

### Option 2: Docker

Docker provides a nice way to isolate the `chemprop` code and environment. To install and run our code in a Docker container, follow these steps:

1. `cd /path/to/chemprop`
2. `docker build -t chemprop .`
3. `docker run -it chemprop:latest /bin/bash`

Note that you will need to run the latter command with nvidia-docker if you are on a GPU machine in order to be able to access the GPUs.





## File Naming Conventions

* **ReIndex of Compounds**

|  Category  |                Format                |
| :--------: | :----------------------------------: |
|   active   |   active0001(active + four digits)   |
|   decoys   |  decoys0000001(decoy +seven digits)  |
|  chemDiv   | ligand_0000001(ligand_+seven digits) |
| smiles.csv |        [ UniqueID , smiles ]         |



* **Files for Docking**

  |      File Name      |             Path              |                         Description                         |
  | :-----------------: | :---------------------------: | :---------------------------------------------------------: |
  |    protein.pdbqt    |           ../temp/            |                 receptor for vinalc docking                 |
  |    protein.mol2     |           ../temp/            |                 receptor for rdock docking                  |
  |     ligand.mol2     |           ../temp/            | original ligand，used to define the pocket center for rdock |
  | compounds_all.pdbqt |           ../temp/            |    compounds in pdbqt format( Includeing  active/decoys)    |
  |                     |    ../single_mol2_chemDiv/    |                 mol2 format single molecule                 |
  |                     | ../active_decoys_single_mol2/ |        mol2 format single molecule（actives/decoys）        |
  |     smiles_all      |           ../temp/            |  all compounds in smiles format(Includeing actives/decoys)  |
  |     geoList.txt     |           ../temp/            |           Pocket center and dimension for vinalc            |
  |     recList.txt     |           ../temp/            |               Receptor file path holding file               |
  |     ligList.txt     |           ../temp/            |            small molecule file path holding file            |
  |     protein.prm     |           ../temp/            |                configuration file for rdock                 |



* ****

|          FileName          |                       Description                       |
| :------------------------: | :-----------------------------------------------------: |
|       key_value.txt        |           score of first pose (vinalc result)           |
|      vinalc_top20.txt      |               top_20% vinalc Compound ID                |
|     vinalc_top_20.mol2     |         top_20% vinalc Compound in mol2 format          |
|     compounds_rdock.sd     | top_20% vinalc Compound for rdock docking in sdf format |
| model/ligand_0000001.pdbqt |          split vinalc result into single files          |



* Running

1. Run all：

```
[ shell temp]$    ./vinalc_dock.sh
```

2. Run step by step：

```
#step1:vinalc docking
[shell temp]$	 sbatch sub-vinalc-cnlong-openmpi.sbatch   

#step2:vinalc result collection
[shell temp]$	 python sort_for_vinalc_pdbqt.py

#step3:rdock docking
[shell temp]$	 sbatch sub-cnnl.sbatch				

#step4:rdock result collection
[shell temp]$	 python get_label.py
```

3. Result Files（12 files， ~30MB）

```
double_dock_model_1_train.csv				double_dock_model_1_active_decoys.csv
double_dock_model_2_train.csv				double_dock_model_3_active_decoys.csv
double_dock_model_2_train.csv				double_dock_model_3_active_decoys.csv

vinalc_model_1_train.csv					vinalc_model_1_active_decoys.csv
vinalc_model_2_train.csv					vinalc_model_2_active_decoys.csv
vinalc_model_3_train.csv					vinalc_model_3_active_decoys.csv
```

4. clean files

```
# Clean all
[shell temp]$	 ./clean_all.sh

# Clean Intermediate files
[shell temp]$	 ./clean_partial.sh


```



## Summary of Scripts

---

|   confile   |      inputfile      |              script              |         work          | outputfile |
| :---------: | :-----------------: | :------------------------------: | :-------------------: | ---------- |
| ligList.txt | compounds_all.pdbqt | sub-vinalc-cnlong-openmpi.sbatch |       rdock.sh        | ——         |
| protein.prm |     geoList.txt     |         sub-cnnl.sbatch          | rdock_process.sbatch  | ——         |
| recList.txt |     ligand.mol2     |     sort_for_vinalc_pdbqt.py     |    vinalc_dock.sh     | ——         |
|             |    protein.mol2     |         rdock_future.py          | vinalc_process.sbatch | ——         |
|             |    protein.pdbqt    |           get_label.py           |     clean_all.sh      | ——         |
|             |   smiles_all.csv    |                                  |                       | ——         |


