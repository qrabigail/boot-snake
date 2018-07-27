# BOOTS

BOOTS is a **B**asic **O**bject-**O**riented **T**raffic **S**imulator which empirically compares the performance of defense algorithms in trustless distributed networks.  It uses bitcoin data to simulate users joining and leaving the network. Using this data as a base, it can simulate sybil attacks from an adversary, and implement a defense algorithm to prevent the sybils from overtaking the network. The data from the simulation can be used to evaluate the efficiency of defense algorithms, and compare the costs incurred to the ‘good’ users and the adversary.

## Usage
The program is packaged into an executable for convenience, but also includes the source code so it can be modified or extended. Either way, you must have the pickle and data files in the same directory as the code. Unzip `data.zip` for the full dataset, or `test_data.zip` for a small sample of the data (useful for debugging).

### Quick Start
Start BOOTS by running the executable. Use the buttons to modify the parameters of the attack, the algorithms to use, etc and start. When the simulation is complete, the results will be displayed as a chart. This data can be exported to a CSV file.

### From Source
BOOTS is written in python 3.6 using the scipy, numpy, matplotlib, and tkinter libraries. With these libraries installed, it can be started from the console with `python main.py`. See the design section for more information about the objects and classes.


## Design & Documentation
The graphical user interface creates and runs a simulation based on the user’s variables, and charts the resulting data.

A simulation is composed of four objects: event data, a network, an attack, and a defense algorithm. These four objects have little to no knowledge of the other components and overall system. The simulation manipulates these objects while running, and records the data generated.


### Event Data
The event data object is mainly a wrapper around a large array that contains the arrival and departure times of all good ids. This array is stored as a memory map so it doesn’t have to be loaded into working memory all at once. The object has methods used to add events to the memory map, which are used to generate the object, and methods that calculate the total and net changes at a timestamp which are used to run the simulation.

The included data file was generated with converter.py from a MATLAB array of scrubbed bitcoin network activity. The converter creates a new event data object, then adds an arrival and departure event for each entry in the MATLAB file. The resulting object is saved as a .pickle file, and the join/leave array is saved as a .dat file. A new event data object needs to be created only once, because the simulator will open these saved files to access the object. 

### Network
The network object is initialized with one optional variable, the initial size of the network. This object simply keeps track of the number and ids of good users in the network, as well as the number of sybil users in the network. At each time step, the simulation adds and removes users according to the event data, and the defense algorithm uses the current state of the network to evaluate the costs incurred.

### Attack
The attack object is initialized with values for alpha (computing power of the adversary divided by computing power of ‘good’ users), attack size (how many sybils to create), and a start and end fraction of time when the attacker will add sybils. The simulation sets the attack interval based on the length of time in the event data. Then, at every time step in the simulation, the attack calculates how many sybils should be created (based on the current timestamp) and the simulation adds them to the network.

### Extending
New attacks can extend the generic attack object and just override the sybils_added_at function. This function simply takes in a timestamp and returns how many sybils to add at that time.

### Defense Algorithm
The defense object is initialized with no variables. The simulation sets its variables with the event data and attack when it begins, because they depend on the attack and data. At each time step, the simulation calls the defense algorithm to calculate the entry costs for the ids (good and sybil) that are joining the network, and evaluate the state of the network. When the algorithm evaluates the network, it may determine that a purge puzzle is necessary. When this happens, it will modify the network to reduce the number of sybil ids present and calculate the costs to users in the network. The simulation saves this data so it can be charted and exported at the end of the simulation. 

### Extending
New algorithms can be created by extending the generic defense_algorithm object. A new algorithm should override: 
set_vars, to save any relevant information about the network and attack
evaluate, which will purge the network if necessary and return the costs to users
get_entry_costs, which returns the total costs for n arrivals at a single time step


