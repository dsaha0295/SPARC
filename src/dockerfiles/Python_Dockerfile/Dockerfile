# Base image
FROM python:3.11-slim

# Install system dependencies (needed for sklearn + xgboost)
RUN apt-get update && apt-get install -y \
    build-essential \
    gcc \
    g++ \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages directly
RUN pip install --no-cache-dir \
    pandas \
    numpy \
    scikit-learn==1.6.1 \ 
    xgboost \
    joblib

