# Inscribe-test-part-2

Customer Engineer Exercise Part 2

## Introduction

This a guide on how to get started with the Inscribe AI API for document fraud detection using Python

## Prerequisites

Python 3.9+

Install python dependencies

```
python -m pip install requests
```

## 1. Authentication and Credentials

In order to use the API you will need to obtain your unique API key which is like a password that will be used to authenticate all of your requests.

### How to obtain an API Key:

1. Login to your [account](https://app.inscribe.ai/#/auth/login?redirectUrl=%23/customers)

   > If you do not have an account please [reach out](https://www.inscribe.ai/demo-request) to get set up.

2. Go to [Settings -> API Keys](https://app.inscribe.ai/#/settings/api)

3. Generate your first secret token using the `Create API Key` button.
4. Copy and save it immediately - you will not be able to view it later.
5. You can create as many secret tokens as you need. All of your created API keys are viewable on the [Settings -> API Keys](https://app.inscribe.ai/#/settings/api) page but you won't be able to reveal the values of these secret tokens.

### Best Practices for Storing and Using API Keys

Use Environment Variables: Store your token in a `.env` file, not directly in your code.

1. In the root folder of your project, create a file called `.env`

2. Inside the `.env` file, add your generated Inscribe API secret token and name it 'API_KEY'

```
API_KEY=INSCRIBE_API_SECRET_TOKEN_GOES_HERE
```

3. Install `python-dotenv`:

```
pip install python-dotenv

```

4. Load the `.env` into your project

```
import requests
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv("API_KEY")

headers = {
    "accept": "application/json",
    "Authorization": f"Inscribe {api_key}"
}


response = requests.get("https://api.inscribe.ai/api/v2/customers", headers=headers)

print(response.text)

```

## 2. Making the First Request

To begin working with the Inscribe API, your first step is to create a customer folder by making a POST request. Customer folders act as containers for document sets, and each new customer you evaluate should have their own folder.

The request must include:

- The proper **headers** (e.g., authorization, content type)

- A **body** with the customer details in JSON format

**Required Headers**:
| Header | Description |
|----------------|-----------------------------------------|
| `accept` | `application/json` |
| `content-type` | `application/json`|
| `Authorization` | `Inscribe API Key` |

**Body Parameters**:

| Field           | Description                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------- |
| `name`          | A unique name given to the customer by you.(required)                                                               |
| `approved`      | Set the value true if you have approved the customer, false if you have rejected the customer. The default is null. |
| `notes`         | Additional notes that you want to record about this customer.                                                       |
| `persons`       | List of persons associated with the customer record                                                                 |
| `company`       | The company associated with the customer record                                                                     |
| `verify_entity` | Entities that are requested to be verified.                                                                         |

### Example POST request to Create a Customer:

The code below shows how to create a customer with the following header and body parameters filled in

```
import requests
from dotenv import load_dotenv

# import the API Key from the .env file
load_dotenv()
api_key = os.getenv("API_KEY")

url = "https://api.inscribe.ai/api/v2/customers"


# Example of Customer body parameters
payload = {
    "company": { "name": "Acme Inc" },
    "verify_entity": { "income": {
            "personal_income": {
                "lower_income_tolerance": 0.1,
                "under_reported_tolerance": 0.1
            },
            "revenue": { "lower_income_tolerance": 0.1 }
        } },
    "name": "Frank Abagnale",
    "approved": True,
    "notes": "John is a new customer",
    "persons": [
        {
            "address": {
                "line_1": "1000 Walnut St",
                "line_2": "Kansas City",
                "city": "Kansas",
                "post_code": "64106",
                "state": "MO",
                "country": "USA"
            },
            "first_name": "John",
            "last_name": "Smith",
            "date_of_birth": "1994-12-31T00:00:00.000Z",
            "phone_number": "+353 85 123 4567",
            "id_number": "078-05-1120"
        }
    ]
}
# Define the required headers

headers = {
    "accept": "application/json",
    "content-type": "application/json",
    "Authorization": f"Inscribe {api_key}"
}

response = requests.post(url, json=payload, headers=headers)

print(response.text)


```

## 3. Uploading Documents & Checking Fraud

A document object is a core component of Inscribe’s product. A document is typically a PDF or image that can be uploaded and associated with a customer object. When documents are uploaded, they are automatically analyzed by Inscribe's fraud detection and credit analysis systems.

To analyze documents for fraud, upload them to an existing customer folder using the `/customers/{id}/documents` endpoint. Once uploaded, the document(s) are queued for processing by Inscribe’s fraud detection, parsing, and optionally verification algorithms.

### Supported Documents:

Inscribe fraud detection has coverage over all document types and languages

- Max file size: 50MB

- Max pages (PDFs): 350

- Supported file types:
  ` application/pdf`, `image/jpg`, `image/png`, `application/octet-stream`

### Upload a Document:

Follow these steps to upload a document to Inscribe for analysis:

1. **Get the Customer ID**

   You’ll need the `customer id` of the customer you’re uploading the document for. This is typically returned when you create a customer object.

2. **Prepare the Endpoint**

   Use the following endpoint to upload your document:

```
https://api.inscribe.ai/api/v2/customers/{customer_id}/documents
```

Replace `{customer_id}` with the actual `customer id`.

3. **Prepare Body Parameters**:

| Field             | Description                                                                                   |
| ----------------- | --------------------------------------------------------------------------------------------- |
| `file   `         | A list of files to be uploaded for analysis.                                                  |
| `verify_entities` | Each verify_entity object in this array corresponds to a file in the above "files" parameter. |
| `tags`            | Additional notes that you want to record about this customer.                                 |

**Tags:**

When uploading a document, there is an option to assign an associated tag. Tags are string values chosen by the user and can be used to group documents together across customers.

**Verification Checks:**

Inscribe can run verification checks against a document. Verification checks can be added in the document upload request. The supported document verification requests are:

- Name
- Address
- ID Number
- Company

Inscribe also supports the customisation of verification checks. Customized verification checks must be included under the strings field.

### Example POST Request to Upload a Document:

Below is an example of how to upload a document for customer Jane Smith (customer_id = 40), where `frank_abagnale.pdf` is the document to be analyzed.

```
import requests
import os
import json

customer_id = "40"
url = f"https://api.inscribe.ai/api/v2/customers/{customer_id}/documents"
headers = {
    "Authorization": "Inscribe YOUR_API_KEY"
}

verify_entities = [{
    "name": "Jane Smith",
    "address": "1000 Walnut Kansas City MO",
    "company": "Smith & Sons",
    "id_number": "92914567",
    "strings": [{"key": "phone", "input": "929-7113-145"}]
}]

tags = [{"text": "onboarding"}, {"text": "new_customer"}]

# File path to file named 'frank_abagnale.pdf'
DOCUMENT_PATH = "./frank_abagnale.pdf"


with open(DOCUMENT_PATH, "rb") as document:
    filename = os.path.basename(document.name)
    files = {
        "verify_entities": (None, json.dumps(verify_entities), 'application/json'),
        "tags": (None, json.dumps(tags), 'application/json'),
        "file": (filename, document, "application/octet-stream")
    }


    response = requests.post(url, headers=headers, files=files)

    print(response.text)

```

### How to handle Successful Response vs Common Errors

Below is an example of a successful response with a status code of **202**. This indicates that the 'Bank Statement' (`john_smith_payslip_dec20.pdf`) document has been successfully processed for the customer named John Smith (`customer_id = 42`).

It also shows that the document has been flagged as fraudulent, with a **trust score of 50**, a **quality score of 50** and **review status of ACCEPTED**

```
{
  "data": [
    {
      "uuid": "6e764ab5-4617-40d5-9076-0bd0e537b4d1",
      "id": "42",
      "name": "john_smith_payslip_dec20.pdf",
      "created_at": "2019-12-12T15:35:08.460Z",
      "customer_id": "684502aa-268a-4c2a-b166-ac4bfa2caf44",
      "customer_name": "John Smith",
      "state": "PROCESSED",
      "total_pages": 1,
      "document_type": "BANK_STATEMENT",
      "subtypes": [
        "CP_575"
      ],
      "file_type": "TRUE_PDF",
      "language": "en",
      "is_fraudulent": true,
      "trust_score": 50,
      "quality_score": 50,
      "review_status": "ACCEPTED",
      "urls": {
        "web_app": "https://app.inscribe.ai/#/customer/0efae611-b3eb-4b84-8ed0-a4e92ad3bf4b/document/1234",
        "api": "https://api.inscribe.ai/api/v2/customers/0efae611-b3eb-4b84-8ed0-a4e92ad3bf4b/documents/1234",
        "original_file": "https://production-us-west-2-assets.inscribeusercontent.com/files/users/123/documents/8a2d3b0c-27be-4513-91ab-52c6e0a19691_document.png"
      }
    }
  ]
}

```

### Important fields:

The table below highlights key fields from the response that provide valuable insight into the document's validation and analysis. You'll also find links to relevant documentation on the Inscribe AI website for further reference.

| Field         | Description                                                                                                                                                                                                                                                 |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| is_fraudulent | Has the value true if Inscribe’s fraud detection algorithms have identified the document as being fraudulent, false if sufficient evidence of fraud is not found in the document. This value is only available when the document is in the PROCESSED state. |
| trust_score   | The trust score of a document, between 0 and 100. More information can be found at https://help.inscribe.ai/en/articles/5557654-trust-score.                                                                                                                |
| quality_score | The quality score of a document, between 0 and 100. More information can be found at https://help.inscribe.ai/en/articles/5567690-quality-score.                                                                                                            |
| review_status | The current status of document level feedback. More information can be found at https://help.inscribe.ai/en/articles/8523290-document-level-feedback.                                                                                                       |

**The `state` field is critical for determining the current status of a document and whether you can rely on its analysis results.**

| State                    | Description                                                                   |
| ------------------------ | ----------------------------------------------------------------------------- |
| `PROCESSING`             | This document is still being processed.                                       |
| `PROCESSED`              | This document has finished processing and results are available.              |
| `ERROR_PROCESSING`       | An error occurred while processing this document.                             |
| `VIRUS_DETECTED`         | This document was found to contain a virus and will not be processed.         |
| `CORRUPT_DATA_DETECTED`  | This document was found to contain corrupted data and could not be processed. |
| `PASSWORD_PROTECTED`     | This document is password protected.                                          |
| `EXCEEDS_MAX_PAGE_COUNT` | This document exceeds the maximum page count of 350.                          |

### Common Errors:

Status codes **400**, **403**, **404**, and **429** indicate a faulted state and are not considered successful responses.  
Be sure to review the returned error message to help troubleshoot the issue effectively.

| Status Code | Description                                                                                                                                                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **400**     | **Bad Request** – You have provided incorrect or malformed data.                                                                                                                                                                     |
| **403**     | **Forbidden** – You do not have permission to access this resource.                                                                                                                                                                  |
| **429**     | **Too Many Requests** – You have exceeded the request limit of 200 requests per 5 minutes. Wait at least 30 seconds before retrying. <br> [More info on rate limiting](https://docs.inscribe.ai/docs/are-the-endpoints-rate-limited) |

## 4. Next Steps

### Possible Follow-Up Endpoints or Advanced Features

In a production integration, webhooks can be used to be notified and act on a document that has finished processing.

Implement webhooks to receive notifications about important events in real-time, such as document state changes, customer approval status changes, or completion of a collect session.

The webhook feature will allow seamless integration with external systems, ensuring that updates are processed immediately without manual intervention.

### Best Practices for Scaling or Automating the Process in Production

- Concurrency : Ensure that webhook payloads are handled concurrently without blocking other system processes. Use asynchronous methods and worker queues to manage high-volume notifications efficiently.

- Retries: In the event of a failure or an unreachable URL, implement a retry mechanism with exponential backoff to handle temporary outages or delays in processing.

**To Setup Webhooks check out this guide**: [Getting started with Webhooks](https://docs.inscribe.ai/docs/getting-started-with-inscribe-webhooks)

## 5. Security Considerations

### How Inscribe handles data security ?

In addition to being SOC II and ISO27001 certified, we store and encrypt all data securely in our VPC, and we never send PII to models that we don’t host ourselves.

### Guidance for encryption in transit and at rest?

## 6. Troubleshooting & Support

### Common issues (invalid tokens, 4xx/5xx errors, rate limits)

1. **Invalid Tokens:**
   An invalid token means the API cannot verify your application's identity or permissions.

   **Causes:** Expired tokens, incorrect tokens, or tokens revoked by Inscribe AI.

   **Troubleshooting:**

   Verify the token is valid and hasn't expired.
   Make sure you are using the correct token for your specific Inscribe AI account and application.
   If the token is expired, obtain a new token through [Settings -> API Keys](https://app.inscribe.ai/#/settings/api)

2. **4xx Errors:**
   These errors indicate that the request is not valid or cannot be fulfilled due to client-side issues.

   **400 Bad Request:** The request body is invalid or the request is not in the correct format.

   **401 Unauthorized:** The token is invalid or missing.

   **403 Forbidden:** The application does not have permission to access the resource.

   **404 Not Found:** The requested resource (API endpoint) does not exist.

   **429 Too Many Requests:** You have exceeded the rate limit for your account.

   **Troubleshooting:**

   Check the [API documentation](https://docs.inscribe.ai/reference/overview) to ensure the request parameters and format are correct.

   Verify the authorization token is valid and included in the request.

   If you're exceeding the rate limit, reduce the number of requests or wait for the rate limit to reset.

3. **5xx Errors:**

   These errors indicate that the server has encountered an issue while processing the request.

   **500 Internal Server Error:** A general server error.

   **503 Service Unavailable:** The server is temporarily unavailable due to overload or maintenance.

   **Troubleshooting:**

   Retry the request after a short delay, as the server might be experiencing temporary issues.

   If the issue persists, contact Inscribe AI support by emailing intercom-support@inscribe.ai.

   Implement exponential backoff to handle potential server-side issues.

4. **Rate Limits:**
   Rate limits are restrictions on the number of requests an application can make within a given time period.

   **Causes:** Excessive API usage, potential misuse, or protecting Inscribe AI's infrastructure.

   **Troubleshooting:**

   Reduce the number of requests, especially if you're exceeding the limit.

   Implement retry logic with exponential backoff to handle 429 errors.

   Monitor your API usage to identify and address potential bottlenecks.

### Where to reach out

Customer may initiate a helpdesk ticket any time by emailing intercom-support@inscribe.ai.

There is also a contact form found here: https://www.inscribe.ai/contact-us
