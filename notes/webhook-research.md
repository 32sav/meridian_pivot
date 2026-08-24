# Webhook Verification Research

## 1. What is a webhook?

A webhook is a mechanism that allows one application to automatically send information to another application when a particular event occurs.

Instead of the receiving application continuously asking whether something has happened, the sending application sends an HTTP request to a specific endpoint when the event occurs.

For example, when an event happens in a system, the system can send a POST request to my application's webhook endpoint containing information about that event.

In simple terms:

Sender → HTTP request → Webhook endpoint → Application processes the event


## 2. Why does a webhook need verification?

A webhook endpoint is normally accessible through the network, so my application should not automatically trust every request that reaches it.

An unauthorized person could send a fake request to the endpoint and pretend to be the legitimate service.

For example, someone could send a fake request saying that a badge has been successfully printed even though the printer service never sent that message.

Webhook verification provides a way for the receiving application to check that the request was sent by the expected source and that the request data has not been changed.

Therefore, the application should verify the webhook before processing the event.


## 3. What is a webhook signature?

A webhook signature is a value generated from the webhook's contents using a secret shared between the sender and receiver.

The sender uses the secret and the request data with a cryptographic algorithm to produce the signature.

The signature is normally sent along with the webhook request, often in an HTTP header.

The receiving application can then independently calculate what the signature should be and compare it with the received signature.

If they match, the request can pass verification.


## 4. How does signature verification work?

The sender and receiver have access to the same secret.

The sender uses the secret and the webhook payload to generate a signature.

The sender sends the payload together with the signature to the receiving webhook endpoint.

The receiver obtains the original request body and the signature from the request.

The receiver uses the same secret and verification method to calculate its own expected signature.

The received signature and calculated signature are then compared.

The basic process is:

Sender:
Secret + Payload → Signature

Receiver:
Secret + Received Payload → Expected Signature

Then:

Expected Signature = Received Signature
        ↓
      Valid

Expected Signature ≠ Received Signature
        ↓
     Invalid


## 5. What happens when verification fails?

When verification fails, the application should not trust or process the webhook as a legitimate event.

The request should be rejected and the application should return an appropriate error response.

The application should also avoid performing the business operation associated with the webhook.

For example, if a webhook claims that a badge has been printed, the application should not mark the attendee as having a completed print job until the webhook has passed verification.


## 6. Technologies and libraries I may need

The prototype will use Python.

I may need:

- A Python web framework for receiving HTTP requests.
- A cryptographic/HMAC facility for calculating or checking signatures.
- HTTP request/response handling.
- A method for testing webhook requests.
- Python's testing tools for testing valid and invalid requests.

I will determine the exact libraries and implementation approach during my research and development.


## 7. Resources Consulted

| Resource | Date/Time | What I learned |
|---|---|---|
| GitHub Webhooks documentation | 23 August 2026 | Webhook deliveries can be validated using a signature generated with a secret. |
| GitHub Validating Webhook Deliveries | 23 August 2026 | The receiver can calculate a signature and compare it with the signature received in the request. |
| Stripe Webhook Documentation | 23 August 2026 | The original request body is important when verifying webhook signatures. |
| Python documentation | 23 August 2026 | Python provides tools that can be used for cryptographic and HTTP-related operations. |


## 8. Questions I Still Have

- How exactly is the webhook signature generated?
- Which HTTP header will contain the signature?
- How can I access the original/raw request body?
- Which Python library or built-in module should I use?
- How should the application compare the signatures safely?
- What HTTP response should be returned when verification fails?
- How can I test both valid and invalid webhook requests?


## 9. My Understanding

I understand webhook verification as a security step that happens before my application trusts an incoming webhook.

The sender creates a signature using a secret and information from the request. The receiving application uses the same secret and the received request data to calculate the expected signature. It then compares the expected signature with the signature received from the sender.

If the verification succeeds, the application can continue processing the webhook. If it fails, the application should reject the request and should not perform the action associated with it.

The important lesson I have learned is that receiving a request does not automatically mean that the request should be trusted.

## 10. Detailed Research Findings
 ### 10.1 Signature Generation

The prototype will use HMAC with SHA-256. The sender combines the shared secret with the exact request body to produce a signature. The receiver repeats the calculation using the same secret and body

### 10.2 Signature Header

X-Hub-Signature-256

### 10.3 Raw Request Body

The signature must be calculated against the original request body. We shouldn't parse the JSON first and then reconstruct it, because even small changes in formatting or representation can produce different data and therefore a different signature.

### 10.4 Python Tools

hmac
hashlib

### 10.5 Safe Signature Comparison

hmac.compare_digest()

### 10.6 Failed Verification

If the signature is missing or doesn't match the calculated signature, the application must reject the webhook and must not process the event

### 10.7 Testing Strategy

Correct signature → accepted
Incorrect signature → rejected
Missing signature → rejected
Modified request body → rejected

## 11. Prototype Design

 ### 11.1. Objective

The objective of the prototype is to demonstrate how a Python application can receive a webhook and verify its signature before trusting or processing the information. The prototype should accept legitimate webhook requests and reject requests that fail verification.

### Input

The prototype will receive an HTTP POST request containing a webhook payload. The request will also contain a signature in the `X-Hub-Signature-256` header. The application will use a shared secret to verify the signature.

### Verification Process

The application will first obtain the original request body and the signature from the request header. It will use the shared secret and HMAC-SHA256 to calculate the expected signature. The expected signature will then be safely compared with the received signature.

The webhook will only be considered valid when the signatures match.

### Valid Request

When the received signature matches the signature calculated by the application, the webhook will be accepted. The prototype will then process the event and return a successful response.

### Invalid Request

When the received signature does not match the calculated signature, the webhook will be rejected. The application must not process the event.

### Missing Signature

If the webhook does not contain the required signature header, the application will reject the request because it cannot verify where the request came from.

### Modified Payload

If the payload is changed after the signature was created, the calculated signature should no longer match the received signature. The prototype should therefore reject the modified request.

### Expected Responses

A correctly verified webhook should receive a successful response. A webhook with an invalid or missing signature should receive a rejection response. The response should make it clear whether the verification succeeded or failed.

### Test Cases

1. A valid webhook with the correct signature should be accepted.
2. A webhook with an incorrect signature should be rejected.
3. A webhook without a signature should be rejected.
4. A webhook whose payload has been modified should be rejected.