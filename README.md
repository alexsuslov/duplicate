# Income

```plantuml
@startuml
'https://plantuml.com/sequence-diagram

autonumber
group queue.Pool
  SD -> Pool: {id}
  Pool --> SD : pid
  SD -> Pool: {id}
  Pool --> SD : pid
  SD -> Pool: {id}
  Pool --> SD : pid
  SD -> Pool: {id}
  Pool --> SD : pid
end


group (duplicate.Avoid)
  SD -> Filter: {id}
  SD -> Filter: {id}
  SD -> Filter: {id}
  SD -> Filter: {id}
end
note right
Collect duplicate
https://github.com/alexsuslov/duplicate
end note
group Pool
  Filter -> Agent: {id}
end

note right
Queue pool (Ram, Stored)
https://github.com/alexsuslov/queue
end note

  Agent -> SD: Get state by id
  SD --> Agent: State
Agent->Postbox : Post
Postbox -->Agent : Result add to queue

@enduml

```

```go
const service="SERVICE"

Pool := &queue.Pool{}
Avoid := &duplicate.Avoid{}

go func(p *queue.Pool, d duplicate.Avoid) {
  for{
    if key, ok := Pool.Pop().(string); ok {
      ctx, fn := context.WithCancel(context.Background())
      d.Push(key,fn)
      if err:=Post(ctx, key); err!=nil{
        if err1:=ErrorTicket(key, err); err1!=nil{
          Notify(service, key, err, err1)
        }
      }
      d.Remove(req.ID)
    }
  }
}(Pool, Avoid)

func(w http.ResponseWriter, r *http.Request) {
  key := r.URL.Query().Get("key")
  Pool.Push(key)
  w.Write([]byte("done"))
}


```
