---
title: Apache Mesos - Task State Reasons
layout: documentation
---

# Task State Reasons

Some TaskStatus messages will arrive with the `reason` field set to a value
that can allow frameworks to display better error messages and to implement
special behaviour for some of the reasons.

For most reasons, the `message` field of the TaskStatus message will give a
more detailed, human-readable error description.

Not all status updates will contain a reason.


# Guidelines for Framework Authors

Frameworks that implement their own executors are free to set the reason field
on any status messages they produce.

Note that executors can not generally rely on the fact that the scheduler will
see the status update with the reason set by the executor, since only the
latest update for each different task state is stored and re-transmitted. See
in particular the description of `REASON_RECONCILIATION` below.

Most reasons describe conditions that can only be detected in the master or
agent code, and will accompany automatically generated status updates from
either of these.

For consistency with the existing usages of the different task reasons, we
recommend that executors restrict themselves to the following subset if they
use a non-default reason in their status updates.

<table class="table table-striped">

<tr><td><code>     REASON_TASK_CHECK_STATUS_UPDATED
</code></td><td>   For executors that support running task checks, it is
                   recommended to generate a status update with this reason
                   every time the task check status changes, together with a
                   human-readable description of the change in
                   the <code> message </code> field.
</td></tr>


<tr><td><code>     REASON_TASK_HEALTH_CHECK_STATUS_UPDATED
</code></td><td>   For executors that support running task health checks, it
                   is recommended to generate a status update with this reason
                   every time the health check status changes, together with a
                   human-readable description of the change in
                   the <code> message </code> field.
<strong>           Note:
</strong>          The built-in executors additionally send an update with
                   this reason every time a health check is unhealthy.
</td></tr>


<tr><td><code>     REASON_TASK_INVALID
</code></td><td>   For executors that implement their own task validation
                   logic, this reason can be used when the validation check
                   fails, together with a human-readable description of the
                   failed check in the <code> message </code> field.
</td></tr>


<tr><td><code>     REASON_TASK_UNAUTHORIZED
</code></td><td>   For executors that implement their own authorization logic,
                   this reason can be used when authorization fails, together
                   with a human-readable description in
                   the <code> message </code> field.
</td></tr></table>



# Reference of Reasons Currently Used in Mesos

## Deprecated Reasons

The reason `REASON_COMMAND_EXECUTOR_FAILED` is deprecated and will be removed
in the future. It should not be referenced by newly written code.


## Unused Reasons

The reasons `REASON_CONTAINER_LIMITATION`, `REASON_INVALID_FRAMEWORKID`,
`REASON_SLAVE_UNKNOWN`, `REASON_TASK_UNKNOWN` and
`REASON_EXECUTOR_UNREGISTERED` are not used as of Mesos 1.4.


## Reasons for Terminal Status Updates

For these status updates, the reason indicates *why* the task state changed.
Typically, a given reason will always appear together
with the same state.

Typically they are generated by mesos when an error occurs that prevents
the executor from sending its own status update messages.

Below, a partition-aware framework means a framework which has the
`Capability::PARTITION_AWARE` capability bit set in its `FrameworkInfo`.
Messages generated on the master will have the `source` field set to
`SOURCE_MASTER` and messages generated on the agent will have it set
to `SOURCE_AGENT` in the v1 API or `SOURCE_SLAVE` in the v0 API.

As of Mesos 1.4, the following reasons are being used.


### For state `TASK_FAILED`
#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_CONTAINER_LAUNCH_FAILED
</code></td><td>   The task could not be launched because its container failed
                   to launch.
</td></tr>


<tr><td><code>     REASON_CONTAINER_LIMITATION_MEMORY
</code></td><td>   The container in which the task was running exceeded its
                   memory allocation.
</td></tr>


<tr><td><code>     REASON_CONTAINER_LIMITATION_DISK
</code></td><td>   The container in which the task was running exceeded its
                   disk quota.
</td></tr>


<tr> <td><code>    REASON_IO_SWITCHBOARD_EXITED
</code></td><td>   The I/O switchboard server terminated unexpectedly.
</td></tr>


<tr><td><code>     REASON_EXECUTOR_REGISTRATION_TIMEOUT
</code></td><td>   The executor for this task didn't register with the agent
                   within the allowed time limit.
</td></tr>


<tr><td><code>     REASON_EXECUTOR_REREGISTRATION_TIMEOUT
</code></td><td>   The executor for this task lost connection and didn't
                   reregister within the allowed time limit.
</td></tr>


<tr><td><code>     REASON_EXECUTOR_TERMINATED
</code></td><td>   The tasks' executor terminated abnormally, and no more
                   specific reason could be determined.
</td></tr></table>



### For state `TASK_KILLED`
#### In status updates generated on the master:

<table class="table table-striped">

<tr><td><code>     REASON_FRAMEWORK_REMOVED
</code></td><td>   The framework to which this task belonged was removed.
<br/><strong>      Note:
</strong>          The status update will be sent out before the task is
                   actually killed.
</td></tr>
<tr><td><code>     REASON_TASK_KILLED_DURING_LAUNCH
</code></td><td>   This task, or a task within this task group, was killed
                   before delivery to the agent.
</td></tr></table>


#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_TASK_KILLED_DURING_LAUNCH
</code></td><td>   This task, or a task within this task group, was killed
                   before delivery to the executor.
<br/><strong>      Note:
</strong>          Prior to version 1.5, the agent would in this situation
                   sometimes send status updates with reason set
                   to <code> REASON_EXECUTOR_UNREGISTERED </code> and
                   sometimes without any reason set, depending on details of
                   the timing of the executor launch and the kill command.
</td></tr></table>



### For state `TASK_ERROR`
#### In status updates generated on the master:

<table class="table table-striped">

<tr><td><code>     REASON_TASK_INVALID
</code></td><td>   Task or resource validation checks failed.
</td></tr>


<tr><td><code>     REASON_TASK_GROUP_INVALID
</code></td><td>   Task group or resource validation checks failed.
</td></tr>


<tr><td><code>     REASON_TASK_UNAUTHORIZED
</code></td><td>   Task authorization failed on the master.
</td></tr>


<tr><td><code>     REASON_TASK_GROUP_UNAUTHORIZED
</code></td><td>   Task group authorization failed on the master.
</td></tr></table>


#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_TASK_UNAUTHORIZED
</code></td><td>   Task authorization failed on the agent.
</td></tr>


<tr><td><code>     REASON_TASK_GROUP_UNAUTHORIZED
</code></td><td>   Task group authorization failed on the agent.
</td></tr></table>



### For state `TASK_LOST`
#### In status updates generated on the master:

<table class="table table-striped">

<tr><td rowspan="2">
<code>             REASON_SLAVE_DISCONNECTED
</code></td><td>   The agent on which the task was running disconnected, and
                   didn't reconnect in time.
</td></tr>
<tr><td>           The task was part of an accepted offer, but the agent
                   sending the offer disconnected in the meantime.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr>


<tr><td><code>     REASON_MASTER_DISCONNECTED
</code></td><td>   The task was part of an accepted offer which couldn't be
                   sent to the master, because it was disconnected.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
<br/><strong>      Note:
</strong>          Despite the source being set to <code> SOURCE_MASTER </code>,
                   the message is not sent from the master but locally from
                   the scheduler driver.
<strong>           Note:
</strong>          This reason is only used in the v0 API.
</td></tr>


<tr><td rowspan="3">
<code>             REASON_SLAVE_REMOVED
</code></td><td>   The agent on which the task was running was removed.
</td></tr>
<tr><td>           The task was part of an accepted offer, but the agent
                   sending the offer was disconnected in the meantime.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will be
                   to <code> TASK_DROPPED </code> instead.
</td></tr>
<tr><td>           The agent on which the task was running was marked
                   unreachable.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_UNREACHABLE </code> instead.
</td></tr>


<tr><td><code>     REASON_RESOURCES_UNKNOWN
</code></td><td>   The task was part of an accepted offer which used
                   checkpointed resources that are not known to the master.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr></table>


#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_SLAVE_RESTARTED
</code></td><td>   The task was launched during an agent restart, and never
                   got forwarded to the executor.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr>


<tr><td><code>     REASON_CONTAINER_PREEMPTED
</code></td><td>   The container in which the task was running was pre-empted
                   by a QoS correction.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will be changed
                   to <code> TASK_GONE </code> instead.
</td></tr>


<tr><td><code>     REASON_CONTAINER_UPDATE_FAILED
</code></td><td>   The container in which the task was running was discarded
                   because a resource update failed.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_GONE </code> instead.
</td></tr>


<tr><td><code>     REASON_EXECUTOR_TERMINATED
</code></td><td>   The executor which was supposed to execute this task was
                   already terminated, or the agent receives an instruction to
                   kill the task before the executor was started.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr>


<tr><td><code>     REASON_GC_ERROR
</code></td><td>   A directory to be used by this task was scheduled for GC
                   and it could not be unscheduled.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr>


<tr><td><code>     REASON_INVALID_OFFERS
</code></td><td>   This task belonged to an accepted offer that didn't pass
                   validation checks.
<br/><strong>      Note:
</strong>          For partition-aware frameworks, the state will
                   be <code> TASK_DROPPED </code> instead.
</td></tr></table>



### For state `TASK_DROPPED`:
#### In status updates generated on the master:

<table class="table table-striped">

<tr><td><code>     REASON_SLAVE_DISCONNECTED
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>


<tr><td><code>     REASON_SLAVE_REMOVED
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>


<tr><td><code>     REASON_RESOURCES_UNKNOWN
</code></td><td>   See <code> TASK_LOST </code>
</td></tr></table>


#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_SLAVE_RESTARTED
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>


<tr><td><code>     REASON_GC_ERROR
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>


<tr><td><code>     REASON_INVALID_OFFERS
</code></td><td>   See <code> TASK_LOST </code>
</td></tr></table>



### For state `TASK_UNREACHABLE`:
#### In status updates generated on the master:

<table class="table table-striped">

<tr><td><code>     REASON_SLAVE_REMOVED
</code></td><td>   See <code> TASK_LOST <code>
</td></tr></table>



### For state `TASK_GONE`
#### In status updates generated on the agent:

<table class="table table-striped">

<tr><td><code>     REASON_CONTAINER_UPDATE_FAILED
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>

<tr><td><code>     REASON_CONTAINER_PREEMPTED
</code></td><td>   See <code> TASK_LOST </code>
</td></tr>

<tr><td><code>     REASON_EXECUTOR_PREEMPTED
</code></td><td>   Renamed to <code> REASON_CONTAINER_PREEMPTED </code> in
                   Mesos 0.26.
</td></tr></table>



## Reasons for Non-Terminal Status Updates

These reasons do not cause a state change, and will be sent along with the
last known state of the task. The reason field indicates *why* the status
update was sent.


<table class="table table-striped">

<tr><td><code>     REASON_RECONCILIATION
</code></td><td>   A framework requested implicit or explicit reconciliation
                   for this task.
<br/><strong>      Note:
</strong>          Status updates with this reason are not the original ones,
                   but rather a modified copy that is re-sent from the master.
                   In particular, the original <code> data </code>
                   and <code> message </code> fields are erased and the
                   original <code> reason </code> field is overwritten
                   by <code> REASON_RECONCILIATION </code>.
</td></tr>


<tr><td><code>     REASON_TASK_CHECK_STATUS_UPDATED
</code></td><td>   A task check notified the agent that its state changed.
<br/><strong>      Note:
</strong>          This reason is set by the executor, so for tasks that are
                   running with a custom executor, whether or not status
                   updates with this reasons are sent depends on that
                   executors implementation.
<strong>           Note:
</strong>          Currently, when using one of the built-in executors, this
                   reason is only used within status updates with task
                   state <code> TASK_RUNNING </code>.
</td></tr>


<tr><td><code>     REASON_TASK_HEALTH_CHECK_STATUS_UPDATED
</code></td><td>   A task health check notified the agent that its
                   state changed.
<br/><strong>      Note:
</strong>          This reason is set by the executor, so for tasks that are
                   running with a custom executor, whether or not status
                   updates with this reasons are sent depends on that
                   executors implementation.
<strong>           Note:
</strong>          Currently, when using one of the built-in executors, this
                   reason is only used within status updates with task
                   state <code> TASK_RUNNING </code>.
</td></tr>


<tr><td><code>     REASON_SLAVE_REREGISTERED
</code></td><td>   The agent on which the task was running has reregistered
                   after being marked unreachable by the master.
<br/><strong>      Note:
</strong>          Due to garbage collection of the unreachable and gone agents
                   in the registry and master state Mesos also sends such status
                   updates for agents unknown to the master.
<strong>           Note:
</strong>          Status updates with this reason are modified copies re-sent
                   by the master which reflect the states of the tasks reported
                   by the agent upon its re-registration. See comments for
                   <code> REASON_RECONCILIATION </code>.
</td></tr></table>