---
title: Settings Management 
summary: What I like to use for managing settings in my code.
authors:
    - Logan Startoni
date: 2025-03-22
---
# Overview
Below you will find my settings management. This is the pattern that I like to follow, to easily maintain settings via a common interface.

# Python Code:
This sets up the configuration for four different environments. Through inheritance it can override just the variables that are environment specific. 

```python
import os
from functools import lru_cache

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    """Base Settings class where all values should be defined."""
    model_config = SettingsConfigDict(env_prefix='PROJECT_PREFIX_')
    # Test Configuration
    debug: bool = True


class PyTestingSettings(Settings):
    """Settings for the app when it is run for Testing."""

    # Test Configuration
    debug: bool = False


class DockerSettings(Settings):
    """Settings for the app when it is run in Docker."""

    whisper_model: str = 'turbo'


class ProductionSettings(DockerSettings):
    """Settings for the app when it is run in Production."""

    # Test Configuration
    debug: bool = False

@lru_cache()
def get_settings(*, override_environment: str = None) -> Settings:
    """Get the settings based on the environment."""
    env = os.getenv("PROJECT_PREFIX_ENVIRONMENT", "production")
    if override_environment:
        env = override_environment
    env = env.lower()
    if env == "development":
        return Settings()
    elif env == "docker":
        return DockerSettings()
    elif env == "testing":
        return PyTestingSettings()
    else:
        return ProductionSettings()

```