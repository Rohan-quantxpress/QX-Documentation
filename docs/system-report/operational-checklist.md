# Operational Checklist

## Before Running

- [ ] Verify `generate_report.py` exists.
- [ ] Verify `process_config.json` exists.
- [ ] Verify configured process names are correct.
- [ ] Verify Docker CLI is available when Docker components are configured.
- [ ] Verify the executing user has required Docker permissions.

## During Execution

- [ ] Configuration is loaded.
- [ ] Operating system is detected.
- [ ] Configured processes are evaluated.
- [ ] Docker containers are checked.
- [ ] Native processes are checked.
- [ ] HTTP checks are performed where configured.
- [ ] CPU and memory are collected.
- [ ] Process groups are evaluated.
- [ ] Host resources are collected.
- [ ] Markdown report is generated.
- [ ] HTML report is generated.
- [ ] JSON report is generated when requested.

## After Execution

- [ ] Verify `report.md`.
- [ ] Verify `report.html`.
- [ ] Review overall health status.
- [ ] Review critical processes.
- [ ] Review down processes.
- [ ] Review HTTP failures.
- [ ] Review impacted groups.
- [ ] Review host resource usage.
- [ ] Review troubleshooting FAQ.
- [ ] Verify `status.json` if JSON output was requested.

## Quick Health Review

Check the report in the following order:

1. Overall status
2. Critical processes
3. Down processes
4. HTTP health checks
5. Impacted groups
6. CPU and memory
7. Host resources
8. Troubleshooting FAQ