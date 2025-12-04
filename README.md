# Wyze Cam V3 Security Analysis

A comprehensive security analysis framework for the Wyze Cam v3, developed as part of a thesis project focused on defining a verification framework for Cyber Resilience Act (CRA) compliance.

## Overview

This repository contains the tools, scripts, and documentation from a security research project that:

1. **Develops a verification framework** to transform CRA regulatory requirements into concrete, repeatable testing practices
2. **Validates the framework** through practical security analysis of the Wyze Cam v3, a popular consumer IoT device

The Wyze Cam v3 was selected as a case study due to its widespread adoption, affordable price point (€30-90), and diverse feature set including 1080p video, motion/sound detection, local microSD storage, and cloud services.

## Key Findings

### Network Security
- **No exposed ports with standard firmware**: Full TCP/UDP scans (65,535 ports) revealed no open services
- **Client-only architecture**: The device operates as a pure client, communicating only with Wyze cloud services
- **Strong TLS implementation**: Uses TLS 1.2/1.3 with robust cipher suites (TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256)
- **Proprietary video streaming protocol**: UDP-based, non-standard streaming that doesn't match RTP specifications

### RTSP Firmware Analysis
When the optional RTSP firmware (4.61.0.3) is installed:
- Port 554 (RTSP) is exposed
- Uses LIVE555 Streaming Media library (with known CVE vulnerabilities)
- Digest authentication is required for all requests
- Path traversal attacks were unsuccessful due to proper input validation

### Rate Limiting
- Wyze API implements effective rate limiting
- Triggers HTTP 429 after approximately 20-22 consecutive failed requests
- Resets after approximately one minute
- Protects against brute-force and DoS attacks

### MITM Attack Resistance
- SSL/TLS certificate validation is properly implemented
- Self-signed certificates are rejected (TLS Alert 48: "unknown_ca")
- DNS spoofing attempts were intercepted but TLS validation prevented exploitation

### Firmware Vulnerabilities
Analysis of firmware version 4.36.14.3497 revealed:
- **Root credentials**: SHA-512 hashed password in `/etc/shadow` (resistant to dictionary attacks)
- **Hardcoded Wi-Fi credentials**: Factory Wi-Fi configuration in `wpa.conf`
- **Exposed Wyze endpoints**: Production and beta API endpoints visible in configuration files
- **Factory mode vulnerability**: Potential code execution via `Test.tar` on microSD
- **Outdated components**: Linux kernel 3.10.14 with numerous known CVEs
- **CVE analysis results**:
  - AppFS: 12 Critical, 569 High, 1475 Medium, 35 Low
  - RootFS: 4 Critical, 13 High, 18 Medium, 1 Low

## Repository Contents

### Scripts

| Script | Description |
|--------|-------------|
| `script jago/wyze_attack_rtsp.sh` | RTSP port scanning using nmap with rtsp-url-brute script |
| `script jago/injection.sh` | Path traversal and header injection tests |
| `script jago/fuzz_rtsp_advanced.sh` | Advanced RTSP fuzzing with digest authentication |
| `script jago/injection_nocredentials.sh` | Injection tests without authentication |
| `script jago/rate_limiting_test.py` | Wyze API rate limiting verification |
| `script jago/wyze_nbstat.sh` | NetBIOS status check |
| `script jago/wyze_smb2sec.sh` | SMB2 security testing |

### Documentation

- `thesis.pdf` - Complete thesis document with detailed methodology and findings
- `FCC Report/` - FCC certification documentation for the Wyze Cam v3

## Test Environment Setup

### Requirements
- macOS or Linux system with Wi-Fi capability
- Wireshark for packet analysis
- nmap for port scanning
- Python 3 with `requests` library
- OpenSSL for certificate generation
- netcat (nc) for raw network communication
- bettercap (optional, for MITM testing)

### Network Configuration
1. Create a Wi-Fi hotspot on your analysis machine (2.4 GHz, WPA2)
2. Connect the Wyze Cam to the hotspot
3. Verify DHCP lease assignment (macOS: `/var/db/dhcpd_leases`)
4. The camera typically receives IP `192.168.2.2`

### Running Tests

**RTSP Attack (requires RTSP firmware):**
```bash
cd "script jago"
./wyze_attack_rtsp.sh
```

**Injection Tests:**
```bash
cd "script jago"
./injection.sh
```

**Rate Limiting Test:**
```bash
cd "script jago"
python3 rate_limiting_test.py
```

## CRA Compliance Summary

| Requirement | Status | Notes |
|-------------|--------|-------|
| Network security (1.1.2b) | Partial | No open ports, encrypted communications |
| Update process (1.1.2c) | Compliant | HTTPS delivery, user notification |
| Encryption (1.1.2e) | Partial | TLS implemented, key management unknown |
| DoS protection (1.1.2h) | Partial | Rate limiting present, no CAPTCHA |
| SBOM availability (1.2.1) | Non-compliant | No public SBOM available |
| Support period disclosure (1.2.2) | Partial | 1-year policy, no release frequency disclosed |

## Tools Used

- **Wireshark** - Network protocol analyzer
- **nmap** - Network scanner with NSE scripts
- **Binwalk** - Firmware extraction and analysis
- **sasquatch** - SquashFS extraction
- **cve-bin-tool** (Intel) - CVE vulnerability scanning
- **John the Ripper** - Password hash cracking
- **bettercap** - MITM and network attacks
- **OpenSSL** - Certificate generation and TLS testing

## References

- [Cyber Resilience Act (EU) 2024/2847](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R2847)
- [Wyze Cam v3 Product Page](https://www.wyze.com/products/wyze-cam-v3)
- [Wyze Firmware Downloads](https://support.wyze.com/hc/en-us/articles/360024852172)
- [FCC ID: 2AUIUWYZEC3F](https://fcc.report/FCC-ID/2AUIUWYZEC3F/)
- [LIVE555 CVE Database](https://app.opencve.io/cve/?vendor=live555&product=streamingmedia)

## Disclaimer

This research was conducted for educational and academic purposes only. All testing was performed on devices owned by the researcher in a controlled, isolated environment. The techniques described should only be used for authorized security testing and research.

## Authors

- Jago Revrenna - Security Research & Thesis Development
- Based on prior work by Matteo Mariotti, Alessandro Brighenti, and Simone Rossi

## License

This project is for educational and research purposes. See the thesis document for complete methodology and citations.