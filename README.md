# Nocky Cyber Lab
Hi, I am Nocky256 (CeoNocky) Uganda 🇺🇬 

This is where I learn Cyber Security - not to hack, but to protect.

## What I am learning now:
- [x] Created my first GitHub repo
- [ ] Python basics
- [ ] How networks work
- [ ] Ethical hacking labs on TryHackMe

## My goal:
To become a Cyber Security Analyst from Uganda 🇺🇬 

Contact: ceonocky
import os
import sys
import time
import asyncio
import random
import heapq
import logging
import subprocess
from concurrent import futures

import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from scipy.stats import ks_2samp
from prometheus_client import start_http_server, Gauge, Counter
import grpc
from fastapi import FastAPI
from pydantic import BaseModel
import uvicorn

# =====================================================================
# 0. AUDIT LOGGING CONFIGURATION
# =====================================================================
logging.basicConfig(
    filename="audit.log",
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)

def audit_log(message: str, level: str = "INFO"):
    print(message)
    if level == "WARNING":
        logging.warning(message)
    elif level == "ERROR":
        logging.error(message)
    else:
        logging.info(message)


# =====================================================================
# 1. SETUP ENVIRONMENT (Certificates & Protobuf)
# =====================================================================
def setup_environment():
    os.makedirs("certs", exist_ok=True)
    os.makedirs("proto", exist_ok=True)

    if not os.path.exists("certs/server.crt"):
        audit_log("[SETUP] Generating mTLS certificates...")
        subprocess.run('openssl req -x509 -newkey rsa:2048 -days 365 -nodes -keyout certs/ca.key -out certs/ca.crt -subj "/CN=nexdominon-ca"', shell=True, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        subprocess.run('openssl req -newkey rsa:2048 -nodes -keyout certs/server.key -out certs/server.csr -subj "/CN=localhost"', shell=True, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        subprocess.run('openssl x509 -req -in certs/server.csr -CA certs/ca.crt -CAkey certs/ca.key -CAcreateserial -out certs/server.crt -days 365', shell=True, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        subprocess.run('openssl req -newkey rsa:2048 -nodes -keyout certs/client.key -out certs/client.csr -subj "/CN=worker-node"', shell=True, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        subprocess.run('openssl x509 -req -in certs/client.csr -CA certs/ca.crt -CAkey certs/ca.key -CAcreateserial -out certs/client.crt -days 365', shell=True, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

    proto_content = """syntax = "proto3";
package nexdominon;

service
