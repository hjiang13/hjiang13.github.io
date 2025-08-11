---
title: "HAPPA: HPC Application Resilience Analysis Platform"
collection: portfolio
excerpt: "A modular platform for HPC Application Resilience Analysis that integrates Large Language Models to understand long code sequences and predict resilience with superior accuracy."
date: 2024-01-01
image: 
  path: /images/hpc-resilience.jpg
  alt: "HAPPA Platform Architecture"
links:
  - label: "GitHub"
    url: "https://github.com/hjiang13/CODEBERT-REGRESSION"
  - label: "Paper"
    url: "https://ieeexplore.ieee.org/abstract/document/10806619"
  - label: "Dataset"
    url: "https://github.com/hjiang13/DARE"
  - label: "Conference"
    url: "https://sc24.supercomputing.org/"
---

## Project Overview

HAPPA (HPC Application Resilience Analysis Platform) is my flagship research project that represents a breakthrough in combining Large Language Models with high-performance computing resilience analysis. This work was presented at the prestigious SC'24 conference as part of the Doctoral Showcase and has been published in the **SRDS'24** conference proceedings.

## Key Innovations

### LLM Integration for Code Understanding
- **Long Sequence Processing**: Advanced LLM integration to understand and analyze long code sequences
- **Code Representation**: Innovative techniques for representing HPC applications in a format suitable for LLM analysis
- **Semantic Analysis**: Deep understanding of code semantics beyond traditional static analysis

### Superior Predictive Performance
- **DARE Dataset**: Utilized the comprehensive DARE (Dataset for REsilience analysis using Fault Injection) dataset for training and evaluation
- **MSE Achievement**: Achieved mean squared error (MSE) of 0.078 in Silent Data Corruption (SDC) prediction
- **Benchmark Comparison**: Significantly outperformed the existing PARIS model (MSE: 0.1172)
- **Accuracy Improvement**: 30% improvement in resilience prediction accuracy

## Technical Architecture

The platform consists of several sophisticated components:

- **LLVM Integration**: Custom passes for soft error injection and analysis
- **Neural Network Models**: BERT-based code understanding for resilience prediction
- **Real-time Monitoring**: Continuous system health assessment during execution
- **Scalable Design**: Support for large-scale HPC applications and distributed systems

## Research Contributions

### 1. HPC Loop Resilience Analysis
- **13 Dwarfs of Parallelism**: Comprehensive analysis of computational patterns
- **SDC Rate Quantification**: Quantified Silent Data Corruption rates for each pattern
- **Prompt Engineering**: Advanced LLM prompting techniques for loop semantics identification
- **Error-Prone Loop Detection**: Identified which loops are more susceptible to errors

### 2. IR Code Analysis with LLMs
- **Model Evaluation**: Comprehensive testing of GPT-4o, GPT-3.5, and CodeLlama
- **Low-Level Analysis**: Decompiling IR code, generating Control Flow Graphs (CFGs)
- **Code Execution Simulation**: Advanced simulation capabilities for IR code analysis
- **Program Analysis Applications**: Insights into LLM effectiveness for compiler-level tasks

### 3. Novel Code Representation Module
- **Code Chunking**: Fixed-size segment division for long code sequences
- **Embedding Aggregation**: Three aggregation methods explored:
  - MeanPooling technique
  - MaxPooling technique
  - LSTM-based aggregation (HAppA-LSTM)
- **Context Preservation**: Maintains code context information across segments

### 4. KeyBERT Integration
- **Keyword Extraction**: Automatic identification of source code key words
- **Importance Analysis**: Comprehensive analysis of code patterns contributing to error rates
- **Pattern Recognition**: Enhanced understanding of resilience factors

## Impact and Recognition

- **Conference Publication**: Published in SRDS'24 - International Symposium on Reliable Distributed Systems
- **Conference Presentation**: Featured at SC'24 Doctoral Showcase in Atlanta, GA
- **Research Recognition**: Significant contribution to AI-driven HPC resilience
- **Industry Relevance**: Addresses critical challenges in system reliability and fault tolerance
- **Academic Contribution**: Advances the intersection of artificial intelligence and high-performance computing

## Future Applications

This platform serves as a foundation for:
- **Intelligent Fault Tolerance**: AI-driven system reliability enhancement
- **Compiler Optimization**: LLM-assisted code analysis and optimization
- **HPC System Design**: Data-driven approaches to resilient system architecture
- **Research Collaboration**: Framework for future HPC resilience research

## Technical Details

- **Programming Languages**: C++, Python, LLVM IR
- **ML Frameworks**: PyTorch, Transformers, BERT, LSTM
- **Tools**: LLVM/Clang, CUDA, OpenMP, KeyBERT
- **Platform**: Linux-based HPC environments
- **Performance**: Real-time analysis with minimal overhead

## Publication Details

- **Conference**: SRDS'24 - 43rd International Symposium on Reliable Distributed Systems
- **Date**: September 23, 2024
- **DOI**: [IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/10806619)
- **Code**: [GitHub Repository](https://github.com/hjiang13/CODEBERT-REGRESSION)
- **Dataset**: [DARE Dataset](https://github.com/hjiang13/DARE)

This work represents a significant milestone in my PhD research and demonstrates the potential of combining cutting-edge AI technologies with traditional HPC system analysis. The publication in SRDS'24 validates the technical contributions and establishes HAPPA as a benchmark for predictive accuracy in HPC resilience analysis. 
