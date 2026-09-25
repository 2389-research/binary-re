# STIX/TAXII Threat Intelligence Formatting

If a binary is potentially malicious, format findings for sharing via STIX/TAXII:

```json
{
  "type": "malware",
  "spec_version": "2.1",
  "id": "malware--...",
  "name": "telemetry-client",
  "malware_types": ["spyware"],
  "capabilities": ["exfiltrates-data"],
  "implementation_languages": ["c"]
}
```
