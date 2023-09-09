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

## In the Main Module
```python
import logging

log_format = ""
logging.basicConfig(format=log_format)

logger = logging.getLogger()
```

### Description of Above
1. Import the logging module
   - Allows us to use the internal logging module in python
2. Setup the log format, this sets up a standard log format that will be used across your file. The documentation for what is available is located [here](https://docs.python.org/3/library/logging.html#logrecord-attributes). My Logstring does the following. I found that it helps me with debugging by having it setup this way.
   - TODO: Add my goto log string here.
3. Configure the logging library with the format.
4. get a logger instance.


## In any supporting File
```python
import logging

logger = logging.getLogger()
```

The logger here is configured with the configuration that was applied in the main file so that only needs to be configured in one location and then the other loggers in the module pick up that configuration. 