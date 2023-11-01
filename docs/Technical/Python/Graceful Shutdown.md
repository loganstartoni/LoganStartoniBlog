---
title: Graceful Shutdown
summary: A simple way to Gracefully Shutdown your program instead of an instant shutdown. Allowing you to close things out. 
authors:
    - Logan Startoni
date: 2023-09-09
---
# Overview
This Describes the process of allowing your program to gracefully shutdown instead of being force quit suddently. Allowing you to close things down correctly instead of leaving things hanging. 

# Python Class Needed

```python
import signal
import time

class GracefulShutdown:
  kill_now = False
  
  def __init__(self):
      """The initializer to start watching for sigint, and sigterm signals."""
    signal.signal(signal.SIGINT, self.exit_gracefully)
    signal.signal(signal.SIGTERM, self.exit_gracefully)

  def exit_gracefully(self, *args):
    """Update the saved value of kill_now to be true to let who ever is watching know its time to start shutting down."""
    self.kill_now = True

```


###
Source: https://stackoverflow.com/questions/18499497/how-to-process-sigterm-signal-gracefully
