#To-Do List

A running list of planned work, learning goals, and stretch projects for Prower. Completed items are marked and kept for reference.

---

## In Progress

- [ ] Finish configuring Prometheus custom metrics and alerting - including Discord alerts for critical infrastructure events
- [ ] Configure Node Exporter down alert rule in Prometheus
- [ ] Fix CEJ auto-start script - migrate to `screen` or `tmux` for reliable background process management
- [ ] Experiment with Linux performance tuning in server environments

 ---

 ## Backlog 
 - [x] Finish Prometheus VM/CT monitoring - resolved 03-07-2026, see Incident-Reports.md
 - [x] Standardized postmortem format - completed, see POSTMORTEM.md and Incident-Reports.md
 - [ ] Learn basic bash scripting - starting with auto-filling incident/postmortem templates?
 - [ ] Get first pull request accepted on an external repo
 - [ ] Look into MC server client resyncing
 - [ ] Look into the ins and outs of CDNs and learning basic concepts

 ---

 ## Stretch Goals

 - [x] Build local LLM on Metal-01 and experiment with self-hosted AI tooling - completed, 01-19-2026 found little utility but was a fun thing to do, is now deleted to dedicate server resources elsewhere.
 - [ ] Purchase enterprise level switch to build VLANs and get more familiar with switching CLI
 - [ ] Build failure analysis and postmortem pipelines - automate as much as possible with scripts that push pre-filled templates
 - [ ] Auto shutdown script for power outage scenarios - monitoring systems shutdown last
 - [ ] Local physical seismic detector that can send emergency shutdown requests to Prower
 - [ ] Self-hosted cloud storage - experimental S3-style project

---

## Notes
Completed items are kept here for reference rather than deleted. Dates and links to relevant files are added when items are resolved.
