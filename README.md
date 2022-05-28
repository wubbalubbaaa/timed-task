# timed-task

It's a library helps you to add timed tasks in persistent way

here is the simplest demo:

```go 


type serialize struct{}

func (s *serialize) Deserialize(data []byte) (interface{}, error) {
	digitt, _ := strconv.ParseInt(string(data), 10, 64)
	t := time.Unix(digitt/1e9, digitt-int64(int(digitt/1e9))*1e9)
	return t, nil
}

func (s *serialize) Serialize(data interface{}) ([]byte, error) {
	if v, ok := data.(time.Time); ok {
		t := v.UnixNano()
		return []byte(strconv.FormatInt(t, 10)), nil
	}
	return nil, errors.ErrSerialize
}

func TestTimedTask(t *testing.T) {
	var se serialize
	timedtask, err := NewTimedTask("tmp", &se)
	if err != nil {
		log.Printf("new timedtask faild : %s", err)
	}
	err = timedtask.Recover()
	if err != nil {
		log.Printf("recover faild : %s", err)
	}
	timedtask.SetTask(func(msg interface{}) {
		t := time.Now()
		fmt.Println("触发误差:", t.Sub(msg.(time.Time)))
	})
	for i := 0; i < 10000; i++ {
		t := time.Millisecond * time.Duration(10*rand.Intn(100)+1)
		timedtask.AddTrigger(t, time.Now().Add(t))
		time.Sleep(time.Millisecond * 10)
	}

}

```

And if you want to do fixed-duration task , you can use this way:

```go
	var se serialize
	ctx, cancel := context.WithCancel(context.Background())
	go func() {
		time.Sleep(time.Minute * 10)
		cancel()
	}()
	timedtask, err := NewTimedTask("tmp", &se)
	if err != nil {
		log.Printf("new timedtask faild : %s", err)
	}
	err = timedtask.Recover()
	if err != nil {
		log.Printf("recover faild : %s", err)
	}
	timedtask.SetTask(func(msg interface{}) {
		// do something
		t := time.Now()
		fmt.Println("触发误差:", t.Sub(msg.(time.Time)), "触发时间:", msg.(time.Time))
		// 添加下一次
		timedtask.TriggerAfter(time.Second, time.Now().Add(time.Second))
	})
	triggerTime, _ := time.ParseInLocation("2006-01-02 15:04:05", "2022-05-28 09:42:11", time.Local)
	timedtask.TriggerWhen(triggerTime, triggerTime)
	<-ctx.Done()

```