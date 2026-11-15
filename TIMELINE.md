# Successful-action timeline

| Order | Action | Result |
|---:|---|---|
| 1 | Nmap service discovery | Identified SSH and HTTP; application paths were revealed |
| 2 | Review `/data/` | Application database and JSON stores were exposed |
| 3 | Inspect the SQLite database | Plaintext administrator credential storage was confirmed |
| 4 | Review the `file` parameter | LFI enabled PHP source disclosure |
| 5 | Inspect administrative source | Unsafe `eval()` use in custom template processing was identified |
| 6 | Validate the quiz-description processing path | PHP execution was confirmed in the web-server context |
| 7 | Retrieve the challenge flag | Successful CTF completion |

This timeline is intentionally concise. It omits recovered secrets, session identifiers and reusable command-execution payloads.
