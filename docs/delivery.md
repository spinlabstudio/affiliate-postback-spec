# Delivery and security

## Transport

- `POST` with a JSON body over HTTPS. Avoid GET postbacks with values in the query string: they end up in access logs and analytics.
- Respond `2xx` only after the event is stored. Anything else counts as a failure.

## Retries

- Retry with exponential backoff: 1 min, 5 min, 30 min, 2 h, 12 h, then park the event in a dead-letter queue and alert.
- The receiver deduplicates on `event_id`. That makes retries safe and lets you replay a whole day after an outage.

## Ordering

Do not assume events arrive in order. An `ftd` can arrive before its `registration` after a retry. Receivers should accept it and link by `click_id` and `player_ref`.

## Signing

Sign the raw body with HMAC-SHA256 and a per-tracker secret:

```python
import hmac, hashlib, time

def sign(secret: bytes, body: bytes) -> dict:
    ts = str(int(time.time()))
    mac = hmac.new(secret, ts.encode() + b"." + body, hashlib.sha256).hexdigest()
    return {"X-Postback-Timestamp": ts, "X-Postback-Signature": mac}

def verify(secret: bytes, body: bytes, ts: str, sig: str, max_age=300) -> bool:
    if abs(time.time() - int(ts)) > max_age:
        return False  # replay protection
    expected = hmac.new(secret, ts.encode() + b"." + body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, sig)
```

## Test mode

Give each affiliate a sandbox endpoint and a `test: true` flag. Fire one of each event type during onboarding and have both sides confirm receipt before real traffic starts. Most disputes are prevented in this ten-minute step.

## Privacy

Postbacks carry pseudonymous IDs and amounts only. No names, emails, IP addresses or document data. Document the data sharing in your privacy notice and in the affiliate agreement.
