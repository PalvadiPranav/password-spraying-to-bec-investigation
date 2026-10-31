# CLD-IR-0001: Microsoft 365 Account Takeover (Simulated SOC Report)

My incident report for the [MyFirstHack](https://myfirstcyberjob.com) Cloudora CEO Account Takeover exercise.
Cloudora is fictional; all identities, IPs and logs are synthetic.

## Contents
- [`CLD-IR-0001_Cloudora_M365_Account_Takeover.pdf`](CLD-IR-0001_Cloudora_M365_Account_Takeover.pdf): my full report
- [`resources/`](resources/): the sign-in and audit logs plus the KQL query pack, so you can run the investigation and write your own report

## Scenario
Password spraying against 20+ Microsoft 365 accounts led to two confirmed compromises. The attacker registered a rogue MFA device ("Pixel 6") and created a hidden inbox rule ("RSS Subscriptions") to conceal finance emails, which are signs of Business Email Compromise preparation.

## MITRE ATT&CK
T1110.003, T1078, T1098.005, T1564.008

## Author
P.S.V.S. Pranav, CEH v13
