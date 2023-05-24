## currentTerm
```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/log.go#L112

 func (l *Log) currentTerm() uint64 {
     if len(l.entries) == 0 {
         return l.startTerm
     }
        
     return l.entries[len(l.entries)-1].Term // 最后一个已提交 entry 所属的任期
 }
```



## AppendEntries
```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/server.go#L939

// Processes the "append entries" request.
func (s *server) processAppendEntriesRequest(req *AppendEntriesRequest) (*AppendEntriesResponse, bool) {
    if req.Term < s.currentTerm
        return _, false

    if req.Term == s.currentTerm {
        if s.state == Candidate  // step-down to follower when it is a candidate
            s.setState(Follower)
        s.leader = req.LeaderName
    } else {
        s.updateCurrentTerm(req.Term, req.LeaderName)
    }

    // Reject if log doesn't contain a matching previous entry.
    if err := s.log.truncate(req.PrevLogIndex, req.PrevLogTerm); err != nil {
        return newAppendEntriesResponse(s.currentTerm, false, s.log.currentIndex(), s.log.CommitIndex()), true
    }

    s.log.appendEntries(req.Entries)      // Append entries to the log.
    s.log.setCommitIndex(req.CommitIndex) // Commit up to the commit index.

    // once the server appended and committed all the log entries from the leader
    return newAppendEntriesResponse(s.currentTerm, true, s.log.currentIndex(), s.log.CommitIndex()), true
}

// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/log.go#L399
// Truncates the log to the given index and term. This only works if the log
// at the index has not been committed.
func (l *Log) truncate(index uint64, term uint64) error {
    if index < l.commitIndex // Do not allow committed entries to be truncated.
        return fmt.Errorf("raft.Log: Index is already committed (%v): (IDX=%v, TERM=%v)", l.commitIndex, index, term)

    if index > l.startIndex + len(l.entries) // Do not truncate past end of entries.
        return fmt.Errorf("raft.Log: Entry index does not exist (MAX=%v): (IDX=%v, TERM=%v)", len(l.entries), index, term)

    // If we're truncating everything then just clear the entries.
    if index == l.startIndex {
        l.file.Truncate(0)
        l.file.Seek(0, os.SEEK_SET)
        l.entries = []*LogEntry{}
    } else {
        // Do not truncate if the entry at index does not have the matching term.
        entry := l.entries[index-l.startIndex-1]
        if len(l.entries) > 0 && entry.Term != term
            return fmt.Errorf("raft.Log: Entry at index does not have matching term (%v): (IDX=%v, TERM=%v)", entry.Term, index, term)

        // Otherwise truncate up to the desired entry.
        if index < l.startIndex+uint64(len(l.entries)) {
            position := l.entries[index-l.startIndex].Position
            l.file.Truncate(position)
            l.file.Seek(position, os.SEEK_SET)
            l.entries = l.entries[0 : index-l.startIndex]
        }
    }

    return nil
}
```

```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/log.go#L467

func (l *Log) appendEntries(entries []*protobuf.LogEntry) error {
    startPosition, _ := l.file.Seek(0, os.SEEK_CUR) // 定位到起始写入位置

    for i := range entries { // Append each entry util hit an error.
        logEntry := &LogEntry{
            log:       l,             // 日志文件
            Position:  startPosition, // 起始写入位置
            pb:        entries[i],    // 待写入 log entry
        }

        size = l.writeEntry(logEntry, w)
        startPosition += size
    }

    return nil
}

func (l *Log) writeEntry(entry *LogEntry, w io.Writer) (int64, error) {
    if len(l.entries) > 0 {
        lastEntry := l.entries[len(l.entries)-1] // 上一个已经写入日志的 entry

        if entry.Term < lastEntry.Term           // 待写入 entry 所带的任期号不能小于前一 entry 所带的任期号
            return -1, Errorf("raft.Log: Cannot append entry with earlier term")
        if entry.Term == lastEntry.Term && entry.Index <= lastEntry.Index // 写入位置必须在前一个 entry 之后
            return -1, Errorf("raft.Log: Cannot append entry with earlier index in the same term")
    }

    size := entry.Encode(w) // 写到持久存储，然后就可以 append 到 entries list 了
    l.entries.append(entry)

    return int64(size), nil
}
```


## RequestVote

```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/server.go#L1071

func (s *server) processRequestVoteRequest(req *RequestVoteRequest) (*RequestVoteResponse, bool) {
    if _, ok := s.peers[req.CandidateName]; !ok // Candidate 节点不在本集群，直接 deny
        return _, false

    if req.Term < s.Term   // 请求来自更早的任期（old term），直接拒绝
        return _, false

    if req.Term > s.Term { // 看到了比本节点还要新的任期号（term number），update 到本节点
        s.updateCurrentTerm(req.Term, "")
    } else if s.votedFor != "" && s.votedFor != req.CandidateName { // 当前节点已经投给了其他 candidate
        return _, false
    }

    lastIndex, lastTerm := s.log.lastInfo()
    if lastIndex > req.LastLogIndex || lastTerm > req.LastLogTerm // 如果 candidate 的 log 比我们的要老，则不投给它
        return _, false

    // 投票给该 candidate，然后重置本节点的 election timeout
    s.votedFor = req.CandidateName
    return newRequestVoteResponse(s.currentTerm, true), true
}
```


## Follower

```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/server.go#L664

func (s *server) followerLoop() {
    for s.State() == Follower {
        select {
        case e := <-s.c:
            switch req := e.target.(type) {
            case JoinCommand:
                //If no log entries exist and a self-join command is issued then immediately become leader and commit entry.
                if s.log.currentIndex() == 0 && req.NodeName() == s.Name() {
                    s.setState(Leader)
                    s.processCommand(req, e)
                }
            case *AppendEntriesRequest:
                // If heartbeats get too close to the election timeout then send an event.
                if elapsedTime > electionTimeout*ElectionTimeoutThresholdPercent {
                    s.DispatchEvent(ElectionTimeoutThresholdEventType)
                }
                s.processAppendEntriesRequest(req)
            case *RequestVoteRequest:
                s.processRequestVoteRequest(req)
            case *SnapshotRequest:
                s.processSnapshotRequest(req)
            }

        case <-timeoutChan:
            s.setState(Candidate)
        }

        timeoutChan = afterBetween(s.ElectionTimeout(), s.ElectionTimeout()*2)
    }
}
```

## Candidate
```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/server.go#L730

// The event loop that is run when the server is in a Candidate state.
func (s *server) candidateLoop() {
    prevLeader := s.leader
    s.leader = ""

    lastLogIndex, lastLogTerm := s.log.lastInfo()
    doVote := true
    votesGranted := 0

    for s.State() == Candidate {
        if doVote {
            s.currentTerm++      // Increment current term, vote for self.
            s.votedFor = s.name

            // Send RequestVote RPCs to all other servers.
            respChan = make(chan *RequestVoteResponse, len(s.peers))
            for _, peer := range s.peers {
                 sendVoteRequest(s.currentTerm, s.name, lastLogIndex, lastLogTerm, respChan)
            }

            // Wait for either:
            //   * Votes received from majority of servers: become leader
            //   * AppendEntries RPC received from new leader: step down.
            //   * Election timeout elapses without election resolution: increment term, start new election
            //   * Discover higher term: step down (§5.1)
            votesGranted = 1
            timeoutChan = afterBetween(s.ElectionTimeout(), s.ElectionTimeout()*2)
            doVote = false
        }

        // If we received enough votes then stop waiting for more votes.
        if votesGranted == s.QuorumSize() {
            s.setState(Leader)
            return
        }

        // Collect votes from peers.
        select {
        case resp := <-respChan:
            if success := s.processVoteResponse(resp); success
                votesGranted++

        case e := <-s.c:
            var err error
            switch req := e.target.(type) {
            case Command:
                err = NotLeaderError
            case *AppendEntriesRequest:
                s.processAppendEntriesRequest(req)
            case *RequestVoteRequest:
                s.processRequestVoteRequest(req)
            }

            // Callback to event.
            e.c <- err

        case <-timeoutChan:
            doVote = true
        }
    }
}
```

## Leader

```go
// https://github.com/etcd-io/etcd/blob/release-0.4/third_party/github.com/goraft/raft/server.go#L811

func (s *server) leaderLoop() {
    logIndex, _ := s.log.lastInfo()

    // Update the peers prevLogIndex to leader's lastLogIndex and start heartbeat.
    for _, peer := range s.peers {
        peer.setPrevLogIndex(logIndex)
        peer.startHeartbeat() // 定期发送心跳
    }

    // Commit a NOP after the server becomes leader.
    // "Upon election: send initial empty AppendEntries RPCs (heartbeat) to each server."
    s.Do(NOPCommand{})

    // Begin to collect response from followers
    for s.State() == Leader {
        select {
        case e := <-s.c:
            switch req := e.target.(type) {
            case Command:
                s.processCommand(req, e)
                continue
            case *AppendEntriesRequest:
                s.processAppendEntriesRequest(req)
            case *AppendEntriesResponse:
                s.processAppendEntriesResponse(req)
            case *RequestVoteRequest:
                s.processRequestVoteRequest(req)
            }
        }
    }

    s.syncedPeer = nil
}
```