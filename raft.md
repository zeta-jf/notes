
# 2. Replicated state machine

first machine logs its operations, second machine use these logs to reach the same state

consenus algorithms have following properties:

- ensuer safety
- available as long as any majority of the servers are reachable.
- do not depend on timming to ensure the consistency
- comaand can complete as soon as a majority of the cluster has responsed(minority of slow servers need not impoct the overall system performance)

# 5. The raft consensus algorithm

- Leader election
- Log replication
- Safety

## 5.1 Raft basics

each terms begins with an election

if a server's term is lower than others, then it will update its current term.

if a leader or a condidate find its term is smaller than current term, it immediately change the state to follower

if a server receives a request with a state term number, it rejects the request

## 5.2 Leader election

leader use rpc to call followers to maintain their authority

if there is no heartbeat request from leader for a serveral time:
1. follower would change its state to candidate
2. increase term
3. issue RequestVote RPCs to each of the other servers
wait untile one of the following things happened:
(a) it wins the election
(b) another server establishes itself as leader
(c) a period of time goes by with no winner

in each term, each server could only vote for one candidate
it then send heartbeat to all servers to maintain authority

if a candidate received a AppendEntryies RPC:
if term >= current term: change state to follower and process the RPC
if term <= current: reject request

if no winner, candidate will increase the term and begin a new election

## 5.3 Log replication
log: term and index (position at the log)
committed entry: leader has replicated it on majority of the servers.

Log Matching Property:
> if two entries in different logs have the same index and term, then they store the same command

> if two entries in different logs have the same index and term, then the logs are identical in all preceding entries

when a leader first comes to power, it initializes all nextIndex values to the index just after the last one in its log. And force followers to change their logs.

## 5.4 Safety
goal: leader store all committed log entries

raft prevent a candidate from winning an election unless its log contains all committed entries.

followers would deny the vote if its own log is more up-to-date than that of the candidate

how to compare up-to-date: comparing term, same term then compare length

### example:

leader 1 1 1 4 4 5 5 6 6 6 

a      1 1 1 4 4 5 5 6 6 
b      1 1 1 4 
c      1 1 1 4 4 5 5 6 6 6 6
d      1 1 1 4 4 5 5 6 6 6 7 7  
e      1 1 1 4 4 4 4  
f      1 1 1 2 2 2 3 3 3 3 3

committed log entries: 1 1 1 4 4 5 5 6 6
potential candidates: a, b, d

* the reason b cannot be the leader:
1) servers would not vote for the server whose log is not updated as its own
2) the lastest log of the majority is the second 6
so b cannot get the vote of the majority

* example: 5 servers, leader committed log1, f1, f2 recevied, f3, f4 not, then leader, f1, crashed.

in this situation, if f3 or f4 wants to become leader, it will still need 3 votes, but f2 would reject if f3 or f4 is the candidate, because the log entry in f3 or f4 is outdated compared to the f2, so the only possible final state is f2 become the leader.

