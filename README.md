# Example

![](sd2agent1.png)

```go
const service="SERVICE"

Pool := &queue.Pool{}
Avoid := &duplicate.Avoid{}

go loop(Pool, Avoid)

func loop(p *queue.Pool, d *duplicate.Avoid) {
	for {
		key, ok := p.Pop().(string)
		if ok {
			ctx, fn := context.WithCancel(context.Background())
			d.Push(key, fn)
			go func() {
				Post(ctx, key)
				if ctx.Err() == nil {
					d.Remove(key)
				}
			}()
		}
	}
}


func(w http.ResponseWriter, r *http.Request) {
  key := r.URL.Query().Get("key")
  Pool.Push(key)
  w.Write([]byte("done"))
}


```
