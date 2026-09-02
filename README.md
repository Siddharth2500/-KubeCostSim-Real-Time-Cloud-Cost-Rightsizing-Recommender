# 💰 KubeCostSim - Real-Time Cloud Cost & Rightsizing Recommender
Python · Pandas · NumPy · Matplotlib  

KubeCostSim is a Python simulator for **real-time cloud cost analysis** and **rightsizing recommendations**.  
It mimics tools like Kubecost or AWS Cost Explorer by simulating workload requests vs. actual usage, calculating waste, and suggesting optimal requests to reduce spend.  
Outputs include **CSV exports** and **charts** for quick analysis in Colab or locally.

--------------------------------------

## ✨ What It Does
- Simulates CPU/Memory **usage** for multiple workloads (steady, spiky, bursty)  
- Calculates **per-second cost** based on **requested** resources  
- Generates **rightsizing recommendations** using `avg`, `p90`, or `p95` usage  
- Exports **cost_samples.csv**, **rightsizing.csv**  
- Produces charts:  
  - **cost_timeseries.png** — cumulative cost proxy over time  
  - **cpu_requests usage.png**, **mem_requests_usage.png** — request vs usage  
  - **requests_vs_usage.png** — combined overview  

----------------------------

## 🛠 Tech & Languages

| Layer        | Tech         | Notes                                      |
|--------------|--------------|--------------------------------------------|
| Language     | Python 3.10+ | Clean dataclasses + simple OOP             |
| Data         | Pandas       | Aggregation, CSV export                    |
| Math/Random  | NumPy        | Resource fluctuation & percentiles         |
| Charts       | Matplotlib   | Cost and rightsizing visualizations        |
| Runtime      | Colab/Local  | Works in Google Colab or your machine      |

---------

## 🌐 Architecture

**Flow**
1. **Workloads** simulate CPU/Memory use around their requests, with profile-based variance.  
2. **CostEngine** samples at an interval and logs per-second cost (request-based).  
3. **Rightsizing** computes recommended requests using **headroom × (avg/p90/p95)**.  
4. Results are saved to **CSV** and **PNG** charts.

**Diagram**
┌────────────┐ requests + usage ┌───────────────┐
│ Workloads │ ───────────────────▶ │ Cost Engine │
│ (web/api) │ │ History/Cost │
└────────────┘ └──────┬────────┘
│
analyze│rightsizing
▼
┌────────────────────────┐
│ Rightsizing Recommender│
│ CSV + Savings per hour │
└─────────┬──────────────┘
│
┌──────▼────────┐
│ Visualization │
│ PNG charts │
└───────────────┘

yaml
Copy code

-----

## 📦 Repository Structure

kubecostsim/
├─ app/
│ ├─ workloads.py # Workload definition & simulation
│ ├─ cost_engine.py # Engine, pricing, rightsizing
├─ outputs/
│ ├─ cost_samples.csv # Raw usage & cost data
│ └─ rightsizing.csv # Recommendations with savings
├─ tests/
│ └─ test_costs.py # Basic cost/rightsize unit tests
├─ requirements.txt
└─ README.md

yaml
Copy code

> In Colab you can run everything in a single cell; the same files will be produced in the notebook’s working directory.

------

## ▶️ Run in Google Colab

Paste the **single code cell** into Colab and run.  
Then, in a new cell, explore outputs:

```python
import pandas as pd

print(pd.read_csv("cost_samples.csv").tail(6))
print(pd.read_csv("rightsizing.csv"))
Artifacts created:

cost_samples.csv

rightsizing.csv

cost_timeseries.png

cpu_requests_usage.png

mem_requests_usage.png

requests_vs_usage.png

🔗 Configuration
Inside the code, tweak these commonly changed settings:

Setting	Default	Meaning
duration	20s	Total simulation time
interval	1s	Sampling interval
policy	p90	Rightsizing basis: avg, p90, p95
headroom	1.2	Safety multiplier over usage
min_cpu	0.25	Floor for CPU recommendation (vCPU)
min_mem	0.25	Floor for Memory recommendation (GB)
price_cpu	0.031	$ per vCPU hour
price_mem	0.004	$ per GB hour

Workload profiles:

steady: low variance

spiky: occasional spikes

bursty: frequent spikes and wider variance

📊 Sample Outputs
cost_samples.csv

ts	workload	cpu_req	cpu_used	mem_req	mem_used	cost_per_sec
14:40:10	web-frontend	1.0	0.72	1.0	0.64	0.000010
14:40:10	api	1.5	0.95	2.0	1.41	0.000017

rightsizing.csv

workload	policy	headroom	cpu_req	cpu_reco	mem_req	mem_reco	estimated_savings_hr_$
web-frontend	p90	1.2	1.0	0.86	1.0	0.79	0.0031
api	p90	1.2	1.5	1.12	2.0	1.70	0.0054

Charts

cost_timeseries.png: cumulative cost (proxy) vs. time

cpu_requests_usage.png: CPU request vs avg vs p95 per workload

mem_requests_usage.png: Memory request vs avg vs p95 per workload

🧪 Tests
Example tests you can add:

CPU/Memory request floors are respected

Savings never negative

p95 ≥ p90 ≥ avg for the same workload

Run (if you add tests locally):

bash
Copy code
pytest -q
🐳 Docker (Optional)
Build and run:

bash
Copy code
docker build -t kubecostsim:latest .
docker run -it -v $PWD:/data kubecostsim:latest
Mount a volume to copy out CSV/PNG artifacts.

☸️ Kubernetes (Optional Extension)
Wrap engine with FastAPI/Flask endpoints: /simulate, /rightsizing

Save artifacts to a PVC or S3

Schedule periodic audits via CronJob

Emit Prometheus metrics (cost proxy, savings potential)

🔐 Production Notes
Pull real usage from Prometheus/CloudWatch

Use actual price catalogs (AWS/GCP/Azure) per region & instance class

Add SLO-aware policies (use p95/p99 and traffic windows)

Include business rules (min replicas, burst tolerance)
---------

## 👤 Author
Siddharth Raut — DevOps & Cloud Engineer
