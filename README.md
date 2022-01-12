# Farm

Farm out your bare metal at the edge. 

![image](https://user-images.githubusercontent.com/3028982/125092283-2f204d00-e09f-11eb-8fcd-56cad02fe429.png)

## Changelog
v0.0.4 - January 11th, 2022
```
Added compute GPU support
Added trex for eth ( ETHADDR=0x.. )
```

v0.0.3 - December 31st, 2021
```
Added video transcode and live streaming
```

## Future Development
  - [X] Auto Initialization
  - [X] Host System cap grep
  - [X] Podman rootless
  - [X] Video Transcode
  - [X] T-Rex Ethereum Miner by default waiting for jobs
  - [ ] Live Streaming resolution fixes
  - [ ] Live Streaming javascript player
  - [ ] Live Streaming feed distribution for high viewer streams
  - [ ] Speed Test
  - [ ] Networking DHCP
  - [ ] Networking Static IP
  - [ ] Offchain SEV attestor
  - [ ] Offchain attestation queue
  - [ ] SGX podman
  
## Status
  - [X] docker (podman) jobs can be created.
  - [ ] encrypted_vm (AMD SEV) jobs are not supported yet.
  - [ ] encrypted_container (SGX) jobs are not supported yet.

## Introduction
Farm allows you to harvest yields from your metal, whether you have a few systems lying around untapped or operate at fullscale.

It does this by giving you a price for CPU, Ram, Disk and GPU from users on the Zod EDGE.

Think of running your farm as connecting your metal to a giant supercompute cluster, where you get a yield for liquidity that that metal provides.

Farm has 2 modes of operation; confidential and plaintext.
  - AMD EPYC provides confidential VMs 🔒
  - Intel SGX confidential mode is not ready yet 🕑
  - All other processors provide plaintext sandbox mode 📄

The value of your farm is dependent upon:
  - Total uplink bandwith (1Gbe recommended) (many states now offer 10Gbe fiber for 200$ a month https://www.utopiafiber.com/10g-info/)
  - CPU Type (EPYCs 7xx3 Milan most valuable ATM, this will level out as more CPUs support confidential compute)
  - Total CPUs and Ram
  - Total GPUs (for jobs that require GPUs)
  - Only NVIDIA Gpu is supported.
  - Location Location Location (GPUs at the edge in South Korea are more valuable then those same GPUs in Kyrgyzstan)

## How to join the edge
Visit https://transcode.zod.tv/clusterfarm and click `View Farm Key` to get your current NEAR access key secret_key + your near account set as envvars.  You need to login with your NEAR wallet at transcode.zod.tv to be issued a NEAR access key, allowance on NEAR access keys is 0.25 NEAR, so if this key were to get compromised, you can lose at most 0.25 NEAR. If you lose the key you should revoke it at wallet.near.org.

## System Prep
To provide NVIDIA Gpus, you need NVIDIA drivers + CUDA + nvidia-smi.  
  
To provide transcode jobs, all included bits are in the farm binary.  

To provide live streaming, all included bits are in the farm binary.  

To provide rootless containers (docker basically) you need to have podman, slirp4netns, fuse-overlayfs installed.  

You also need to allow cgroupsv2 to set a cpu quota, by default this is not enabled on every distro.
```
sudo mkdir -p /etc/systemd/system/user@.service.d/
echo -e '[Service]\\nDelegate=memory pids cpu cpuset io' | sudo tee -a /etc/systemd/system/user@.service.d/delegate.conf
//and reboot
```
The AUTO=true envvar will make farm complete the setup steps automatically but only on Ubuntu 22.04.

## Environmental Variables

```
//Near
NEAR_PKEY - Your NEAR access key secret_key
NEAR_ACCOUNT - Your NEAR account

//Farm
WORKFOLDER - Path to store misc items, disk images and non podman storage (default ~/.cache/zod)
AUTO - Auto setup your system if using Ubuntu 22.04+ true | false | null (default null)
LOG_LEVEL - How verbose to log 1 | 2 | 3 (default 1)

//Video
VIDEOFOLDER - Path to store video transcode jobs (default /tmp/vid)
MAX_CPU_BUSY - Max cpu usage % before transcode jobs are refused (default 80)
MAX_TRANSCODE_JOBS - Max concurrent transcode jobs (default 8)
MAX_TRANSCODE_JOBS_PER_GPU - Max concurrent transcode jobs per GPU (default 2)

//Video Live (optionally turn your node public facing for live streaming)
//Your LIVE_HOST must be accessible over the internet on port 443, 
//but you can place cloudflare proxy mode infront and only expose 
//port 80 for simplicity. Cloudflare proxy mode will handle the SSL
//automatically via LetsEncrypt.
LIVE_HOST - DNS name to access your farm publicly (default null)
LIVE_PORT - Port (default 80)
LIVE_SSL_PORT - SSL Port (default 443)
LIVE_SSL_CERT - SSL Cert valid for DNS of LIVE_HOST (default null)
LIVE_SSL_KEY - SSL Key for SSL Cert (default null)

//Flags
COMPUTE - Allow compute jobs (default true)
TRANSCODE - Allow transcode jobs (default true)
LIVE - Allow live jobs if LIVE_HOST is set (default true)

//Eth
ETHADDR - Idle GPUs will mine eth using trex.
