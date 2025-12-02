# notify-africa-go

**Golang client for [notify.africa](https://notify.africa): Seamlessly integrate SMS and Email communications in your Go applications.**

---

## Overview

`notify-africa-go` is an easy-to-use Go SDK for sending SMS and Email messages via the [notify.africa](https://notify.africa) platform. It provides simple interfaces for authentication, configuration, and sending bulk or individual messages.

---

## Features

- **Send SMS & Email**: Programmatically send messages to recipients or lists.
- **Batch & Single Messaging**: Dispatch single or batch messages efficiently.
- **Error Handling**: Structured error reporting for easy troubleshooting.
- **Environment-based Config**: Use environment variables for security & flexibility.
- **Extensible**: Customize request URLs for testing/staging/production.

---

## Installation

Install via `go get`:

```bash
go get github.com/tacherasasi/notify-africa-go
```

---

## Configuration

Set required credentials and URLs as environment variables:

```bash
export SMS_APIKEY=your_sms_api_key              # Required for SMS
export SMS_SENDER_ID=your_sender_id             # SMS sender name/ID
export SMS_URL=https://api.notify.africa/api/v1/api/messages/batch   # Optional: override default SMS URL
export EMAIL_APIKEY=your_email_api_key          # Required for Email
```

Alternatively, you can pass API keys directly when creating clients (not recommended for production).

---

## Quick Start Example

Here’s a ready-to-run SMS sending example:

```go
package main

import (
	"context"
	"log"
	"os"
	"time"
	"github.com/tacherasasi/notify-africa-go/client"
)

func main() {
	// Load configuration from environment
	cfg := client.Config{
		SMSApiKey:   os.Getenv("SMS_APIKEY"),
		EmailApiKey: os.Getenv("EMAIL_APIKEY"),
	}
	c := client.NewClient(cfg)

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	// Optionally override SMS base URL (removing API endpoint)
	if smsURL := os.Getenv("SMS_URL"); smsURL != "" {
		base := smsURL[:len(smsURL)-len("/api/v1/api/messages/batch")]
		c.SMS.SetBaseURL(base)
	}

	senderID := os.Getenv("SMS_SENDER_ID")
	to := []string{"255686477074"} // Recipient phone(s)

	batchResp, err := c.SMS.SendBatchSMS(ctx, to, "Test SMS from Notify Africa Go!", senderID)
	if err != nil {
		log.Fatalf("Test SMS error: %v", err)
	}
	log.Printf(
		"Test SMS sent! Status: %d, Message: %s, Sent: %d, Credits: %d, Balance: %d",
		batchResp.Status, batchResp.Message, batchResp.Data.MessageCount,
		batchResp.Data.CreditsDeducted, batchResp.Data.Balance,
	)
}
```

---

## Direct Package Usage

For advanced or custom use cases, import individual SMS and Email packages:

```go
import (
	"os"
	"github.com/tacherasasi/notify-africa-go/sms"
	"github.com/tacherasasi/notify-africa-go/email"
)

// SMS Setup
smsClient := sms.NewClient(os.Getenv("SMS_APIKEY"))
smsClient.SetBaseURL("https://api.notify.africa") // Optional custom base URL
// smsClient.SendBatchSMS(...) or smsClient.SendSingleSMS(...)

// Email Setup
emailClient := email.NewClient(os.Getenv("EMAIL_APIKEY"))
emailClient.SetBaseURL("https://api.notify.africa/v2") // Optional custom base URL
// emailClient.Send(...)
```

---

## API Reference

### Client Initialization

- `client.NewClient(config)`  
  Returns a new Client instance.  
  - `Config`: Provide SMSApiKey and EmailApiKey fields.

### SMS Methods

- `SendBatchSMS(ctx, recipients, message, senderID)`  
  Send a message to multiple phone numbers.
- `SendSingleSMS(ctx, recipient, message, senderID)`  
  Send a message to a single phone number.

### Email Methods

- `Send(ctx, recipients, subject, body)`  
  Send an email to one or more recipients.

### Optional: SetBaseURL

- For testing, staging, or regional endpoints.

```go
smsClient.SetBaseURL("https://api.notify.africa")
emailClient.SetBaseURL("https://api.notify.africa/v2")
```

---

## Best Practices

- **Store API keys in environment variables**, not in code, to keep them secure.
- Use contexts (`context.Context`) for timeouts/cancellations when making network requests.
- Log/send errors to a monitoring system for production deployments.

---

## Troubleshooting

- **Invalid API Key?** Double-check your API key values and .env file or CI secret storage.
- **Endpoint errors?** Make sure the base URLs match your intended region or environment.
- **Rate limits?** APIs may have rate limits; consult [notify.africa docs](https://notify.africa) for limits & policies.

---

## Contributing

Pull requests, feature requests, and feedback are welcome!  
Please open an issue or PR and follow the [Contributor Guidelines](CONTRIBUTING.md) if available.

---

## License

This project is licensed under the [MIT License](LICENSE).  
Feel free to use it in commercial and non-commercial projects.
