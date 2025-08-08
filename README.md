# swift-weights
Speeding up the loading of ML models by efficiently splitting the weights across multiple disks then reading them in parallel during model initialization

## Motivation

M.2 drives now read at ~7.5GB/s or so today. If you were running 4 of them in RAID you could probably achieve 30GB/s. But what if you *don't* have access to 4 m.2 slots?

The answer is a "methodology" I'm calling swift-weights.

Requirements:
- At least 2 Thunderbolt ports (ideally TB4/5)
- As many m.2 drives as there are TB ports

Execution:
- Split the weights of a model (safetensors) into `N` folders such that if we have 4 drives and 8 weights:
```
- drive_0
  - model-00001-of-000008.safetensors
  - model-00002-of-000008.safetensors
- drive_1
  - model-00003-of-000008.safetensors
  - model-00004-of-000008.safetensors
- drive_2
  - model-00005-of-000008.safetensors
  - model-00006-of-000008.safetensors
- drive_2
  - model-00007-of-000008.safetensors
  - model-00008-of-000008.safetensors
```
Then by mounting and loading all m.2 drives using an interface like this [40Gbps Docking Station](https://www.amazon.com/dp/B0CRHHL6WY), you should be able to read them in parallel and achieve a hypothetical 160Gbps (20GB/s) *without needing to modify the existing storage in your system*.

This is especially relvent for systems like the M3 ultra which have 6 TB ports each. So if you were trying to load in large models like Kimi-K2, it (hypothetically) can have a cold-start of <30 seconds if scaled properly.

This repo contains helpers for splitting of your weights and loading them into memory in parallel.
