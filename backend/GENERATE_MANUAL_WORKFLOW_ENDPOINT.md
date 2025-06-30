# Generate Manual Workflow Endpoint

## Overview

The `generate_manual_workflow` endpoint is a new endpoint in `manualworkflow.py` that replaces the `generate_optimal_order` functionality. It creates complex workflows with subfolders based on AI suggested and/or manually selected technologies, with proper naming standards and sanitization.

## Endpoint Details

- **URL**: `/manualworkflow/generate_manual_workflow`
- **Method**: `POST`
- **Content-Type**: `application/json`

## Input Parameters

| Parameter         | Type   | Required | Default      | Description                                                                       |
| ----------------- | ------ | -------- | ------------ | --------------------------------------------------------------------------------- |
| `technologies`    | array  | Yes      | -            | List of technology names to include in the workflow                               |
| `use_case`        | string | Yes      | -            | Description of the use case for the workflow                                      |
| `user_code`       | string | No       | "LBA"        | User code prefix for naming                                                       |
| `environment`     | string | No       | "saas_dev"   | Target environment (saas_dev, saas_preprod, saas_prod, vse_dev, vse_qa, vse_prod) |
| `folder_name`     | string | No       | "demo-genai" | Base folder name                                                                  |
| `application`     | string | No       | "demo-genai" | Application name                                                                  |
| `sub_application` | string | No       | "demo-genai" | Sub-application name                                                              |

## Features

### 1. AI-Powered Optimal Order Generation

- Uses Azure OpenAI to determine the optimal execution order of technologies
- Considers technical dependencies, data flow, resource dependencies, and error handling
- Falls back to original order if AI fails

### 2. AI-Powered Naming Generation

- Generates descriptive names for folders and jobs based on the use case
- Makes names specific to the business context
- Falls back to sanitized default names if AI fails

### 3. Naming Standard Enforcement

- **Folder Names**: `[usercode]_[sanitized-name]`
- **Job Names**: `[usercode]-[sanitized-name]`
- **Sanitization**: Only allows letters, digits, and hyphens
- **Error Handling**: Returns error if invalid characters are detected

### 4. Workflow Creation

- Creates complex workflows with proper folder structure
- Chains jobs in optimal order
- Saves workflow JSON to `output.json`
- Returns comprehensive response with all generated data

## Response Format

```json
{
  "message": "Manual workflow generated successfully",
  "workflow": "...", // Raw workflow JSON
  "optimal_order": ["Technology1", "Technology2", ...],
  "ordered_jobs": ["USERCODE-job1", "USERCODE-job2", ...],
  "environment": "saas_dev",
  "controlm_server": "IN01",
  "folder_name": "USERCODE_sanitized-folder-name",
  "sanitized_folder_name": "sanitized-folder-name",
  "sanitized_job_names": {
    "original_tech": "USERCODE-sanitized-job-name",
    ...
  }
}
```

## Error Handling

- **400 Bad Request**: Missing required parameters or invalid environment
- **500 Internal Server Error**: Workflow creation failures, AI service errors, or file system errors

## Example Usage

```python
import requests

data = {
    "technologies": ["SAP R3", "Oracle EBS", "File Transfer"],
    "use_case": "Data migration from SAP R3 to Oracle EBS with file transfer validation",
    "user_code": "TEST",
    "environment": "saas_dev"
}

response = requests.post(
    "http://localhost:5000/manualworkflow/generate_manual_workflow",
    headers={"Content-Type": "application/json"},
    json=data
)

if response.status_code == 200:
    result = response.json()
    print(f"Workflow created: {result['folder_name']}")
    print(f"Jobs: {result['ordered_jobs']}")
```

## Sanitization Rules

The `sanitize_name()` function ensures all names follow the naming standard:

1. Replace any characters that are not letters, digits, or hyphens with hyphens
2. Remove multiple consecutive hyphens
3. Remove leading and trailing hyphens
4. Ensure the result is not empty

## Differences from generate_optimal_order

- **Complete workflow generation** instead of just optimal order
- **AI-powered naming** for folders and jobs
- **Naming standard enforcement** with sanitization
- **Comprehensive response** with all generated data
- **Error handling** for invalid characters
- **Use case specific naming** based on business context
