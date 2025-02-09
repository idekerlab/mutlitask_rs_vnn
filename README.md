# Multitask-VNN: a visible neural network multi-task learning model for drug response prediction
Multitask-VNN is an interpretable neural network-based model that predicts
cell response to a set of drugs with similar function.
This framework integrates information across multiple levels of 
cancer cell biology to understand drug response, and can serve 
to identify and explain biomarkers for clinical application.

Multitask-VNN characterizes each cell line using its genotype;
the feature vector for each cell is a binary vector representing
mutational status and copy number variations of the genes 
used in clinical panels like Foundation Medicine (n=718).

Related publication:
Zhao, Singhal, et al. Cancer Mutations Converge on a Collection of Protein Assemblies to Predict Resistance to Replication Stress. Cancer Discov 1 March 2024; 14 (3): 508–523. https://doi.org/10.1158/2159-8290.CD-23-0641

# Environment set up for training and testing
The model training/testing scripts require the following environmental setup:

* Hardware required for training a new model
    * GPU server with CUDA>=11 installed

* Software
    * Python >=3.6
    * Anaconda
        * Relevant information for installing Anaconda can be found in: 
        https://docs.conda.io/projects/conda/en/latest/user-guide/install/.
    * PyTorch
        * The current release of this model was trained/tested using PyTorch 1.8.0
        * Depending on the specification of your machine, run appropriate command to install PyTorch.
        The installation command line can be found in https://pytorch.org/.
        * For a **LINUX-based GPU server** with **CUDA version 11.1**, run the following command line:
        ```angular2
        conda install pytorch torchvision cudatoolkit=11.1 -c pytorch
        ```

* Set up a virtual environment
    * If you are training a new model or test the pre-trained model using a GPU server,
     run the following command line:
    to set up a virtual environment (cuda11_env).
        ```angular2
         conda env create -f cuda11_env.yml
        ```

Required input files:
1. Cell feature files:
    * _gene2ind.txt_: make sure you are using _gene2ind.txt_ file provided in this repository.
    * _cell2ind.txt_: a tab-delimited file where the 1st column is index of cells 
        and the 2nd column is the name of cells (genotypes).
    * _cell2mutation.txt_: a comma-delimited file where each row has 718 binary values
         indicating each gene is mutated (1) or not (0).
    The column index of each gene should match with those in _gene2ind.txt_ file. 
    The line number should match with the indices of cells in _cell2ind.txt_ file.
    * _cell2cndeletion.txt_: a comma-delimited file where each row has 718 binary values
         indicating copy number deletion (1) (0 for not).
    * _cell2amplification.txt_: a comma-delimited file where each row has 718 binary values
         indicating copy number amplification (1) (0 for not).

2. Test data file: _test_data.txt_
    * A tab-delimited file containing all data points that you want to estimate drug response for.
    The 1st column contains cell IDs and rest of the columns contain drug response AUC.

To load a pre-trained model used for analyses in our manuscript 
and make prediction for the cell lines of your interest, 
execute the following:

1. Make sure you have _gene2ind.txt_, _cell2ind.txt_, _cell2mutation.txt_, _cell2cndeletion.txt_,
_cell2amplification.txt_, and your file containing test data in proper format (examples are provided in
_sample_ folder)

2. To run the model in a GPU server,  execute the following:
    ```
    python predict.py   -gene2id gene2ind.txt
                        -cell2id cell2ind.txt
                        -genotype cell2mutation.txt
                        -cn_deletions cell2cndeletion.txt
                        -cn_amplifications cell2amplification.txt
                        -predict test_data.txt
                        -hidden <path_to_directory_to_store_hidden_values>
                        -result <path_to_directory_to_store_prediction_results>
                        -load <path_to_model_file>
                        -cuda <GPU_unit_to_use>
                        -batchsize 2000 (or any other value)
    ```
    * An example bash script (_test.sh_) is provided in _sample_ folder.


# Train a new model
To train a new model using a custom data set, first make sure that you have
a proper virtual environment set up. Also make sure that you have all the required files
to run the training scripts:

1. Cell feature files:
    * _gene2ind.txt_: make sure you are using _gene2ind.txt_ file provided in this repository.
    * _cell2ind.txt_: a tab-delimited file where the 1st column is index of cells 
        and the 2nd column is the name of cells (genotypes).
    * _cell2mutation.txt_: a comma-delimited file where each row has 718 binary values
         indicating each gene is mutated (1) or not (0).
    The column index of each gene should match with those in _gene2ind.txt_ file. 
    The line number should match with the indices of cells in _cell2ind.txt_ file.
    * _cell2cndeletion.txt_: a comma-delimited file where each row has 718 binary values
         indicating copy number deletion (1) (0 for not).
    * _cell2amplification.txt_: a comma-delimited file where each row has 718 binary values
         indicating copy number amplification (1) (0 for not).

2. Training data file: _train_data.txt_
    * A tab-delimited file containing all data points that you want to use to train the model.
    The 1st column contains cell IDs and rest of the columns contain drug response AUC.
    * To help create train data from a pan-drug file, a python script "create_train_data.py"
    can be executed. To execute it, please refer to "scripts/create_input.sh" bash script.

3. Ontology (hierarchy) file: _ontology.txt_
    * A tab-delimited file that contains the ontology (hierarchy) that defines the structure of a branch
    of a VNN that encodes the genotypes. The first column is always a term (assembly),
    and the second column is a term or a gene.
    The third column should be set to "default" when the line represents a link between terms,
    "gene" when the line represents an annotation link between a term and a gene.
    The following is an example describing a sample hierarchy.

        ![](https://github.com/idekerlab/mutlitask_vnn/blob/main/sample/ontology_image_sample.png)

    ```
     GO:0045834	GO:0045923	default
     GO:0045834	GO:0043552	default
     GO:0045923	AKT2	gene
     GO:0045923	IL1B	gene
     GO:0043552	PIK3R4	gene
     GO:0043552	SRC	gene
     GO:0043552	FLT1	gene       
    ```

     * Example of the file (_ontology.txt_) is provided in _sample_ folder.


There are several optional parameters that you can provide in addition to the input files:

1. _-modeldir_: a name of directory where you want to store the trained models. The default
is set to "MODEL" in the current working directory.

2. _-genotype_hiddens_: a number of neurons to assign each subsystem in the hierarchy.
The default is set to 12.

3. _-epoch_: the number of epoch to run during the training phase. The default is set to 300.

4. _-batchsize_: the size of each batch to process at a time. The deafult is set to 5000.
You may increase this number to speed up the training process within the memory capacity
of your GPU server.

5. _lr_: Learning rate. Default is set 0.001.

6. _wd_: Weight decay. Default is set 0.001.

7. _-cuda_: the ID of GPU unit that you want to use for the model training. The default setting
is to use GPU 0.

* All the parameters are mentioned in the src/train_helper.py file.

Finally, to train a multitask-VNN, execute a command line similar to the example provided in
_sample/train.sh_:

```
python -u train.py  -onto ontology.txt
                    -gene2id gene2ind.txt
                    -cell2id cell2ind.txt
                    -genotype cell2mutation.txt
                    -cn_deletions cell2cndeletion.txt
                    -cn_amplifications cell2amplification.txt
                    -tasks task_list_RS.txt
                    -train train_data.txt
                    -modeldir sample/model
                    -genotype_hiddens 12
                    -epoch 100
                    -batchsize 64
                    -cuda 0
                    -lr 0.0005
                    -wd 0.0005
```
