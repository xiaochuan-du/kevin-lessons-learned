---
title: "X methods to manage python configuration"
slug: "python-configuration-methods"
date: 2019-12-18T00:00:00+11:00
draft: false
description: "A survey of configuration-management options for Python applications, including files, environment variables, and Hydra."
tags: ["Python","configuration","software engineering"]
canonicalURL: "https://dxiaochuan.medium.com/summary-of-python-config-626f2d5f6041"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/summary-of-python-config-626f2d5f6041).*

## Updates

- 2020–02–21: change the title and content from “5 methods” to “X methods” to scale for future updates. Add an introduction to hydra.

- 2021–01–27: added gin config in the reference

- 2021–06–01: added dynaconf in the reference

## Introduction

Config management (to be clarified, not software configuration management)for python application is a common but not trivial task. A good design config management should support a flexible parameter setting and dynamic loading. Here, I would like to summarize the frequently used configuration management strategies. Hopefully, you can find the most suitable methods to help you create applications.

- Config file(Json / yaml / INI / *.cfg).

- Database

- Centralized Configuration service

- Operation system’s environment variables

- Python scripts

- Multilayer config

- Hydra

- Gin config

- dynaconf

Before digging deep into the introduction, let’s look at what a good config management function looks like?

Traditionally, Configuration files are used to manage various settings. We might have different settings for the different environments (dev/staging/ prod …), and during development and operation, we may update parameters from time to time. Here, I use the following treats to define a good design:

- Config status should be tracked by software configuration management systems(SCM, like git, ClearCase, svn, …)

- Config status should be grouped into clusters so that DevOps is able to load different clusters according to requirements.

- Config status should be able to be overwritten without changing source code.

- Config status should support comments to make it easy to maintain.

Now let’s look at whether the mentioned ways can fulfill these needs.

## 1. Config file(JSON / YAML / INI / *.cfg)

Config files are trackable to SCM, support comments, and developers can put different parameters into different config files, thus they are DevOps friendly. But developers’ have to update parameters in their source code when needed, this makes it a little bit harder to debug config related errors.

Some example in different formats are listed as follows:

JSON:

```
// config.json
{
    "mysql":{
        "host":"localhost",
        "user":"root",
        "passwd":"my secret password",
        "db":"write-math"
    },
    "other":{
        "preprocessing_queue":[
            "preprocessing.scale_and_center",
            "preprocessing.dot_reduction",
            "preprocessing.connect_lines"
            ],
        "use_anonymous":true
    }
}
```

Loading:

```
import json

with open('config.json') as json_data_file:
    data = json.load(json_data_file)
print(data)
```

YAML (need `pyyaml`):

```
mysql:
    host: localhost
    user: root
    passwd: my secret password
    db: write-math
other:
    preprocessing_queue:
        - preprocessing.scale_and_center
        - preprocessing.dot_reduction
        - preprocessing.connect_lines
    use_anonymous: yes
```

Loading

```
import yaml

with open("config.yml", 'r') as ymlfile:
    cfg = yaml.load(ymlfile)
```

INIT

```
[mysql]
host=localhost
user=root
passwd=my secret password
db=write-math

[other]
preprocessing_queue=["preprocessing.scale_and_center",
                     "preprocessing.dot_reduction",
                     "preprocessing.connect_lines"]
use_anonymous=yes
```

Loading

```
#!/usr/bin/env python

import ConfigParser
import io

# Load the configuration file
with open("config.ini") as f:
    sample_config = f.read()
config = ConfigParser.RawConfigParser(allow_no_value=True)
config.readfp(io.BytesIO(sample_config))

# List all contents
print("List all contents")
for section in config.sections():
    print("Section: %s" % section)
    for options in config.options(section):
        print("x %s:::%s:::%s" % (options,
                                  config.get(section, options),
                                  str(type(options))))

# Print some contents
print("\nPrint some contents")
print(config.get('other', 'use_anonymous'))  # Just get the value
print(config.getboolean('other', 'use_anonymous'))
```

## 2. Database

Reading/writing config into a database is not as popular as config files. However, in some scenarios, they are helpful to decouple the system.

For example, when the operation and development team are completely isolated, and system maintainers have limited experience in programming and operation system, but they are good at operating database. Using the database makes the config updating more robustness to typo, and avoids the troubles of manually rebooting the program or reloading config.

Config in a database is not trackable by SCM, and online overwritten seems to be meaningless to this method.

## 3. Centralized Configuration service

In this day and age, microservices are becoming more and more adopted. Some practitioners think it is cool to use a centralized configuration service to manage all system configs. The config users and config operators should reach a consensus on config API, and then an ID token will be allocated for the config users. After that, config users are able to use this token to fetch or update configs.

The method can meet our predefined requirements, but it tends to be expensive to develop and operate a centralized service. Besides, the high availability (HA) of the service is crucial, because it is a potential single point of failure for the system.

## 4. Operation system’s environment variables

According to the [twelve-factor app](https://12factor.net/), this is the most recommended way to config. But there are some flaws in practice.

- Too many environment variables to maintain

- Not trackable for SCM

- Hard to record comments for parameters

- An independent environment variable list should be handed over from dev to ops

## 5. Python scripts

Employing python script sometime can leverage the pros of environment variables and config files. In addition, it enables developers to put clear logics into the config file, which brings more flexibility. Last but not least, it facilitates using software engineering practices to write modularized and clear parameters setting.

Example:

```
//config.py
from pathlib import Path
import os
import socket

basedir = os.path.abspath(os.path.dirname(__file__))

__version__ = "0.1.22"
class Config:
    HOST_NAME = socket.getfqdn(socket.gethostname())
    HOST_IP = socket.gethostbyname(HOST_NAME)

    # s3 setting
    AWS_REGION_NAME = 'us-east-1'
    S3_BUCKET_NAME = 'default'
    S3_PREFIX = 'dfs'

    # DIR
    DATA_ROOT = Path(f'{basedir}/data')
    MODEL_ROOT = None

    # Debug setting
    DEBUG = True
class OnPremiseWorker(Config):
    DATA_ROOT = Path(os.getenv(
      'APP_DATA_ROOT'),
      f'/data/xxx/data'
    )
    MODEL_ROOT = Path(os.getenv(
      'APP_MODEL_ROOT'),
      f'/data/xxx/models'
    )
    DEBUG = Path(os.getenv(
      'APP_VERBOSE'),
      0
    )
class Debug(Config):
    DEBUG = True
config_cls = {
    'default': Debug,
    'prem': OnPremiseWorker,
    'debug': Debug,
}
```

Loading

```
from config import config_cls
config = config_cls[os.getenv('ENV', 'default')]
print(config.DEBUG)
```

## 6. Multilayer config

The aforementioned methods all have their pros and cons in practice, so someone figures out it is a good strategy to simply use them all in their software, and config values come from different sources take different priority. For example, configs set in environment variables will overwrite the value in config files. A good example is [Apache Airflow](https://airflow.apache.org/) project, the universal order of precedence for all configuration options is as follows:

```
1. set as an environment variable
2. set as a command environment variable
3. set in airflow.cfg
4. command in airflow.cfg
5. Airflow’s built in defaults
```

This implementation takes the burden off developers with flexible configuration choices. If you want to steal their implementation, please click [here](https://github.com/apache/airflow/blob/175a1604638016b0a663711cc584496c2fdcd828/airflow/configuration.py#L233).

## 7. Hydra

Multilayer config and python scripts seem to be good enough to solve 90% of daily tasks. But when you need to write configuration files for machine learning applications, you probably will be surprised by how many configuration items you need to set and how verbose your scripts are. Could we have a hierarchical configuration by composition and override it through config files and the command line? (adopting command-line arguments is a well-known config method, I ignore that in this article, but it is still very useful for simple applications)

Thanks to [Omry Yadan](https://github.com/omry) who created a tool called [hydra](https://hydra.cc/) for Facebook AI which takes config python software to another level and saves us from endless technical selection discussion with teammates. The following is a simple example.

Configuration file: config.yaml

```
db:
  driver: mysql
  user: omry
  pass: secret
```

Python file: `my_app.py`

```
import hydra
from omegaconf import DictConfig
@hydra.main(config_path="config.yaml")
def my_app(cfg : DictConfig) -> None:
    print(cfg.pretty())
if __name__ == "__main__":
    my_app()
```

It seems that the only downside of Hydra is environment variables support, but due to the fact that it is a newly created project, and the function possibly be implemented in the foreseeing future [[issue](https://github.com/facebookresearch/hydra/issues/7)], it is an extremely promising project to follow.

## 8. Gin config

Gin provides a lightweight configuration framework for Python, based on dependency injection. Functions or classes can be decorated with `@gin.configurable`, allowing default parameter values to be supplied from a config file (or passed via the command line) using a simple but powerful syntax. It is powerful to config a project with a single entry point, especially when using a CLI & config files setting. But compared with other methods, the dependency injection based config is not well understood by the data science community, and they may not play very well with your favorite IDE.

## 9. dynaconf

Dynaconf supports Multiple file formats `toml|yaml|json|ini|py` and also customizable loaders. It also fully supports for environment variables to override existing settings (dotenv support included). The other handy function it has is it has optional layered system for multi environments `[default, development, testing, production]`

Moreover, it provides a way to protect sensitive information (passwords/tokens).

After loading the config object, basically you can use it as a standard python object, so it has a similar advantage of the`python script` method.

## Conclusion:

Before the emergence of dynaconf, I recommend using python scripts and Multilayer config for configuration due to their flexibility and low maintenance. But now, I suggest to use dynaconf as a default method and only consider switching to python scripts when you have some limitations of your deployed environment.

Besides, for some special occasions, database, and centralized config servers still have their place.

## References:

- Configuration files in Python

- configparser

- Microservices Architectures: Centralized Configuration and Config Server

- The twelve-factor app

- Hydra — A fresh look at configuration for machine learning projects

- OmegaConf

- Gin config

- dynaconf
