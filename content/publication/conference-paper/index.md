---
title: 'HAppA: A Modular Platform for HPC Application Resilience Analysis with LLMs Embedded'

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
  - admin

# Author notes (optional)
author_notes:
  - 'Equal contribution'
  - 'Equal contribution'

date: '2024-09-23T00:00:00Z'
doi: ''

# Schedule page publish date (NOT publication's date).
publishDate: '2017-01-01T00:00:00Z'

# Publication type.
# Accepts a single type but formatted as a YAML list (for Hugo requirements).
# Enter a publication type from the CSL standard.
publication_types: ['paper-conference']

# Publication name and optional abbreviated publication name.
publication: In 2024 43rd International Symposium on Reliable Distributed Systems (SRDS)
publication_short: In *SRDS'24*

abstract: High-performance computing (HPC) systems are increasingly vulnerable to soft errors, which pose significant challenges in maintaining computational accuracy and reliability. Predicting the resilience of HPC applications to these errors is crucial for robust code protection and detailed resilience analysis. In this study, we present HAppA, a modular platform designed for HPC Application Resilience Analysis. Embedding Large Language Models (LLMs), HAppA addresses understanding the context information of long code sequences typical in HPC applications. HAppA implements a novel code representation module that chunks the code into fixed-size segments and aggregates the embeddings of these segments. Three aggregation methods have been explored -- MeanPooling, MaxPooling, and LSTM-based techniques. We built a DAtaset for REsilience analysis using Fault Injection (FI), named DARE. Using our DARE dataset, HAppA is trained for regression prediction tasks. Our evaluation results demonstrate the predictive accuracy of HAppA compared to other models, particularly noting that the LSTM-based aggregation method - HAppA-LSTM - achieves a mean squared error (MSE) of 0.078 for SDC prediction, surpassing the existing state-of-the-art PARIS model, which recorded an MSE of 0.1172. Additionally, HAppA with the KeyBERT model extracts a list of key words representing the source code. A comprehensive importance analysis of these key words further elucidates the code patterns contributing to the error rate. These findings highlight the effectiveness of HAppA in analyzing the resilience of HPC applications and establish a new benchmark for predictive accuracy in resilience.

# Summary. An optional shortened abstract.
#summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags:
  - HPC system 
  - Resilience Analysis
  - Large Language Models

# Display this page in the Featured widget?
featured: true

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://ieeexplore.ieee.org/abstract/document/10806619'
url_code: 'https://github.com/hjiang13/CODEBERT-REGRESSION'
url_dataset: 'https://github.com/hjiang13/DARE'
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
image:
  caption: 'Overview of the framework'
  focal_point: ''
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
  - Resilience Analysis

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: example
---

{{% callout note %}}
Click the _Cite_ button above to to import publication metadata into your reference management software.
{{% /callout %}}
