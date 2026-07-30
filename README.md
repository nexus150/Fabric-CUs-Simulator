# User Manual: Consumption and Throttling Simulator
**English** | [Spanish](README.es.md)

## 1. Introduction and Warnings
Welcome to the Microsoft Fabric CUs Consumption Simulator. This is an educational and collaborative tool (open source on GitHub) designed to help you deeply understand the mechanics of consumption, smoothing, and throttling on the platform.

> [!WARNING]
> Use exclusively for educational purposes to understand the concepts and mechanisms, not for exact production capacity planning. The calculations are approximations based on publicly available documentation.

## 2. Initial Setup: Language and Formats
![Costs Panel](assets/language.png)

Before starting the simulation, go to the top bar. Here you can change the interface language (English or Spanish) and configure how you prefer to display numbers and dates throughout the simulation.

> [!IMPORTANT]
> It is vital to define these formats **before starting the clock**, since changing the language or format in the middle of a simulation will trigger a complete reset to ensure data consistency.

## 3. Real-Time Cost Monitoring
![Costs Panel](assets/money.png)

In the top right corner, there is a real-time financial panel. As time advances, these counters will accumulate the financial cost of the simulation. You will see:
- The cost covered by your capacity **Reservations** (usually cheaper).
- The cost charged for excess usage or **PAYG** (Pay As You Go).
- Different hypothetical scenarios (what would have happened if everything was PAYG or everything was Reserved).

## 4. Infrastructure and Reservations
![Infrastructure Panel](assets/infra_panel.png)

The left panel, **1. Infrastructure**, is the foundation of the simulator:
- **Pricing (PAYG and Reserved)**: Define how much an F2 costs per hour (both PAYG and reservation) to project billing.
- **Purchased Capacity (SKU)**: Choose your base capacity size (e.g., F8). The simulator will automatically multiply the F2 costs by the necessary factor.
- **Time Horizon**: Define how far into the future the simulator should run (e.g., 72 hours).
- **Add Reservations**: Here you can purchase CU reservations. For example, on an F8 capacity, you could have 2 reservations of 2 CUs each, effectively covering half of its CUs. This will drastically reduce your final bill compared to using 100% PAYG.

## 5. Job Injection and Smoothing
![Job Injection Panel](assets/job_injection.png)

The **3. Job Injection** panel allows you to simulate real platform activity. Jobs in Fabric do not consume everything at once; instead, their consumption is "smoothed" (distributed) over time.
1. **Background Type**: Background jobs, such as training a massive model or refreshing a data warehouse (ETL), are strictly smoothed over **24 hours (2880 Timepoints)**.
2. **Interactive Type**: Immediate queries, such as clicking on a Power BI dashboard, are smoothed in much shorter bursts. In the simulator, you can choose a smoothing period ranging **from 5 minutes up to 64 minutes**, depending on the workload type (as established by Microsoft's smoothing documentation).

It is important to highlight that here we will load completed jobs, for which we already know their cost and how many TPs they are smoothed over. In Fabric, running jobs can also report consumption, and the total CUs are updated once the job finishes, but for the purpose of this simulation, it is understood that we are loading completed jobs with a defined CU cost.

The jobs can be seen in the jobs panel right below the simulation chart. The jobs displayed will be those of the selected timepoint.

We can create new jobs with the simulation running or paused.

If we load jobs before starting the simulation, we will see them in the jobs panel just below the chart. Once the simulation starts, the displayed jobs are those corresponding to the Selected TP.

![Jobs Panel](assets/jobs.png)

## 6. Clock and Speed Control
![Time Control Panel](assets/time_control.png)

The **2. Time Control** panel is the engine of the simulation.
- **Speed Slider**: Allows you to accelerate the passage of time. You can go from 1 Timepoint per second (slow and detailed) up to 60 Timepoints per second to observe the evolution from a bird's-eye view.
- Use the "Start Clock" / "Pause Clock" buttons to start and stop the chart's progress.
- You can stop the chart whenever you want to analyze a single timepoint or the entire simulation in peace.

## 7. The Chart and Navigation
![Chart Controls](assets/chart_controls.png)
![Running Chart](assets/chart_running.gif)
The main canvas shows the constant battle between your jobs and your capacity:
- **Dotted Red Line**: This is your maximum purchased capacity.
- **Blue Layer**: Background Jobs.
- **Red Layer**: Interactive Jobs.
If the stacked jobs exceed the red line, the **Overage Bucket** comes into play. The Fabric platform does not reject jobs immediately; instead, it accumulates that excess as "debt". When consumption falls below the red line, that leftover margin will be used to "burn down" and pay off the accumulated debt.

To select a timepoint, simply click on it directly on the chart at any height; this will display the list of active jobs at that TP as well as the detailed timepoint analysis panels. (Keep in mind that active jobs are those for which consumption is being paid at that TP, not necessarily running jobs, which fall outside the scope of this simulator).

You can interact with the buttons located above the chart:
- **Min / Max**: View the entire simulation or zoom in on the current point in time.
- **Sel TP**: Hyper-detailed zoom on the Selected Timepoint.
- **<- / ->**: Pan the viewport backward or forward in time by hours.
- **Live View**: Restore live tracking.
- **Pause Capacity**: The emergency red button that stops the calculation engine in time to inject all pending consumption at once.

## 8. Timepoint Audit and Analysis (Timepoint Analysis)
![Audit Panel](assets/audit_panel.png)
If you click on any timepoint on the chart, you will be able to analyze that specific clicked Timepoint.
At the bottom of the screen, three panels will appear showing a very detailed audit:
- The **Timepoint Analysis** details exactly what happened in that specific timepoint.
- It shows you the consumption projections for the next 10 minutes, 60 minutes, and 24 hours (vital for calculating throttling).
- The left panel (Previous TP) and right panel (Next TP) allow you to compare the selected timepoint with the previous and next ones (if any).
- You will also see the list of active operations during that timepoint.

## 9. Pause Capacity: Debt Clearing
![Chart showing Throttling](assets/pause_spike.gif)
If you click on **Pause Capacity**, the simulator will stop all jobs. At that moment, a remarkable event happens in Fabric capacity: **any pending accumulated overage and all smoothed consumption projected for the future are instantly imputed into the present, right at the timepoint where capacity is paused**. This generates a colossal spike in the Overage Bucket. Once capacity resumes, the cluster will be debt-free.

## 10. Throttling States
![Chart showing Throttling](assets/Throttling.png)
The system constantly monitors future debt predictions (at 10 min, 60 min, and 24 hours). If these predictions exceed 100% of the available capacity, the status badge will change from **Healthy** to warning colors:
1. **Interactive Delay (Yellow)**: Your interactive operations will wait 20 seconds before starting.
2. **Interactive Rejection (Orange)**: If consumption for the next 10 minutes is exceeded, interactive requests will be rejected.
3. **Background Rejection (Red)**: Upon exceeding 24 hours of future consumption, the capacity rejects new background operations (in the case of this simulator, it simply prevents you from loading more operations). In Fabric, when reaching this state, running background operations are not stopped but rather finish and add more debt; however, this is outside the scope of this simulator.

## 11. Hourly Billing
![Billing Table](assets/hourly_billing.png)
As the clock crosses the `:00` minute of every hour, a record is generated in the bottom left table. You will be able to expand each hour (by clicking the blue "+" icon) to granularly audit exactly which Timepoints were charged via Reservation and which incurred excess charges (PAYG).

## 12. Event Log
![Event Log](assets/event_log_new.png)
In the bottom right, the **Event Log** keeps a chronological trace of all system events. If the system repeatedly enters and exits Throttling (due to debt oscillation), it will be recorded here with its exact timestamp.

## 13. Active Reservations
![Active Reservations](assets/reservation.png)

Just above the event log, the **Active Reservations** panel shows the reservations we currently have active, including the time they were created. From this panel, you can also delete a specific reservation if desired.

## 14. Export and Import
![Export Import](assets/export_import.png)

If you have managed to simulate a fascinating (or catastrophic) use case, you can use the **Export** button in the control panel. This will download the entire state of the simulation as a JSON file, allowing you to share it with colleagues or load it later using **Import** to continue the analysis.


## Component Summary
1. **Initial Setup: Language and Formats**.
2. **Real-Time Cost Monitoring**.
3. **Infrastructure Panel (pricing and capacity selector)**.
4. **Add Reservations**.
5. **Time Control**.
6. **Start Clock**.
7. **Reset Simulation**.
8. **Export Simulation**.
9. **Import Simulation**.
10. **Job Injection**.
11. **Capacity Status Indicator at Current Timepoint**.
12. **Return to Live View**.
13. **Chart Zoom Controls**.
14. **Pause Capacity**.
15. **Chart Body, Consumption, and Throttling**.
16. **List of Active Jobs at Selected Timepoint**.

![Main P1](assets/main_p1_with_bullets.png)

17. **Current Timepoint Analysis**.
18. **Previous Timepoint Analysis**.
19. **Next Timepoint Analysis**.
20. **Hourly Billing Details and Consumption per Timepoint**.
21. **List of Active Reservations at Current Timepoint**.
22. **Event Log**..

![Main P2](assets/main_p2_with_bullets.png)
