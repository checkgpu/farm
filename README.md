# Farm

Farm out your bare metal at the edge. 

![image](https://user-images.githubusercontent.com/3028982/125092283-2f204d00-e09f-11eb-8fcd-56cad02fe429.png)

## Changelog
v0.0.7 - February 9rd, 2022
```
IPFS support
```

v0.0.6 - February 3rd, 2022
```
Added better NVENC support detection
Fix cgroupsv2 detection
Fix Tesla GPU detection
Fix estimated price calculation
Add GPU price table support
Add Podman Args
Require kernel 5.11 now
```

v0.0.5 - January 12th, 2022
```
Added auto update support
```

v0.0.4 - January 11th, 2022
```
Added compute GPU support
Added trex for eth ( ETHADDR=0x.. )
```

v0.0.3 - December 31st, 2021
```
Added video transcode and live streaming
```

## Introduction to Compute
The Zod Farm runs any code that can be containerized. You are not restricted to running WASM binaries or other derivatives. Using GPUs you have the following caps `NVIDIA_DRIVER_CAPABILITIES=utility,compute,graphics,video`, let us know if your usecase requires the `display` cap.
  
## Features
  - [X] IPFS (sandboxed API exposed on http://127.0.0.1:50001/ inside container)
  - [X] docker containers /w GPU (podman)
  - [X] transcode video (ffmpeg)
  - [X] live streaming (basic features)
  - [ ] live streaming recording
  - [ ] encrypted_vm (AMD SEV) jobs are not supported yet.
  - [ ] encrypted_container (SGX) jobs are not supported yet.

## IPFS
IPFS is exposed inside the container as a unix domain socket at `/ipfs`.  
```
#Download a file from IPFS
curl -s --unix-socket /ipfs -X POST \
  "http://dontcare/api/v0/cat?arg=QmYhEsVWdzbCU49XE8F9ybKKAPQHjhfhekoMn4Xvz857rD" --output source.blend

#Upload a file to IPFS and get gateway URL
curl -s --unix-socket /ipfs -X POST \
  http://dontcare/api/v0/add -F 'file=@out.png' | jq -r '.Hash' | awk '{print "https://cloudflare-ipfs.com/ipfs/"$1}'
```

## How to run jobs on the edge
Visit https://github.com/zodtv/examples to see examples of running workloads like blender, machine learning, and more.  

## How to join the edge
Visit https://transcode.zod.tv/clusterfarm and click `View Farm Key` to get your current NEAR access key secret_key + your near account set as envvars.  You need to login with your NEAR wallet at transcode.zod.tv to be issued a NEAR access key, allowance on NEAR access keys is 0.25 NEAR, so if this key were to get compromised, you can lose at most 0.25 NEAR. If you lose the key you should revoke it at wallet.near.org.

## System Prep
Visit https://github.com/zodtv/farm_image specifically `Installing ontop of existing Ubuntu` section.

## Environmental Variables

```
//Near
NEAR_PKEY - Your NEAR access key secret_key
NEAR_ACCOUNT - Your NEAR account

//Farm
WORKFOLDER - Path to store misc items, disk images and non podman storage (default ~/.cache/zod)
LOG_LEVEL - How verbose to log 1 | 2 | 3 (default 1)

//IPFS
IPFS_IPPORT - Location of IPFS API (default http://127.0.0.1:5001)

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
IPFS - Allow IPFS interaction (default true)

//Auto Update
UPDATE - Set the autoupdate mode. (default pull)
  "none" means ignore auto updates.
  "pull" downloads+overrides latest farm without restarting. 
  "exit" exits farm after doing pull.

//Eth (optional)
ETHADDR - Idle GPUs will mine eth using trex. (mutually exclusive with MINERCMD)
MINERCMD - Full commandline of your custom miner (must support specifying GPUs by index) 
  '/usr/bin/minerbz -w 0x0 -p ethstratum+tcp://eth.flexpool.io:4444 stratum+tcp://usw-eth.hiveon.net:4444 -r name'
MINERCMDGPUINDEX - Commandline option to select miners by GPU index (default is "-g") 
  (make sure not to set -g on MINERCMD)
```

## Misc
Farm allows you to harvest yields from your metal, whether you have a few systems lying around untapped or operate at fullscale.

It does this by giving you a price for CPU, Ram, Disk and GPU from users on the Zod EDGE.

Think of running your farm as connecting your metal to a giant supercompute cluster, where you get a yield for compute.

Farm has 2 modes of operation; confidential and plaintext.
  - AMD EPYC provides confidential VMs 🔒 (not ready)
  - Intel SGX confidential mode for containers 🕑 (not ready)
  - All other processors provide plaintext sandbox mode 📄

The value of your farm is dependent upon:
  - Total uplink bandwith (1Gbe recommended) (many states now offer 10Gbe fiber for 200$ a month https://www.utopiafiber.com/10g-info/)
  - CPU Type (EPYCs 7xx3 Milan or Modern XEONs will be more valuable later due to confidential compute support)
  - Total CPUs and Ram
  - Total NVIDIA GPUs
  - Only NVIDIA GPU is supported.
  - Location Location Location (GPUs at the edge in South Korea are more valuable then those same GPUs in Kyrgyzstan)
