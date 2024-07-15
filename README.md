# Graph Analytics Benchmark

This repository contains all the required instructions to execute the graph analytics benchmark
for ArangoDB in comparison to Neo4j.

## Important Source File Locations

Please use this code based on this PR here: 
- https://github.com/arangodb/gral/pull/69 (or branch: feature/add-neo-benchmarks)

### Building Gral/Graph Analytics Engine (GAE) binary
You need to have rust on your machine. Then execute:
```
cargo build --release
```

### Starting Neo4j and ArangoDB
Please use the docker compose file to start both databases: 

```
docker compose up -d
```

You need docker for this. If you do not want to use the docker compose file, read the `docker-compose.yml` and start manually. 
You can also NOT use docker, but then you need to install both variants on your hardware manually (those steps depend then on your OS).
If you do this, please stick to the default ports (which are also mentioned in the docker compose yml file.


### Starting GAE binary
You need ArangoDB up and running (as GAE will connect to ArangoDB). Execute this command from the main directory:

```
./target/release/gral --bind-port 9999 --arangodb-endpoints http://localhost:8529 --arangodb-jwt-secrets ./secrets.jwt &> api_tests/logs/arangodb_auth_release.txt &
```

### Data Import ArangoDB

Working directory for data imports into ArangoDB is:
`gral/examples`

Please see detailed description down below.

### Data Import Neo4j

Working directory for data imports into Neo4j is:
`gral/examples/neo4j`

Please see detailed description down below.

## Introduction

This benchmark is designed to evaluate the performance of the analytic capabilities of graph databases in a fair
and unbiased way. The benchmark is based on the LDBC GRAPHALYTICS BENCHMARK (LDBC GRAPHALYTICS) and uses the same
data to create the datasets in each database instance. In case anything is unclear or potentially wrongly used,
please create an issue or a pull request, and we will review it as soon as possible.

### First environment definition (obsolete)

The environment setup of the first comparison is as follows:

- ArangoDB:
  - started as a single server instance
  - authentication enabled
  - inside docker container

- Neo4j:
  - started as a single server instance
  - authentication enabled
  - inside docker container

### Second environment definition (we decided to use cluster for benchmarks)

The environment setup of the second comparison is as follows:

- ArangoDB:
  - started as a local cluster instance
  - no authentication enabled
  - inside docker container

As Neo4j does not offer a sharded cluster environment, we keep the same environment as in the first comparison.
While the environment is different, the benchmark is executed in the same way as in the first comparison. Also, please
note that ArangoDB now needs additional network requests as it is communicating with additional endpoints.

### Test data definition

The benchmark uses the following graph datasets:

| Graph            | Vertices   | Edges        |
|------------------|------------|--------------|
| datagen-8_0-fb   | 1,706,561  | 107,507,376  |
| dota-league      | 61,170     | 50,870,313   |
| kgs              | 832,247    | 17,891,698   |
| wiki-Talk        | 2,394,385  | 5,021,410    |

With little modifications to the source-code, you can use any of the LDBC datasets available at:
* https://ldbcouncil.org/benchmarks/graphalytics/

The selection of the datasets is based on the size of the graph and the number of edges. The datasets are chosen
to represent a wide range of graph sizes and edge counts, to evaluate the performance of the graph databases in
different scenarios (from small to large graphs).

### Test execution definition

The benchmark is executed using the following steps.

#### Preparation

- Start ArangoDB and Neo4j instances
- Load the datasets into the databases using the provided scripts or manually by your own (if you prefer to do so)

Detailed descriptions of the preparation steps can be found in the following sections.

#### Testcases

Currently, we are measuring only algorithm execution relevant timings. This is split up in two different scenarios:
- Scenario A: Full execution of loading the graph into memory and executing the algorithm
- Scenario B: Full execution as well, but this time we are measuring loading times and algorithm execution times separately

Both scenarios are executed in both environmets:
- Environment A: ArangoDB as a SingleServer instance and Neo4j as a SingleServer instance
- Environment B: ArangoDB as a local cluster instance and Neo4j as a SingleServer instance

The list of used algorithms is as follows:
- PageRank
- Weakly Connected Components
- Strongly Connected Components
- Label Propagation

## Server environment

All database instances are deployed using docker. Either use the attached `docker-compose.yml` file or start the
instances manually as described below.

## Client environment

The client environment is a local machine with Node.js installed. The benchmark is executed using the provided
Node.js script. The script is designed to be executed in a terminal and provides a set of parameters to adjust the
benchmark execution to your needs.

**To install all required dependencies, run the following command:**

```bash
npm install
```

**in all directories containing a `package.json` file**. In case you don't want to use the provided scripts, just read
the source code and execute the commands manually. There are two main helper files which are the most relevant for 
the code execution. One ArangoDB helper file and one Neo4j helper file. Both files contain the same functions but are
adjusted to the specific database.

The benchmark itself is being executed with the JavaScript framework `vitest` which is a simple test runner, but
includes build-in support for benchmarking using `tinybench`. The benchmark is executed in a synchronous way, so
each algorithm is executed one after another. This is done to ensure that the results are not influenced by each other.

## ArangoDB as a SingleServer instance

Start ArangoDB SingleServer instance with authentication enabled.
In the example we make the instance available on port 10000. You can choose any port you like.
At the time of writing this, the latest version of ArangoDB is `3.12.0-2`.

```bash
docker run -d -e ARANGO_ROOT_PASSWORD="" --name arangodb-singleserver-instance -p 10000:8529 \
  arangodb/arangodb arangod --server.endpoint tcp://0.0.0.0:8529
```

## ArangoDB as a local cluster instance

Start ArangoDB local cluster instance with no authentication enabled.
In the example we make the instance available on port 10001. You can choose any port you like.
At the time of writing this, the latest version of ArangoDB is `3.12.0-2`.

```bash
docker run -d -e --name arangodb-cluster-instance \
  -p 8529:8529 arangodb/arangodb arangodb --mode=cluster --local=true --auth.jwt-secret=./secrets/token 
```

### Download dataset

There are scripts prepared which allow you do download the datasets needed locally to your machine.
The working directory for those is: `gral/examples`

You need to download:
- `datagen-8_0-fb`
- `dota-league`
- `kgs`
- `wiki-Talk`

This can be done with: 
```bash
./scripts/downloadSingleDataset datagen-8_0-fb
```

```bash
./scripts/downloadSingleDataset dota-league
```

```bash
./scripts/downloadSingleDataset kgs
```

```bash
./scripts/downloadSingleDataset wiki-Talk
```



### Import ldbc datasets into ArangoDB(s)

Working directory for data imports into ArangoDB is:
`gral/examples`

datagen-8_0-fb
```bash
node main.js --graphName datagen-8_0-fb -d true --mqs 200 --con 30 -e http://127.0.0.1:8529
```

dota-league
```bash
node main.js --graphName dota-league -d true --mqs 200 --con 30 -e http://127.0.0.1:8529
```

kgs
```bash
node main.js --graphName kgs -d true --mqs 200 --con 30 -e http://127.0.0.1:8529
```

wiki-Talk
```bash
node main.js --graphName wiki-Talk -d true --mqs 200 --con 30 -e http://127.0.0.1:8529
```

Note: Please adjust the `--con` and `--mqs` parameters according to your system resources.
Also adjust the `--e` parameter to the correct endpoint if you are using a different port.

With `--help` you can see all available parameters.

## Neo4j

Start Neo4j instance with authentication enabled and gds plugin enabled.
At the time of writing this, the latest version of Neo4j is `5.20.0`.

```bash
docker run -d --publish=7474:7474 --publish=7687:7687 --user="$(id -u):$(id -g)" \
  -e NEO4J_AUTH=neo4j/ --name neo4j-instance \
  --env NEO4J_PLUGINS='["graph-data-science"]' \
  neo4j:latest
```

### Import ldbc datasets into Neo4j

Working directory for data imports into ArangoDB is:
`gral/examples/neo4j`

datagen-8_0-fb
```bash
node main.js --graphName datagen-8_0-fb --dropGraph true --mqs 20 --concurrent 10
```

dota-league
```bash
node main.js --graphName dota-league --dropGraph true --mqs 20 --concurrent 10
```

kgs
```bash
node main.js --graphName kgs --dropGraph true --mqs 20 --concurrent 10
```

wiki-Talk
```bash
node main.js --graphName wiki-Talk --dropGraph true --mqs 20 --concurrent 10
```

Note: We are using the default ports for Neo4j. If you are using a different port, please adjust the script to your
needs. The import here is slower compared to ArangoDB as this is not optimized. This is just a side note and does not
affect the benchmark results. Please feel free to import the data in a more optimized way e.g. with `apoc` module.

## Benchmark Execution

Working directory for the benchmark execution:
`gral/competitive_evaluations`

Either execute 
```bash
npm run benchmark
``` 

or execute the following command:

```bash
vitest bench --run --bail 1 --maxConcurrency 1 --outputJson <your-desired-name>.json
```

You can also executing each available benchmark separately by appending the file name to the command.

Example to execute only one benchmark file (dota-league-split-compute.bench.ts)
```bash
vitest bench --run --bail 1 --maxConcurrency 1 --outputJson <your-desired-name>.json dota-league-split-compute.bench.ts
```

For more details, please refer to the `vitest` documentation:

* https://vitest.dev/api/

