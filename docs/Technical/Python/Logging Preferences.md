---
title: Logging Preferences 
summary: What I like to use for when implementing logging in python modules.
authors:
    - Logan Startoni
date: 2023-09-09
---
# Overview
This describes how I like to configure my logging for python projects that I write. Including the log output format. 

# Getting Started

## In the file that launches your python program
```python
import logging

log_format = "%(asctime)s - %(levelname)s - %(filename)S:%(funcname)s:%(lineno)d - %(message)s"
logging.basicConfig(format=log_format)

logger = logging.getLogger()
```

### Description of Python Code
1. Import the logging module
    - Allows us to use the internal logging module in python
2. Setup the log format, this sets up a standard log format that will be used across your file. The documentation for what is available is located [here](https://docs.python.org/3/library/logging.html#logrecord-attributes). My Logstring does the following. I found that it helps me with debugging by having it setup this way.
    - "%(asctime)s - %(levelname)s - %(filename)S:%(funcname)s:%(lineno)d - %(message)s
      - asctime - The time the log was sent
      - levelname - The log level that was use to send the message
      - filename - The file the log was sent from
      - funcname - The function the log was sent from
      - lineno - The line number the log was sent from
      - message - The log message
3. Configure the logging library with the format.
4. get a logger instance.


## After Configured Logging Usage
```python
import logging

logger = logging.getLogger()
```

The logger here is configured with the configuration that was applied in the main file so that only needs to be configured in one location and then the other loggers in the module pick up that configuration. 
