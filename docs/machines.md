# Machines

All on the 10.0.0.x network, on the Proxmox host, except Anubis.

| Machine | Address | Role | GPU | Runs |
|---|---|---|---|---|
| Proxmox host | 10.0.0.2 | the host | (none used) | the VMs; ZFS pools (`local-znas` and others); drive bays for up to 32 drives |
| Athenaeum | 10.0.0.20 | Athena | | Hermes (user `athena`): gateway, Desktop backend (`hermes serve`), her API server on :8642 |
| Ollama | 10.0.0.21 | her mind | Tesla V100 16 GB (a 32 GB V100 to come) | Ollama (her model, `gemma4-64k`, kept resident; embeddings) |
| Whisper | 10.0.0.22 | her ears | Quadro T1000 8 GB | faster-whisper in Docker, :9000 |
| Kokoro | 10.0.0.23 | her voice | Quadro P1000 4 GB | Kokoro in Docker, :8880 |
| Anubis | 10.1.0.98 | Kayla's workstation | | ComfyUI :8188 (her pictures, until the 32 GB card) and the fingerprint server :8189; Claude Code |

## Soma's agents

Each of the host and the three VMs runs `athena-status` (from [hermes-soma](https://github.com/Kaylachuuu/hermes-soma)
`agent/`): a read-only status service on port 8190 that answers only Athenaeum. Settings in
`/etc/athena-status.json`; the agent in `/opt/athena-status`. On the host it runs as root to read SMART; drives are
read in the background, eight at a time, never woken from standby.

To update one, from your user on Athenaeum, in your clone of hermes-soma:

```
git pull
scp -r agent USER@ADDRESS:soma-agent && ssh -t USER@ADDRESS sudo sh soma-agent/install.sh host|ollama|whisper|kokoro
```

## When the 32 GB V100 arrives

- Soma sees it as a second GPU on the Ollama VM by itself; her usual for that machine relearns over a few days.
- ComfyUI moves off Anubis onto it; point eikon's and holonomic's dream settings at the new address.
