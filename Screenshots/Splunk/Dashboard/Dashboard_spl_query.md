## 1) SPL query for attack_timeline using line chart
```
index=* "sshd" "Failed password"
| timechart count
```

## 2) SPL query for top_attacking_source_ips using bar chart
```
index=* "sshd" "Failed password"
| top limit=10 src_ip
```

## 3) SPL query for authentication_status_by_source_ip using stacked column chart
```
index=* "sshd" ("Failed password" OR "Accepted password") 
| eval Status=if(searchmatch("Failed password"), "Failed Attempts", "Successful Logins") 
| chart count over src_ip by Status
```
