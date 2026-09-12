# Commit Brief 01: Honeypot Decoy Telemetry & Forensic Models

* **Feature**: Autonomous Multi-Port Honeypot Decoy Engine (Feature 1, Step 1)
* **Author**: Harkirat Singh (`harkiratsingh96kk@gmail.com`)
* **Target Files**: `honeypot/__init__.py`, `honeypot/models.py`
* **Commit Message**: `feat(honeypot): implement DecoySession and TelemetryEvent forensic models`

---

## 1. Code Added & Context

In this step, we establish the **forensic telemetry data contracts** before opening any raw decoy sockets to attackers:

1. **`DecoySession` Dataclass**:
   * Tracks an adversary's complete lifecycle across any decoy service (SSH `2222`, HTTP `8080`, MySQL `3306`, Telnet `2323`).
   * Captures the network tuple: `source_ip`, `source_port`, `destination_port`, `protocol`, and active deception `persona`.
   * Records behavioral metrics: `bytes_in`, `bytes_out`, `interactions`, and live `risk_score`.
   * Categorizes adversary intent: `Reconnaissance`, `Brute Force`, `Exploitation`, or `Lateral Movement`.

2. **`TelemetryEvent` Dataclass**:
   * Captures individual granular interactions (e.g. single command entered, HTTP request header, payload buffer).
   * Directional tracking: `inbound` (attacker to decoy) vs `outbound` (deception response to attacker).

3. **Cryptographic Payload Hashing (`content_digest`)**:
   * Automatically computes the SHA-256 digest of every inbound payload upon arrival.
   * Guarantees an immutable forensic chain of custody for all evidence presented to the SOC team.

---

## 2. Why We Wrote It This Way (Technical Rationale)

* **Type Safety & Zero Data Loss**: Rather than using unstructured Python dictionaries (which cause silent `KeyError` crashes in high-throughput socket loops), dataclasses enforce strict schemas.
* **Seamless SQLite & AI Integration**: The `.to_dict()` methods cleanly serialize state for both the SQLite local database and our ChromaDB RAG vector embeddings.
* **Standards Compliant**: Meets digital forensics standards by hashing all attacker keystrokes/payloads at the instant of capture.

---

## 3. Judge Q&A Cheat-Sheet (Exact Speaking Points)

### 🎙️ Q1: *"What did you build in this commit?"*
> **Your Answer**:  
> *"Sir, before opening raw decoy sockets to attackers, we designed a strict forensic data model. In `honeypot/models.py`, we created `DecoySession` and `TelemetryEvent`. Whenever an adversary connects to any decoy port (like our SSH or HTTP services), their session is tracked with byte counters, risk scores, and automatic SHA-256 payload hashing. This ensures every piece of evidence captured has an immutable cryptographic chain of custody before entering our AI copilot."*

### 🎙️ Q2: *"Why not just write string logs to a file or use dictionaries?"*
> **Your Answer**:  
> *"String logs require expensive regex parsing later, and raw dictionaries don't give you type safety. With dataclasses, our socket workers can concurrently stream structured JSON events to both our SQLite database and WebSocket clients with zero parsing overhead."*

### 🎙️ Q3: *"How does this connect to your AI RAG system?"*
> **Your Answer**:  
> *"Every `TelemetryEvent` has an intent field and raw payload. Our RAG engine pulls these structured events, vectorizes them with sentence-transformers, and matches them against MITRE ATT&CK techniques in ChromaDB."*
