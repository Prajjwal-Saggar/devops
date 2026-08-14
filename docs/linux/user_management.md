# Linux User & Group Management

## User Management

- Create a user: `sudo useradd xyz`
- Set password: `sudo passwd xyz`
- Delete user: `sudo userdel xyz`
- Delete user with home directory: `sudo userdel -r xyz`
- Switch user: `su xyz`
- Switch user with login shell: `su - xyz`
- Check current user: `whoami`

## If a User Cannot Be Deleted

Sometimes a user cannot be deleted because they still have running processes. Check the processes:

`ps -u xyz`

If you are already logged in as that user:

`ps`

Check a specific process:

`ps -fp <process_id>`

Normal process termination:

`kill <process_id>`

Forcefully terminate a process:

`kill -9 <process_id>`

Kill all processes belonging to a user:

`sudo pkill -u xyz`

After stopping the processes, delete the user:

`sudo userdel -r xyz`

## Group Management

- Create a group: `sudo groupadd xyz`
- Delete a group: `sudo groupdel xyz`
- Change a user's primary group: `sudo usermod -g xyz danny`
- Add a user to a secondary group: `sudo usermod -aG xyz danny`
- Check a user's groups: `groups danny`
- Get detailed user and group information: `id danny`
