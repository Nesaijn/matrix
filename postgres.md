# Database Backup
```
su postgres
pg_dump synapse > synapse_$(date +'%Y-%m-%d_%H-%M-%S')_version.sql
```
