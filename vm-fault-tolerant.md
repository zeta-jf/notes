
# 1. Introduction

typervisor capture all the necessary information about non-deterministic operations on the primary VM and to replay these operations correctly on the backup VM.

# 2. Basic FT design
all input (internet or mouse or keyboard) are send to the primary, primary use *logging channel* to sync primary with secondary.

## 2.1 Deterministic replay implementation

1. correctly capturing all the input and non-determinism necessary to ensure deterministic execution of a backup virtual machine.
2. correctly applying the inputs and non-determinism to the backup vm
3. not degrade performance.

solution: store all operation into a file(not in the disk)
using logging channel to send operation file

## 2.2 FT Protocol

> **Output Requirement:** if the backup VM ever takes over after a failure of the primary, the backup VM will continue executing in a way that is entirely consistent with all outputs that the primary VM has sent to the external world.

> **Output Rule** the primary VM may not send an output to the external world, until the backup VM has received and acknowledged the log entry associated with the operation producing the output.

cannot guarantee that all outputs are produced exactly once in failure situation.

## 2.3 Detecting and Responding to Failure
using heart beat(udp) to detect failure, use atomic update on shared disk to guarantee there is only one primary.

# 3 Practical implementation of FT

## 3.1 Starting and REstarting FT VMs

vm has a black technology called VMotion, which could copy a vm to a remote host.

it has a cluster, so it could pick a machine from the master and use VMotion to do the backup.

## 3.2 Managing the Logging Channel

Primary produce operation message to the channel, backup consuming message from the channel.

if the channel is full, primary wait (affect the service), if the channel is empty, backup wait (not affect the service)

since one host may has many VM, it is possible that backup VM cannot get enough resources to execute operation messages.

operation messages cannot reach a big number because when backup take over the service, the lag time contains the time of executing the log, if there is too many logs, the lag time would be enormous.

so if the execution lag is greater than 1s, primary VM would get lower resources(like CPU) to force it slow down.


