# Sólarsteinn
Sólarsteinn is a framework for running a Valheim dedicated server on pre-emptible public cloud instances (AWS Spot). The word refers to the legendary sunstones, which the vikings supposedly used to find the sun through clouds while navigating under a completely overcast sky. 

## Goals: 
The goal is to get an affordable, automated, and stable dedicated server running on public clouds that competes in cost with dedicated server offerings from game companies. It's relatively easy to get a dedicated server set up on a unix host, but on-demand pricing for a capable enough server is high ($40+ a month). One way to get around that is to use spot instances, which vary in price based on bids on extra capacity. For a spot-based server, the server has to gracefully handle shutdowns and restarts.

This server should:
- Cost the same or less than equivalent dedicated servers from game hosting companies
- Stably support up to 10 players for a reasonable amount of playtime (~8h) per day
- Have some mechanism to backup data on a regular basis
- Require little to no no human interaction to maintain once set up
  - Update on a regular basis
  - Handle service interruptions gracefully (especially with spot/pre-emptible instances)
- Automate part-time uptime

## Current competing STC/STG offerings:
*TODO make this into a table*
- CitadelServers
- SecretServers
- GPortal
- GTXgaming

## Cost components
Actual cost to run the server is more than just server uptime itself.
- Compute instance uptime
- EBS (hard drive) capacity
  - Root volume
  - Detachable (persistent) game volume
- Snapshot (backup) storage
- Network throughput (data transfer)
- Elastic IP

## Instance types
Valheim dedicated server requires at least 2 CPUs and 4GB RAM. 

- Is a tX.medium sufficient?
  - With 4.42h of CPU burst credits per day, will we hit burst limit?
    - What is baseline CPU usage like? Some users report 100% utilization of a single core as baseline
  - definitely t3 over t2 for this region
  - If we upgrade, do we go for a tX.large (burst limited) or a cX.large (unlimited)
- How does resource impact scale with # of players?
- Hard drive space ($0.10/GB/mo 
  - 2GB baseline for server installation, how does this scale/grow with world exploration
  - How much breathing room for updates? Does steamcmd purge old files?
- Region
  - Which west coast region has the best spot pricing for relevant instances? 
  - How much does ping matter - do we go usw2 by default just to optimize connectivity?
- IP
  - Need a static IP attached to the server at each boot so that users don't have to rotate

![spot](images/ec2_spot_avgs_usw2_210212.jpg)



## Architecture
### Containerization vs. static volume
#### Containers
- May be annoying to update
- Carry additional performance/space overhead. 
- Easier to migrate to other services/hosts
- Easier to orchestrate through pre-emptions/downtime without custom devops scripting.

#### Static volume attached to instance
- Easy to attach/detach 
- Easy to snapshot for backups
- Tied to AWS, harder to migrate to a new server

### Control scheme

#### Orchestration/configuration

Ansible?

#### Necessary scripts

- From controller
  - Start instance
  - Stop instance
  - Attach and mount EBS
- On server
  - Initial installation helpers (optional)
    - Install server components
    - Install world from existing save
    - Swap between multiple worlds? (bonus feature - config file?)
  - Start server
  - Save world
  - Take down server
  - Update server
  - Wrapper: save world, take down (pre-emption response)
  - Wrapper: save world, take down, update, start (update)
