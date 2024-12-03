# FAIR Workflow platform architecture documentation

This document loosely follows the [ARC42](https://arc42.org/overview) template for architecture documentation.

It describes the architecture in the context of the [Destination Earth platform](https://destination-earth.eu/).

## 1. Overview

The FAIR Workflow Platform is designed to allow the execution
and sharing of machine-actionable workflows.
It adheres to the [FAIR principles](https://www.go-fair.org/fair-principles/) (Findable, Accessible, Interoperable, Reusable).
The platform follows an [RO-Crate](https://www.researchobject.org/ro-crate/)-centric approach.
Workflows can be submitted as RO-Crates and their output is available as RO-Crate as well.
By leveraging FAIR Signposting, it allows integrating resulting dataset in the greater linked data ecosystem.

### Key Features

- A user-facing frontend for workflow submission and viewing/retrieving datasets.
- Automatic tracking of workflow execution provenance.
- Datasets are available in a standardized format and described with rich metadata. This allows to share them across data spaces and research domains.

## 3. Context

![Context Diagram](./assets/context.png)

The platform integrates the following external services:

- **ORCID**: For user authentication
- **Linked data context providers**: Provide JSON-LD context for handling metadata
- **S3 Storage**: Object storage for digital objects and workflow data.
- **External Data Repositories**: Workflows can access data from external providers like GBIF or the Copernicus Climate Data Store.
- **DEDL HDA Bridge**: Workflows can access earth observation data and the Destination Earth Digital Twins through the Harmonized Data Access API.
- **External code/container repositories**: Workflows can use code or the execution environment description (docker container) stored in external repositories like Github or Docker Hub.

Human users interact with the platform via the frontend, which handles authentication, workflow submission, and dataset retrieval. Users can also access the Digital Object repository directly. 

Machine agents can retrieve datasets in a machine-readable format through the frontend and Digital Object repository. FAIR signposting
links human-readable frontpages to the machine-readable datasets.


## 4. Solution Strategy

The solution is built with a microservice architecture.
This should ensure extensibility and allow for individual building blocks being replaced.

1. **Frontend:** Manages user interactions and submissions. Provides FAIR signposting to machine-readable content.
2. **Digital Object repository:** Stores datasets.
3. **Submission Service:** Acts as a bridge between the frontend, Digital Object repository and the workflow engine.
4. **Workflow Engine:** Executes workflows.

## 5. Building Blocks

![Component Diagram](./assets/components.png)

### 5.1 Frontend

#### Responsibilities

- Allows users to login with ORCID.
- Provides a GUI for submitting workflows as RO-Crates and monitoring their progress.
- Displays datasets and their provenance.
- Dynamically builds RO-Crates for export and allows to download them or retrieve them as "detached" RO-Crate.
- Provides an admin interface for managing registered users.

#### Interactions

- Submits workflows and metadata to the submission service.
- Queries the Digital Object repository for digital objects.

#### Technical details

- Django frontend with small sprinkles of JavaScript.
- Uses ro-crate-py to build ro crates.

### 5.2 Digital Object repository (Cordra)

#### Responsibilities

- Stores digital objects for data, workflows and respective metadata.

#### Interactions

- Provides data to the frontend.
- Receives data from the submission service.

#### Technical details

- A cordra instance.
- Cordra schema closely resembles the profile of Workflow Run RO-Crates.
- Corda hooks written in Java/Kotlin ensure that all metadata documents are valid JSON-LD.
- Backed by an S3 Bucket for storage.

### 5.3 Submission Service (Argo Connector)

#### Responsibilities

- Orchestrates workflow submission and ingestion of results.

#### Interactions

- Retrieves workflow submissions from frontend and queues the workflow to the workflow engine
- Retrieves workflow results and metadata from the workflow engine upon completion
- Ingests workflows and artifacts into the Digital Object repository

#### Technical details

- Built with FastAPI
- Annotates workflow submissions with metadata for provenance.
- Reads workflow annotations and stored workflow archives from the engine and ingests them into cordra.

### 5.4 Workflow Service (Argo Workflow Engine)

#### Responsibilities

- Executes workflows
- Stored workflow artifacts for submission service
- Notifies the Submission service about finished workflows

#### Interactions

- Retrieves workflows from Submission services and notifies on workflow completion
- Stores workflow artifacts

#### Technical details

- Supports secure key management via Kubernetes secrets
- Automatically adds exit-handlers to submitted workflows to notify the submission service
- Archives workflow artifacts into an S3 bucket.

## 6. Runtime View

### 6.1 Workflow Submission

### 6.2 Data Ingestion

### 6.3 Data Retrieval

## 7. Deployment View

![Combonent Diagram](./assets/deployment.png)
