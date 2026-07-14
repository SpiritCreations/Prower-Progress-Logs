#To-Do List

A running list of planned work, learning goals, and stretch projects for Prower. Completed items are marked and kept for reference.

---

## In Progress

- [ ] Finish configuring Prometheus custom metrics and alerting - including Discord alerts for critical infrastructure events
- [ ] Configure Node Exporter down alert rule in Prometheus
- [ ] Fix CEJ auto-start script - migrate to `screen` or `tmux` for reliable background process management

 ---

 ## Backlog 
 - [x] Finish Prometheus VM/CT monitoring - resolved 03-07-2026, see Incident-Reports.md
 - [x] Standardized postmortem format - completed, see POSTMORTEM.md and Incident-Reports.md
 - [ ] Bash script that auto-fills incident/postmortem templates?
 - [ ] Get first pull request accepted on an external repo
 - [ ] Look into the ins and outs of CDNs and learning basic concepts

 ---

 ## Stretch Goals

 - [x] Build local LLM on Metal-01 and experiment with self-hosted AI tooling - completed, 01-19-2026 found little utility but was a fun thing to do, is now deleted to dedicate server resources elsewhere.
 ~~ - [ ] Purchase enterprise level switch to build VLANs and get more familiar with switching CLI ~~ Purchased a managed switch on accident, but its fulfilling the role needed.
 - [ ] Build failure analysis and postmortem pipelines - automate as much as possible with scripts that push pre-filled templates
 - [ ] Auto shutdown script for power outage scenarios using NUT - monitoring systems shutdown last
 - [ ] Local physical seismic detector that can send emergency shutdown requests to Prower
~~ - [ ] Self-hosted cloud storage - experimental S3-style project ~~ Tried to get a solution working using a Raspberry pi 5 and nvme HAT, but wasn't stable and constantly ran into permission issues, repurposed into a jellyfin server. Will revive this project when storage costs go down or if I can find some certified enterprise drives of some sort.

---

## Notes
Completed items are kept here for reference rather than deleted. Dates and links to relevant files are added when items are resolved.
