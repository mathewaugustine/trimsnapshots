# trimsnapshots
Trim snapshots in AWS periodically, leaving desired number of snaps untouched.

usage: trimsnapshots.py [-h] [-d RETENTION_DAYS] [-s SAFEMODE]
                        [-t SERVER_NAME] [-r REGION] [-c SNAP_NO_TO_KEEP]
                        

Trim AWS snapshots


optional arguments:
  -h, --help            show this help message and exit
  
  -d RETENTION_DAYS, --retention_days RETENTION_DAYS
                        retain snapshots newer than this value. defaults to 7
                        
  -s SAFEMODE, --safemode SAFEMODE
                        set the dry run flag
                        
  -t SERVER_NAME, --server_name SERVER_NAME
                        Please provide the data to populate ServerName tag
                        
  -r REGION, --region REGION
                        region where you want to manage snapshots
                        
  -c SNAP_NO_TO_KEEP, --snap_no_to_keep SNAP_NO_TO_KEEP
                        set number of snapshots to retain. defaults to 2
                        
suggested names for --server_name

LDAP

WIKI
