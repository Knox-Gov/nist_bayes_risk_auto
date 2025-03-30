# NIST 800-53 Bayesian Risk Analyzer

This application automates the Bayesian risk quantification for NIST 800-53 Revision 5 controls using AWS Bedrock LLMs, incorporating an iterative QA refinement process.

## Overview

The application reads a list of NIST 800-53 R5 controls from an input CSV file. For each control, it uses a primary AWS Bedrock LLM to generate a Bayesian risk analysis. A second AWS Bedrock LLM performs a QA check. If the QA check fails, the critique from the QA LLM is fed back to the primary LLM to generate a revised analysis. This process repeats up to a configurable maximum number of attempts. The final analysis for every control (whether QA-approved or failed after max attempts) is written to an output CSV file, along with QA status details.

## Requirements

- Python 3.11 or higher
- AWS account with access to AWS Bedrock
- AWS credentials configured (via `~/.aws/credentials`, environment variables, or IAM role)
- Permissions to invoke Bedrock models (`bedrock:InvokeModel`)

## Installation

### 1. Clone the repository

```bash
git clone <repository-url>
cd nist_bayes_risk_auto
```

### 2. Create a virtual environment

```bash
# For macOS/Linux
python3.11 -m venv venv
source venv/bin/activate

# For Windows
python3.11 -m venv venv
venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

## Input File Format

The application expects a CSV file with the following columns:
- `Family`: The NIST control family (e.g., AC, AU, CM)
- `ID`: The control ID (e.g., AC-1, AC-2)
- `Control Name`: The name of the control
- `NIST Control Description`: The full description of the control

A sample `controls.csv` file is provided with the application.

## Usage

```bash
python nist_bayes_analyzer.py \
    --primary-model-id <primary-model-id> \
    --qa-model-id <qa-model-id> \
    [--input-file controls.csv] \
    [--output-file nist_bayes_analysis_results.csv] \
    [--log-file analysis.log] \
    [--max-attempts 5] \
    [--control-delay 1.0] \
    [--api-call-delay 0.5] \
    [--log-level INFO]
```

### Required Arguments

- `--primary-model-id`: AWS Bedrock model ID for the primary analysis LLM (e.g., `anthropic.claude-3-sonnet-20240229-v1:0`)
- `--qa-model-id`: AWS Bedrock model ID for the QA LLM (e.g., `anthropic.claude-3-haiku-20240307-v1:0`)

### Optional Arguments

- `--input-file`: Path to the input CSV file containing NIST controls (default: `controls.csv`)
- `--output-file`: Path to the output CSV file for all analyses (default: `nist_bayes_analysis_results.csv`)
- `--log-file`: Path to the log file (default: `analysis.log`)
- `--max-attempts`: Maximum number of refinement attempts per control if QA fails (default: `5`)
- `--control-delay`: Delay in seconds between processing controls to avoid rate limiting (default: `0.0`)
- `--api-call-delay`: Delay in seconds between API calls to avoid rate limiting (default: `0.0`)
- `--log-level`: Set the logging level (choices: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`; default: `INFO`)

## Example

```bash
python nist_bayes_analyzer.py \
    --primary-model-id us.anthropic.claude-3-opus-20240229-v1:0 \
    --qa-model-id us.anthropic.claude-3-7-sonnet-20250219-v1:0 \
    --control-delay 1.0 \
    --api-call-delay 0.5
```

## Handling Rate Limits

If you encounter throttling errors from AWS Bedrock, try the following:

1. Increase the `--control-delay` and `--api-call-delay` values
2. The application includes built-in exponential backoff and retry logic for throttling errors
3. If persistent issues occur, consider requesting a rate limit increase from AWS

## Output

The application generates a CSV file with the following columns:

- Basic control information (Family, ID, Control Name)
- Risk scenario details
- Bayesian probability components (Prior, Evidence, Likelihood)
- Calculated posterior probability
- QA status and feedback
- Number of attempts
- Notes/assumptions

## Logging

Detailed logs are written to the specified log file. The log includes:

- Information about each control processing attempt
- API call details
- Error messages and warnings
- Final analysis results

## Troubleshooting

- **AWS Credentials**: Ensure your AWS credentials are properly configured
- **Rate Limiting**: If you encounter throttling errors, increase the delay parameters
- **Model Availability**: Verify the specified model IDs are available in your AWS Bedrock account
- **Input File Format**: Ensure your input CSV has the required columns
