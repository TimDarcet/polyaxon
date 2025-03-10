---
title: "Quick Start With A Github Repo"
sub_link: "quick-start-external-repo"
meta_title: "Polyaxon quick start tutorial With Code tracking on Github - Core Concepts"
meta_description: "Get started with Polyaxon and become familiar with the ecosystem of Polyaxon with a top-level overview and useful links to get you started."
visibility: public
status: published
tags:
    - tutorials
    - concepts
    - quick-start
sidebar: "concepts"
---


This example assumes you have already gone through the [quick start introduction](/concepts/quick-start)

This example uses a Github repo for hosting and tracking code, the same principle applies to other platforms, e.g. GitLab, Bitbucket, ... 
if you want to track your code in-cluster, please check this [quick-start](/concepts/quick-start-internal-repo/) guide instead. 

### Link a Github repo to your project 

```bash
polyaxon project -p quick-start git --url="https://github.com/polyaxon/polyaxon-quick-start"
```

> N.B. if the project was private, you need to add `--private` so that Polyaxon knows that it needs to use the access token to interact with the [external platform](/integrations/scm/). 

### Initialize the project 

This step is optional, we can create a temp folder to initialize the project and take advantage of some caching optimization that the Polyaxon CLI offers.

By initializing a project, you will not need to use `-p quick-start` or `--project=quick-start` to run commands.

If you don't want to init a folder you can skip this step.

Initialize the project with the same name that you used when you created the project in Polyaxon

```bash
$ polyaxon init quick-start --polyaxonfile
```

### Check/Create a polyaxonfile 

Let's create a `polyaxonfile.yaml` if you don't have one yet 

```yaml
version: 1

kind: experiment

build:
  image: tensorflow/tensorflow:1.4.1-py3
  build_steps:
    - pip3 install polyaxon-client

run:
  cmd: python model.py
```

This configuration specifies:

   * The Polyaxon specification `version` we are using.
   * The `kind` of this operation, in this case experiment.
   * The `build` section to build a docker image,
     in this case we want to run our code with the specified tensorflow docker image.
     We are also installing the polyaxon-client for [tracking](/references/polyaxon-tracking-api/) and to send metrics at the end of the experiment.
   * The `run` section to execute our code.


### Start an experiment 

Ok let's start a first experiment using our Github repo.

If you initialized a folder run

```bash
$ polyaxon run

Experiment was created
```

Otherwise you need to run

```bash
$ polyaxon run -p quick-start -f polyaxonfile.yaml

Experiment was created
```

> N.B. even if you initialized a project, You can always switch the project context by useing `-p project-name` or `--project=project-name`

### List the project experiments 

Check your project experiments list

If you initialized a folder run

```bash
$ polyaxon project experiments
```

Otherwise you need to run

```bash
$ polyaxon project -p quick-start experiments
```

### Check the logs 

Check the experiment logs

Info:

```bash
$ polyaxon experiment -p quick-start -xp 1 get

Experiment info:

---------------------  ------------------
id                     1
unique_name            root.quick-start.1
user                   root
project_name           root.quick-start
experiment_group_name
last_status            Building
created_at             a few seconds ago
is_clone               False
num_jobs               0
finished_at
started_at
---------------------  ------------------
...
```

Logs:

```bash
$ polyaxon experiment -p quick-start -xp 1 logs
Building -- creating image -
INFO:tensorflow:global_step/sec: 1.59552
INFO:tensorflow:loss = 0.116677, step = 101 (62.679 sec)
INFO:tensorflow:global_step/sec: 1.47121
INFO:tensorflow:loss = 0.0842577, step = 201 (67.969 sec)
...
```

### Start an experiment group

Let's create another polyaxonfile:

```yaml

---
version: 1

kind: group

hptuning:
  concurrency: 2
  random_search:
    n_experiments: 10

  matrix:
    learning_rate:
      uniform: 0.00001:0.9
    dropout:
      uniform: 0.01:0.9
    activation:
      values: [relu, sigmoid]

params:
  batch_size: 64
  num_steps: 100
  num_epochs: 1

build:
  image: tensorflow/tensorflow:1.4.1-py3
  build_steps:
    - pip3 install --no-cache-dir -U polyaxon-client==0.5.2
  nocache: true

run:
  cmd:  python3 model.py --batch_size={{ batch_size }} \
                         --num_steps={{ num_steps }} \
                         --learning_rate={{ learning_rate }} \
                         --dropout={{ dropout }} \
                         --num_epochs={{ num_epochs }} \
                         --activation={{ activation }}
```

Check the run definition:

```bash
$ polyaxon check -f polyaxonfile_hyperparams.yml --definition


Polyaxonfile valid

This polyaxon specification has experiment group with the following definition:
--------------------  -----------------
Search algorithm      random
Concurrency           2 concurrent runs
Early stopping        deactivated
Experiment to create  10
--------------------  -----------------
```

Start the experiments group

```bash
$ polyaxon run -p quick-start -f polyaxonfile_hyperparams.yml

Creating an experiment group with 20 experiments.

Experiment group was created
```

### Check experiments in the group

Group details:

```bash
polyaxon group -p quick-start -g 1 get

Experiment group info:

-----------------------  ------------------
id                       1
unique_name              root.quick-start.1
user                     root
project_name             root.quick-start
created_at               a few seconds ago
concurrency              2
num_experiments          20
num_pending_experiments  18
num_running_experiments  0
-----------------------  ------------------
```

Experiments in the groups

```bash
$ polyaxon group -p quick-start -g 1 experiments

Experiments for experiment group `1`.


Navigation:

-----  --
count  20
-----  --

Experiments:

  id  unique_name            user    last_status    created_at          num_jobs  finished_at    started_at
----  ---------------------  ------  -------------  -----------------  ---------  -------------  -----------------
   4  root.quick-start.1.4   root    Created        a few seconds ago          0
   5  root.quick-start.1.5   root    Created        a few seconds ago          0
   6  root.quick-start.1.6   root    Created        a few seconds ago          0
   7  root.quick-start.1.7   root    Created        a few seconds ago          0
   8  root.quick-start.1.8   root    Created        a few seconds ago          0
   9  root.quick-start.1.9   root    Created        a few seconds ago          0
  10  root.quick-start.1.10  root    Created        a few seconds ago          0
  11  root.quick-start.1.11  root    Created        a few seconds ago          0
  12  root.quick-start.1.12  root    Running        a few seconds ago          1                 a few seconds ago
  13  root.quick-start.1.13  root    Created        a few seconds ago          0
  14  root.quick-start.1.14  root    Created        a few seconds ago          0
  15  root.quick-start.1.15  root    Running        a few seconds ago          1                 a few seconds ago
  16  root.quick-start.1.16  root    Created        a few seconds ago          0
  17  root.quick-start.1.17  root    Created        a few seconds ago          0
  18  root.quick-start.1.18  root    Created        a few seconds ago          0
  19  root.quick-start.1.19  root    Created        a few seconds ago          0
  20  root.quick-start.1.20  root    Created        a few seconds ago          0
  21  root.quick-start.1.21  root    Created        a few seconds ago          0
  22  root.quick-start.1.22  root    Created        a few seconds ago          0
  23  root.quick-start.1.23  root    Created        a few seconds ago          0
```

Comparing experiments metrics in the groups:

```bash
polyaxon group -p quick-start -g 1 experiments -m

Experiments for experiment group `1`.


Navigation:

-----  --
count  20
-----  --

Experiments:

 id  unique_name                loss          precision    accuracy
---  --__---------------------  ------------  -----------  ----------
  4   root.quick-start.1.4      0.0514547     0.999445      0.9829
  5   root.quick-start.1.5      0.0554655     0.999334      0.9833
  6   root.quick-start.1.6      0.0607866     0.999002      0.9797
  7   root.quick-start.1.7      4.62058       0.902         0.1028
  8   root.quick-start.1.8      0.379242      0.996773      0.8854
  9   root.quick-start.1.9   3635.83          0.902         0.0974
 10   root.quick-start.1.10     0.0462428     0.998892      0.9848
 11   root.quick-start.1.11     2.73637       0.902         0.1135
 12   root.quick-start.1.12     0.394347      0.997983      0.8859
 13   root.quick-start.1.13  2081.07          0             0.098
 14   root.quick-start.1.14     0.0514547     0.999445      0.9829
 15   root.quick-start.1.15     0.0554655     0.999334      0.9833
 16   root.quick-start.1.16     0.0607866     0.999002      0.9797
 17   root.quick-start.1.17     4.62058       0.902         0.1028
 18   root.quick-start.1.18     0.379242      0.996773      0.8854
 19   root.quick-start.1.19     0.388242      0.9963        0.88
 20   root.quick-start.1.20     0.0462428     0.998892      0.9848
 21   root.quick-start.1.21     2.73637       0.902         0.1135
 22   root.quick-start.1.22     0.394347      0.997983      0.8859
 23   root.quick-start.1.23     4.62058       0.902         0.1028
```

Comparing experiments params in the groups:

```bash
polyaxon group -p quick-start -g 1 experiments -d

Experiments for experiment group `1`.


Navigation:

-----  --
count  20
-----  --

Experiments:

  id  unique_name              learning_rate    num_steps    batch_size    num_epochs    dropout  activation
----  ---------------------  ---------------  -----------  ------------  ------------  ---------  ------------
  77  root.quick-start.1.77          0.1              500           128             1       0.3   relu
  76  root.quick-start.1.76          0.07525          500           128             1       0.3   sigmoid
  78  root.quick-start.1.78          0.1              500           128             1       0.3   sigmoid
  74  root.quick-start.1.74          0.0505           500           128             1       0.3   sigmoid
  75  root.quick-start.1.75          0.07525          500           128             1       0.3   relu
  71  root.quick-start.1.71          0.02575          500           128             1       0.3   relu
  73  root.quick-start.1.73          0.0505           500           128             1       0.3   relu
  72  root.quick-start.1.72          0.02575          500           128             1       0.3   sigmoid
  69  root.quick-start.1.69          0.001            500           128             1       0.3   relu
  70  root.quick-start.1.70          0.001            500           128             1       0.3   sigmoid
  68  root.quick-start.1.68          0.1              500           128             1       0.25  sigmoid
  67  root.quick-start.1.67          0.1              500           128             1       0.25  relu
  66  root.quick-start.1.66          0.07525          500           128             1       0.25  sigmoid
  64  root.quick-start.1.64          0.0505           500           128             1       0.25  sigmoid
  65  root.quick-start.1.65          0.07525          500           128             1       0.25  relu
  62  root.quick-start.1.62          0.02575          500           128             1       0.25  sigmoid
  60  root.quick-start.1.60          0.001            500           128             1       0.25  sigmoid
  59  root.quick-start.1.59          0.001            500           128             1       0.25  relu
  63  root.quick-start.1.63          0.0505           500           128             1       0.25  relu
  61  root.quick-start.1.61          0.02575          500           128             1       0.25  relu
```

Filtering only experiments with certain condition

```bash
polyaxon group -p quick-start -g 1 experiments -m -q "status:succeeded, params.activation:relu|sigmoid, metrics.loss:<=0.3"

Experiments for experiment group `1`.


Navigation:

-----  -
count  3
-----  -

Experiments:

  id  unique_name            total_run         loss    precision    accuracy
----  ---------------------  -----------  ---------  -----------  ----------
  71  root.quick-start.8.71  1m 38s       0.274588      0.995453      0.9279
  69  root.quick-start.8.69  1m 12s       0.0491856     0.999223      0.9832
  59  root.quick-start.8.59  2m 14s       0.0471044     0.999445      0.9845
```

### The dashboard 

More information about the project in the dashboard

```bash
$ polyaxon dashboard

Dashboard page will now open in your browser. Continue? [Y/n]: y
```

* Landing page:

    ![index](../../content/images/concepts/dashboard/index.png)

* Login page:

    ![login](../../content/images/concepts/dashboard/login.png)

* Projects list:

    ![project](../../content/images/concepts/dashboard/projects.png)

* Project Overview

    ![project](../../content/images/concepts/dashboard/project_overview.png)

* Experiments

    ![project](../../content/images/concepts/dashboard/experiments.png)

* Experiment Groups

    ![project](../../content/images/concepts/dashboard/experiment_groups.png)

* Experiment Overview

    ![experiment](../../content/images/concepts/dashboard/experiment.png)
    
* Experiment Metrics

    ![experiment](../../content/images/concepts/dashboard/experiment_metrics.png)
   
* Experiment Logs

    ![experiment](../../content/images/concepts/dashboard/experiment_logs.png)

### Tensorboard 

Finally, Let start a tensorboard to see the model outputs:

You can start a tensorboard for a single experiment, for all experiments under a group, or all experiments in a Project

```bash
$ polyaxon tensorboard -p quick-start -xp 23 start

Tensorboard is being deployed for experiment `23`

It may take some time before you can access tensorboard.

Your tensorboard will be available on:

    http://192.168.64.14:32566/tensorboard/root/quick-start/experiments/23/
```

Congratulations! You've trained your first experiments with Polyaxon. Behind the scene a couple of things have happened:

 * You synced your GitHub project and used last commit (can be changed to any git commit/branch)
 * You built a docker image with the latest version of your code
 * You ran the image with the specified command in the polyaxonfile
 * You persisted your logs and outputs to your volume claims
 * You created a group of experiments to fine tune hyperparameters

To gain a deeper understanding of how polyaxon can help you iterate faster with your experiments,
please take some time to familiarize yourself with the [architecture & experimentation workflow](/concepts/architecture/)
