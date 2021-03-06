This file describes the command line interface of the alib project by outlining the steps of the intended workflow.

1. `deploy`
Deploy the codebase (i.e. all *.py files, the topology zoo and the scenario yaml file) to the servers. 
Arguments:

        deploy
            --codebase_id=mcf_baseline_2016_08_24
            <!--(--local_base_dir=../../)                    optionally-->
            --remote_base_dir=/research/mcf_baseline/
            --servers=[loadgen1, loadgen2, loadgen3]
            --extra=scenariogeneration.yml
            
        the command will create the following folders in the remote_base_dir
            /research/mcf_baseline/mcf_baseline_2016_08_24/input
                                                          /src
                                                          /output
                                                          /log
                
2. `generate-scenarios` 
Generate the scenario container pickle. This should run on the server. It needs a scenariogeneration.yml containing the full specification of the scenario parameters and a file to which the pickle of the scenario container will be save. 
Arguments and the amount of thread for multiprocessing:
    
        generate_scenarios
             --scenario_file=/research/mcf_baseline/mcf_baseline_scenarios.pickle
             --parameters=scenariogeneration.yml
            <!--(--threads=1)                    optionally-->
             
    The scenariogeneration.yml is structured in the following way:

      - At the highest level, the yaml is an outline of the tasks which need to be completed during scenariogeneration.
      - For each task, a list of strategies is specified. Each strategy is identified by unique name assigned by the user.
      - For each strategy, the name of the class implementing the strategy is provided, along with the associated parameter space.
  
    From this information, a list of scenario parameters is generated by determining each possible combination of strategies and each combination of parameters.
             
    Example of a `scenariogeneration.yml` file:

      

        request_generator:
            - short_chains:  # some arbitrary identifier for the request generation scheme
                ServiceChainGenerator:
                     number_of_requests: [20]
                     min_number_of_nodes: [1]
                     max_number_of_nodes: [3]
                     probability: [0.5]
                     latency_factor: [1.0, 1.5]
                     node_resource_factor: [1.25, 1.5]
                     edge_resource_factor: [1.0]
            - long_chains:
                ServiceChainGenerator:
                     number_of_requests: [50]
                     min_number_of_nodes: [8]
                     max_number_of_nodes: [10]
                     probability: [0.2]
                     latency_factor: [1.0, 1.5]
                     node_resource_factor: [1.25]
                     edge_resource_factor: [1.0]

        profit_calculator:
            - random:
                RandomEmbeddingProfitCalculator:
                     profit_factor: [1.0]
                     iterations: [200]
            - optimal:
                OptimalEmbeddingProfitCalculator:
                     profit_factor: [1.0]
                     iterations: [200]

        node_placement:
            - neighbors:
                NeighborhoodSearchRestrictionGenerator:
                    potential_nodes_factor: [0.3]

        substrate_transformation:
            - uniform:
                SubstrateTransformation_UniformTypeDistribution:
                    node_types: [[t1, t2, t3]]   # TODO: Convert values to a tuple internally
                    node_capacity: [100.0]
                    node_cost: [1.0]
                    edge_capacity: [100.0]
                    node_type_distribution: [0.3]

        substrate_reader:
            - substrates:
               TopologyZooReader:
                  topology: [Bellcanada, Chinanet, Geant2012, Uunet]




3. `Download`
Download the pickle from the server (via ssh scp) and use the deploy from the cli additionally the pickle as extra 
( Formerly a command of the CLI )

5. `start-experiment`
Start experiment:
 Generate the scripts that should be run on each server. In addition to the scenario pickle distributed in the previous step, all algorithms that should be run on these scenarios need to be specified
Arguments:

        --experiment_yaml=experiment.yml
        --server=localhost
        --server_index=1
        --number_of_server=1

    example of a `experiment.yml` file:
        
        RUN_PARAMETERS:
            
            - ALGORITHM:
                ID: Classic_MCF
                ALGORITHM_PARAMETERS:
                    # Gurobi Parameters:
                    timelimit: [100, 600, 1800]
                    threads: [8]
            - ALGORITHM:
                ID: Decomposition
                ALGORITHM_PARAMETERS:
                    # Gurobi Parameters:
                    timelimit: [100, 600, 1800]
                    threads: [8]
            # ... allow specifying multiple algorithm & algorithm parameters
            # allow iterating through combinations of parameters (same as parameter space from scenariogeneration):
            #     Map algorithmparameters to an index
        
           # it should result in something like
           # [{"ID": "Classic_MCF, "Parameters": {"timelimit": [100, 600, 1800], "threads": [8]}},
           #  {"ID": "Decomposition, "Parameters": {"timelimit": [100, 600, 1800], "threads": [8]}}]

6. `Execute Experiments` :manually




