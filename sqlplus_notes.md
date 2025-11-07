# Oracle SQL*Plus Development Notes

Copy the whole block below and paste into your sqlplus_notes.md (or create a new oracle_apps_deploy.md) and then run the git commands at the end to commit & push.

# Oracle Apps — Deploy & Run Custom Concurrent Program (PO)

**Goal:** Deploy a local SQL file to the application server, register a concurrent program in Oracle EBS (PO), submit it and verify success.

---

## 1. Prepare the SQL file locally
1. Create your SQL script locally (example name): `custom_report.sql`.
2. Test it locally (if possible) against a development database before deployment.

---

## 2. Transfer the file to the application server
1. Open **WinSCP** (or use `scp` from terminal).
2. Connect to the app server with the application OS account (the account used for deployments).
3. Upload the file into the custom location for PO programs.  
   - Example target folder (replace with your environment path):  
     `/apps/<appl_top>/custom/PO/`  
   - If using command line (example):  
     ```bash
     scp custom_report.sql user@server:/apps/<appl_top>/custom/PO/
     ```
4. Set file permissions so the application user can execute/read it (if required):
   ```bash
   ssh user@server
   chmod 750 /apps/<appl_top>/custom/PO/custom_report.sql
   chown applmgr:applmgr /apps/<appl_top>/custom/PO/custom_report.sql


Note: Replace <appl_top>, user, and applmgr with your real values.

3. Create security objects (group & responsibility)

In Oracle EBS (System Administrator responsibility) create a Request Group:

System Administrator → Concurrent → Program → Define Request Groups (or appropriate menu)

Create a new request group: e.g., PO_GROUP_MAHESH

Create a Responsibility and assign it to a user:

System Administrator → Security → Responsibility → Define

Create responsibility name: PO_RESP_MAHESH (display name as needed)

Associate the request group with the responsibility:

System Administrator → Concurrent → Program → Request Groups → attach the program to the request group

Then attach the request group to the responsibility (so users with that responsibility can submit the program)

4. Register the concurrent program (executable + program)

Register the executable:

System Administrator → Concurrent → Program → Executable

Create an executable record:

Executable Name: CUSTOM_PO_REPORT_EXE

Execution Method: Host (or appropriate)

Application Short Name: PO

Execution File Name: full path to your script or wrapper (e.g., /apps/<appl_top>/custom/PO/custom_report.sql or a shell wrapper that invokes sqlplus)

Create / Register the concurrent program:

System Administrator → Concurrent → Program → Define

Program Name: CUSTOM PO REPORT

Short Name: CUST_PO_RPT

Associate with Executable: CUSTOM_PO_REPORT_EXE

Enter other attributes (output format, parameters) as needed

Add the program to the Request Group (created in step 3).

Tip: Many shops use a shell wrapper that calls sqlplus with the script, and register the wrapper as the executable. This helps with environment variables and logging.

5. Submit the concurrent request

Log in with a user who has PO_RESP_MAHESH responsibility.

Navigate to: View -> Requests -> Submit Request (Submit Requests screen).

Select Program: CUSTOM PO REPORT.

Supply parameter values (if any) and submit.

Note the request id shown (e.g., 12345).

6. Monitor and verify

In View Requests or Monitor Requests, find the request id and watch status:

Statuses: Pending → Running → Completed / Error

When completed, view Log and Output files from the request details:

Open the log first to verify steps executed and to see messages printed by the script.

If errors exist, the log usually contains ORA- errors or environment failures.

If the log shows success, confirm expected data/results (for example, check target tables or output file).

Checklist to verify success

Request completed successfully (not Error).

Log file shows script executed start → end with no ORA- errors.

Output file (if produced) has expected content.

If script modifies DB data, verify via SELECT queries in SQL*Plus or SQL Developer.

7. Troubleshooting tips

If the request fails with Permission denied or command not found:

Check executable path and OS permissions for the script.

Ensure environment variables and database connect string are correct in wrapper script.

If the request fails with ORA- errors:

Inspect the log for exact ORA code and line number; fix the SQL and re-deploy.

Use WHENEVER SQLERROR EXIT SQL.SQLCODE in scripts to make failures explicit in logs.

Keep a copy of the wrapper and SQL in version control (this repo) before and after deployment.
