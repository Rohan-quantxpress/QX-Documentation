## 1.1 High-Level Project Structure

The Process Manager project is organized into a small number of main components. Each component has a specific responsibility in the execution, configuration, logging, and monitoring of BOD and EOD processes.

At the root level, the project contains the following important files and folders:

```
project-root/
│
├── lib/
│   ├── config.py
│   ├── email.py
│   └── logger.py
│
├── utils/
│   ├── (BOD and EOD scripts)
│   └── ...
│
├── process_master.py
├── process.json
└── requirement.txt
lib/
```

The lib folder contains the common utility modules used by the Process Manager.

config.py – Contains configuration-related functionality used by the application.
email.py – Provides email-related functionality for sending notifications and alerts.
logger.py – Provides logging functionality used for recording process execution and application events.
utils/

The utils folder contains the individual scripts executed by the Process Manager.

These scripts can perform different BOD and EOD operations, such as downloading files, processing data, updating databases, performing backups, and other scheduled activities.

The scripts in this folder are executed according to their configuration in process.json.

process_master.py

process_master.py is the main Process Manager application.

It is responsible for:

Scheduling BOD and EOD processes
Executing configured Python scripts
Starting, stopping, and restarting configured processes
Monitoring configured processes
Updating process status and monitoring information in Redis
Handling process failures
Sending failure and process-down notifications

The Process Manager also supports monitoring and managing Windows executable processes and Docker containers.

process.json

process.json is the main configuration file for the Process Manager.

It defines:

BOD processes
EOD processes
Process schedules
Application paths
Process parameters
Active/inactive status
Process start, stop, and restart settings

The Process Manager reads this configuration to determine which processes need to run, when they should run, and how they should be managed.

requirement.txt

requirement.txt contains the Python dependencies required by the Process Manager and its supporting scripts.

These dependencies must be installed in the environment before running the Process Manager.

Overall Structure

The relationship between the main project components can be represented as:

                 process.json
                      │
                      ▼
              process_master.py
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        BOD          EOD       Monitoring
          │           │           │
          └───────┬───┘           │
                  ▼               ▼
               utils/          Redis / Alerts
                  │
                  ▼
            BOD & EOD Scripts
                  │
                  ▼
                 lib/
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     config.py email.py logger.py

This structure separates the process execution logic, process configuration, BOD/EOD scripts, and common utility functions, making the Process Manager easier to maintain and extend.