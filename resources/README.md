# Resources for the Cloudora CEO Account Takeover exercise

Use these to investigate the incident and write your own report.

| File | Contents |
|---|---|
| `cloudora_signin_logs.csv` | Synthetic Microsoft 365 sign-in logs (table `CloudoraSignIn_CL`) |
| `cloudora_audit_logs.csv` | Synthetic unified audit logs (table `CloudoraAudit_CL`) |
| `kql_query_pack.txt` | KQL queries to run against both tables |

## How to load
1. Open [Azure Data Explorer](https://dataexplorer.azure.com) (free cluster).
2. Ingest each CSV and name the tables exactly `CloudoraSignIn_CL` and `CloudoraAudit_CL`.
3. At ingestion set `TimeGenerated` = datetime and `ResultType` = **string**, otherwise the failed-login queries return nothing.
4. Run the queries from `kql_query_pack.txt`.

All data is synthetic. Exercise by MyFirstHack (myfirstcyberjob community).
