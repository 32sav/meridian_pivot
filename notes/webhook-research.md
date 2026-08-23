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