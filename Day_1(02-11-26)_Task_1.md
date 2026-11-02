# Client-Side Response Manipulation Allows OTP Verification Bypass

## Target
`https://taskbear.com.au/`

## Technical Severity
**High**

## Vulnerability Details

### URL / Location
`POST /api/v1/uwa/auth/otp/verify`

### Overview
The OTP verification process relies on a client-controlled response. When an invalid OTP is submitted, the server returns `401 Unauthorized`. By intercepting and modifying this response to indicate successful verification, the application accepts the invalid OTP as valid.

The issue was reproduced in both the **email OTP** and **mobile OTP** verification flows.

### Steps to Reproduce

1. Navigate to the Taskbear signup page and enter a test email address.
2. Request an OTP and submit an invalid OTP.
3. Intercept the OTP verification response using Burp Suite.
4. Observe the legitimate server response:

```http
HTTP/2 401 Unauthorized
```

```json
{
  "statusCode": 401,
  "message": "OTP invalid or expired"
}
```

5. Modify the intercepted response to:

```http
HTTP/2 201 Created
```

```json
{
  "contact": "<redacted>",
  "verifiedAt": "<timestamp>"
}
```

6. Forward the modified response to the application.
7. The application displays a successful verification confirmation and proceeds with the registration/onboarding flow despite the invalid OTP.
8. The same behavior can be reproduced in the mobile OTP verification flow.

### Impact

An attacker can bypass the OTP verification step by manipulating the client-side response, without knowing the valid OTP.

This allows an attacker to proceed through account registration and onboarding despite submitting an invalid OTP.

### Evidence

The following screenshots demonstrate the issue:

- Invalid OTP submission.

<img width="1920" height="968" alt="1" src="https://github.com/user-attachments/assets/7b4b979b-90db-4cbd-91a0-277f0185bfe9" />

- Original `401 Unauthorized` response.

<img width="1918" height="872" alt="2" src="https://github.com/user-attachments/assets/0c851994-4410-49d8-982a-9a046eb8927e" />

- Manipulated `201 Created` response.

<img width="1920" height="873" alt="3" src="https://github.com/user-attachments/assets/9dc75b50-99c3-4693-ba0b-dc4257018ab3" />

- The same response manipulation against mobile OTP verification.

<img width="1920" height="872" alt="4" src="https://github.com/user-attachments/assets/93099cc8-964a-4878-af3b-d0b7826ede97" />

- Application proceeding to the onboarding flow after the manipulated response.

<img width="1920" height="973" alt="5" src="https://github.com/user-attachments/assets/a34fb149-9348-4b47-93df-b908d3e543f3" />

### Remediation

OTP verification should be enforced entirely on the server side. The application should maintain the verification state server-side and require successful server-side OTP validation before allowing registration or onboarding to proceed.

Client-side responses should never be treated as proof of successful OTP verification.
