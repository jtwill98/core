### Snapshot API

Snapshots are really a time captured Deployment and effectly refect the a Deployment state.

####Snapshots have 3 primary states:

| STATE | DESCRIPTION |
|:-----------|:----------------------------------------------------------------|
|PROPOSED:| All snapshots on non-system deployments start in the PROPOSED state. When in this state, the annealer will ignore any noderoles in the snapshot, no hooks will be called for any state transitions, and user data for the node roles can be edited.|
|COMMITTED:| When a snapshot is in committed, userdata in node roles cannot be edited, the annealer will transition any nodes in TODO, and hooks will be called for any state transitions.|
|ARCHIVED: | Snapshots in archived state will be ignored by the annealer and by just about everything. Eventually you will be able to look through them to satisfy your curiosity about the past states of the cluster.|

####Substates of the COMMITTED state:
These sub-states are primarily for consumption by the UI.

| SUB-STATE | DESCRIPTION |
|:-----------|:----------------------------------------------------------------|
|ACTIVE | Is when all of the noderoles in a COMMITTED snapshot are in the ACTIVE state, and
|ERROR| Noderole in a COMMITTED snapshot is in the ERROR state.|

####State transitions for snapshots:

| TRANSITION | DESCRIPTION |
|:-------------------------------|:--------------------------------------------|
|(previous snapshot) -> PROPOSED | The previous snapshot is set to ARCHIVED. The new snapshot gets a copy of all the noderoles in the previous snapshot, which will have the same data and the same states as the old ones.|
|PROPOSED -> COMMITTED | All PROPOSED noderoles in the snapshot are transitioned to TODO or BLOCKED, depending on whether or not they have any non-active parents. The only way to go from proposed to committed is via the commit method for the snapshot.|
|COMMITTED -> PROPOSED | The only way to go from COMMITTED to PROPOSED is via the snapshot recall method.|
(COMMITTED || PROPOSED) -> ARCHIVED | A snapshot gets transitioned to ARCHIVED when it is superceded by a new snapshot. Archived snapshots are ignored.|

The system deployment is super-special in that its snapshots can not
be transitioned to PROPOSED.

####Instances:
Instances can only be created by:-

1. Cloning the template instance using barclamp.create_proposal (see Barclamp/Config)
2. Editing an existing configuration which clones from the active instance

#### API Actions

| Verb | URL | Comments |
|:------|:-----------------------|:----------------|
| GET  |api/v2/snapshots |List |
| GET  |api/v2/snapshots/:id |Specific Item |
| PUT  |api/v2/snapshots/:id |Update Item |
| POST  |api/v2/snapshots |Create Item |
| DELETE  |- |NOT SUPPORTED |
| PUT  |api/v2/snapshots/:id/propose |Creates a new snapshot as the Deployment Head |
| PUT  |api/v2/snapshots/:id/commit |Commits snapshot for action by the Annealer |
| PUT  |api/v2/snapshots/:id/recall |Interrupts the Annealer |
| GET  |api/v2/snapshots/:id/node_roles |Shows the node-roles for a snapshot |
| GET  |api/v2/snapshots/:id/graph |Shows the node-role relationships for a snapshot.  Used for the UI Graph|
