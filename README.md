# `workaround-upstart-snafu`

When lied to about the behavior of a job’s main process wrt. forking with the
“`expect`” stanza, Upstart can get into a confused state where it’s tracking a
nonexistent pid.

This hack creates new short-lived processes until one gets the pid in question,
then has its parent die so that it’s going to get reaped by pid 1 (Upstart),
which will get Upstart out of the confused state.

When “`status <jobname>`” says “`<jobname> stop/killed, process 12345`” and
there’s no such process, run “`workaround-upstart-snafu 12345`” and wait until
it exits.

### Taking too long?

Depending on the speed of your system and `pid_max` setting, the process of exhausting all unused PIDs to get to the one you want can take some time. To speed things up (limit the PID space this script has to cover), you can temporarily lower your system's `max_pid` setting:

```
# First, save the current setting for `pid_max`:
OLD_MAX_PID=`sysctl kernel.pid_max`

# Get the current highest PID:
HIGHEST_PID=`ps axo pid | tail -n 1`

# Set `pid_max` to a few hundred higher than that:
sudo sysctl -w kernel.pid_max=$(($HIGHEST_PID+500))

# DON'T FORGET to set it back to it's original value when you're done!
sudo sysctl -w kernel.pid_max=$OLD_MAX_PID
```
