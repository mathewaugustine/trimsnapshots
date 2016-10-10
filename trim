import argparse
import sys
import operator
from boto.ec2 import connect_to_region
from datetime import datetime, timedelta


# Before running script
# Run 'aws configure' on command line and setup aws_access_key and aws_secret_key  and Default region name


parser = argparse.ArgumentParser(description='Trim AWS snapshots')
parser.add_argument('-d', '--retention_days',
                    required=False,
                    default=7,
                    help='retain snapshots newer than this value. defaults to 7')
parser.add_argument('-s', '--safemode',
                    required=False,
                    default='True',
                    help='set the dry run flag')
parser.add_argument('-t', '--server_name',
                    required=False,
                    default=None,
                    help='Please provide the data to populate ServerName tag')
parser.add_argument('-r', '--region',
                    required=False,
                    type=str,
                    default='ap-southeast-2',
                    help='region where you want to manage snapshots')
parser.add_argument('-c', '--snap_no_to_keep',
                    required=False,
                    default=2,
                    help='set number of snapshots to retain. defaults to 2')

args = parser.parse_args()

ec2 = connect_to_region(args.region)

'''
Provide the suggested list of possible server names to use if none provided
'''

if args.server_name is None:
    parser.print_help()
    print 'suggested names for --server_name'
    for res in ec2.get_all_instances():
        for inst in res.instances:
            print inst.tags['Name']
    sys.exit()
else:
    server_name = str(args.server_name)  # accept ServerName tag value from commandline

'''
verify and set safe mode to True or False
'''
if args.safemode == 'True':
    args.safemode = bool(True)
else:
    print('Safemode Disabled')
    args.safemode = bool(False)

print args
'''
 number of days timestamp beyond which the snapshots will be deleted.
'''
try:
    retention_days = int(args.retention_days)
    print "Setting days to retain as {days}".format(days=retention_days)
except IndexError:
    retention_days = 7
    print "days to retain is set to default 7"


snap_no_to_keep = int(args.snap_no_to_keep)
'''
set the tag filters to prevent accidental deletion
'''
tag_value = server_name  # accept ServerName tag value from commandline
tag_name = 'tag:ServerName'  # this is the flag to identify each server type
filters = {
    tag_name: tag_value,
    'tag:delflag': 'on'
}

try:
    want_to_dry_run = args.safemode
    print "DryRun:{0}".format(want_to_dry_run)
except IndexError:
    want_to_dry_run = True
    print "DryRun flag not provided. Defaulting to :{0}".format(want_to_dry_run)

del_counter = 0  # to return count of snapshots deleted

delete_time = datetime.utcnow() - timedelta(days=retention_days)
print 'Warning: Del snapshots older than {days} days ({del_date})'.format(
    days=retention_days,
    del_date=delete_time
)
print 'ALL snapshots with {filters}'.format(filters=filters)
snapshots = ec2.get_all_snapshots(filters=filters)
snap_count = len(snapshots)
print'{0}:{1} available'.format(snapshots, snap_count)

'''
 create a dictionary with snapshot id and date and then sort it based on date
'''
snapdict = {}
for snapshot in snapshots:
    dictentry = {snapshot.id: snapshot.start_time}
    snapdict.update(dictentry)

print 'sorted list by date'
sorted_snap_date = sorted(snapdict.items(), key=operator.itemgetter(1))
print sorted_snap_date


# sorted snaps using lambda function
# snap_sorted = sorted([(s.id, s.start_time) for s in snapshots], key=lambda k: k[1])
# loop through each snapshot

for snapshot_name in sorted_snap_date:
    for snapshot in ec2.get_all_snapshots(snapshot_name[0]):
        print ('PROCESSING ' + snapshot.id + " " + snapshot.start_time)
        if snap_count == snap_no_to_keep:  # continuously check keep value
            print 'snapshot count at desired Keep Value({0}); ' \
                  'Still want to DELETE? lower the count and rerun'.format(snap_no_to_keep)
            print 'Deleted {number} snapshots'.format(number=del_counter)
            exit(127)
        # identify the snapshot start time.
        start_time = datetime.strptime(snapshot.start_time, '%Y-%m-%dT%H:%M:%S.000Z')
        # print 'starttime {} < deletetime {} : snapcount {} == snapnotokeep {}'.format(start_time, delete_time, snap_count, snap_no_to_keep)

        if start_time < delete_time:
            print 'DELETING {id}:size {size} GB:{name}'.format(
                id=snapshot.id,
                size=snapshot.volume_size,
                name=tag_value
            )
            snap_count -= 1
            del_counter += 1
            # The below statements are destructive. Make sure you do a dryrun first to avoid trouble.
            # snapshot.delete(dry_run=True)
            # want_to_dry_run = True
            try:
                print "DryRun is set to:{0}".format(want_to_dry_run)
                snapshot.delete(dry_run=want_to_dry_run)
            except Exception as e:
                print "Dry MODE is set to:{0}".format(args.safemode)
                print e
                print('Use option -s False to override')
print 'Deleted {number} snapshots'.format(number=del_counter)
