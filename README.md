# MAPFAST

This repository contains the official implementation for our paper "MAPFAST: A Deep Algorithm Selector for Multi Agent Path Finding using Shortest Path Embeddings"

## After cloning the repo
```
cd MAPFAST
pip3 install -r requirements.txt
```
This will install all the packages necessary for training/testing the model.

## Dataset

Following items are needed for training the model:
```
1. yaml_details -> Json file which contains file name as key and another json object (which has 'SOLVER' storing the name of the fastest solver and solving time (in seconds) for all the solvers) as value.
2. agent_details -> Json file which contains file name as key and its value is a json object containing the start and goal locations of the agents.
3. map_details -> Json file which contains file name as key and another json object (which has number of agents, dimensions of the input map) as value.
4. .png or .npz files -> Images or compressed numpy representations of the map which will be given as input to CNN. Note that the names of these images/numpy files should match the name in yaml_details/agent_details/map_details.
```
We used [MAPF benchmark](https://movingai.com/benchmarks/mapf.html) to generate the MAPF instances for training and testing. 

Each entry of yaml_details will look as shown below. new_brc202d-even-4_1_40_agents.yaml is the file name and the value is a json. yaml_details\[filename\]\["SOLVER"\] gives the name of the best solver for the given file name. yaml_details\[filename\]\[solver_name\] gives the time which the solver took to solver the instance if it was successful else -1. In our dataset, the possible solvers are BCP, CBSH, CBS, SAT.
```
"new_brc202d-even-4_1_40_agents.yaml": {"SOLVER": "BCP", "BCP": 34.2429, "CBS": -1, "CBSH": 37.1887, "SAT": -1}
```

Each entry of map_details will look as shown below. new_empty-8-8-random-17_2_20_agents.yaml is the file name and the value is a json. map_details\[filename\]\["no_agents"\] will give the number of agents in the input instance, map_details\[filename\]\["mp_dim"\] will give a list where the first value corresponds to the height, second value corresponds to the width of the input map and map_details\[filename\]\["no_obs"\] will give the number of obstacle in the input instance.
```
"new_empty-8-8-random-17_2_20_agents.yaml": {"no_agents": 20, "mp_dim": [8, 8], "no_obs": 0}
```

Each entry of agent_details will look as shown below. maze-32-32-2-even-8_5_agents.yaml is the file name and the value is a json. agent_details\[filename\]\["starts"\] and agent_details\[filename\]\["goals"\] will give a list corresponding to the start and goal location of each of the agents respectively. For example, in the json below, [23, 30] is the start location of agent 3 and [20, 29] is the goal location of agent 3.
```
"maze-32-32-2-even-8_5_agents.yaml": {"starts": [[11, 15], [22, 24], [23, 30], [2, 5], [12, 10]], "goals": [[10, 13], [23, 24], [20, 29], [1, 4], [14, 8]]}
```


## MAPFAST

`MAPFAST.py` contains the necessary class for training and testing the model as described in our paper.

### Training model

Following shows a way to train a model with the default setting given the necessary json files.

```
mapfast = MAPFAST(device, yaml_details, agent_details, map_details, input_location, mapping)

train_list, test_list, valid_list = mapfast.get_train_valid_test_list()

model = mapfast.train_model(train_list, valid_list, model_loc, model_name)
```

This will train the default MAPFAST model which has 14 output neurons (4 for best solver classification, 4 for finish prediction and 6 for pairwise comparison).
Here, device is the device type in which we want to train.

### Testing model

Following shows a way to test a model with the default setting given the necessary json files and model location details.

```
mapfast = MAPFAST(device, yaml_details, agent_details, map_details, input_location, mapping)

train_list, test_list, valid_list = mapfast.get_train_valid_test_list()

prediction_data = mapfast.test_model(test_list, model_loc, model_name)
```

This will test the default MAPFAST model at given location and given name and return a json object with the test details. A sample entry in this json is shown below. new_ht_mansion_n-even-22_2_40_agents.yaml_0 is the filename. `_0` in the end denotes that no augmentation was performed in the input. "best" gives a list where first entry is the predicted best solver and the second entry is ground truth. "BCP", "CBSH", "CBS", "SAT" each gives a list where first entry tells if the solver was successful in solving the instance as predicted by the model and the second entry is ground truth. "0" to "5" each corresponds to output of the comparison neurons.
```
"new_ht_mansion_n-even-22_2_40_agents.yaml_0": {"best": [0, 0], "BCP": [1, 1], "CBS": [1, 1], "CBSH": [0, 0], "SAT": [0, 0], "0": 1, "1": 1, "2": 1, "3": 0, "4": 1, "5": 1}
```

### Note
When creating the `MAPFAST` class, we have to give a json object called mapping which maps the solver name to a unique number.
Example mapping is shown below.
```
{"BCP":0, "CBS":1, "CBSH":2, "SAT":3}
```

## New Dataset

To train the model with new dataset and potentially with different solvers, the necessary `yaml_details`, `agent_details`, `map_details` json files and the input  (image/numpy) has to be generated accordingly. The mapping json has to be changed to reflect the solvers in the portfolio. No other changes inside the code is necessary to train with a new dataset.

## Misc Info

1. `main.py` provides a sample implementation where all the parameters are fetched from a `config.json` file. The location of `config.json` can also be passed as a command line argument to the python script. The default location is json_files folder.

2. `analysis.py` prints a detailed report which includes the accuracy, coverage and custom score details as described in our paper.

3. Use dir(MAPFAST) or help(MAPFAST) to get information and usage details for each function in `MAPFAST` and `InceptionClassificationNet` class.
