# Extension Checkout Status Guard

## Bug

Extension checkout allowed for COMPLETED/CANCELLED sessions.

## Fix

```python
ALLOWED_STATUSES = {"active", "extended"}

async def checkout_extension(session_id, data):
    session = await get_session(session_id)
    
    if session.status not in ALLOWED_STATUSES:
        raise HTTPException(409, f"Cannot extend {session.status} session")
    
    if data["duration_minutes"] <= 0:
        raise HTTPException(400, "Duration must be positive")
    
    max_ext = get_max_extension(session.tier)
    total = session.total_extension + data["duration_minutes"]
    if total > max_ext:
        raise HTTPException(400, f"Max extension ({max_ext} min) exceeded")
    
    return await create_extension(session_id, data)
```

## Tests

```python
def test_completed_rejected():
    session = create_session(status="completed")
    resp = client.post("/api/session-extensions/checkout", json={"session_id": session.id, "duration": 30})
    assert resp.status_code == 409

def test_active_accepted():
    session = create_session(status="active")
    resp = client.post("/api/session-extensions/checkout", json={"session_id": session.id, "duration": 30})
    assert resp.status_code == 200
```
