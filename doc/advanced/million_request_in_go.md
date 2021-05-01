

> 原文地址：https://medium.com/smsjunk/handling-1-million-requests-per-minute-with-golang-f70ac505fcaa
>
> 作者：[Marcio Castilho](https://medium.com/@mcastilho?source=post_page-----f70ac505fcaa--------------------------------)

我在反垃圾邮件，反病毒和反恶意软件行业的多家公司工作了15年以上，现在我知道由于我们每天处理的海量数据，这些系统最终会变得多么复杂。

目前，我是活跃于网络安全行业的公司的smsjunk.com首席执行官和KnowBe4的首席架构师。

有趣的是，在过去约十年的时间里，作为一名软件工程师，我参与的所有Web后端开发大部分都是在`Ruby on Rails`中完成的。 不要误会我的意思，我喜欢`Ruby on Rails`，我相信这是一个了不起的框架，但是过了一段时间，您开始以`ruby`的方式思考和设计系统，而您忘记了如果您可以利用多线程，并行化，快速执行和较小的内存开销，那么软件体系结构本来会多么高效和简单。 多年以来，我一直是C / C ++，Delphi和C＃的开发人员，并且我刚刚开始意识到使用正确的工具完成工作可能会变得多么简单。

> 我对互连网一直在为之奋斗的语言和框架之争不是很了解。我相信效率，生产力和代码可维护性主要取决于您设计解决方案的简单程度。

## 问题

在使用我们的匿名遥测和分析系统时，我们的目标是能够处理来自数百万个客户端的大量POST请求。 Web处理程序将收到一个JSON文档，该文档可能包含许多需要写入Amazon S3的有效负载的集合，以便我们的map-reduce系统以后可以对该数据进行操作。

传统上，我们会考虑使用以下技术来组建系统架构：

- Sidekiq
- Resque
- DelayedJob
- Elasticbeanstalk Worker Tier
- RabbitMQ
- 等等...

并设置2个不同的集群，一个集群用于Web前端，另一个集群用于Worker，因此我们可以扩展我们可以处理的后台工作量。

但是从一开始，我们的团队就知道我们应该在Go中执行此操作，因为在讨论阶段，我们发现这可能是非常庞大的流量系统。 我已经使用Go大约2年了，在此我们开发了一些系统，但是没有一个系统能够承受如此多的负载。

我们首先创建一些结构来定义将通过POST调用接收的Web请求有效负载，以及将其上载到S3存储桶中的方法。

```go
type PayloadCollection struct {
	WindowsVersion  string    `json:"version"`
	Token           string    `json:"token"`
	Payloads        []Payload `json:"data"`
}

type Payload struct {
    // [redacted]
}

func (p *Payload) UploadToS3() error {
	// storageFolder方法可确保在键名中获得相同时间戳的情况下不会发生名称冲突
	storage_path := fmt.Sprintf("%v/%v", p.storageFolder, time.Now().UnixNano())

	bucket := S3Bucket

	b := new(bytes.Buffer)
	encodeErr := json.NewEncoder(b).Encode(payload)
	if encodeErr != nil {
		return encodeErr
	}

	// 我们发布到S3存储桶的所有内容均应标记为 'private'
	var acl = s3.Private
	var contentType = "application/octet-stream"

	return bucket.PutReader(storage_path, b, int64(b.Len()), contentType, acl, s3.Options{})
}
```

## 天真的go示例程序

最初，我们采用了一个非常幼稚的POST处理程序实现，只是试图将作业处理并行化为一个简单的goroutine：

```go
func payloadHandler(w http.ResponseWriter, r *http.Request) {

	if r.Method != "POST" {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	// 将正文读入字符串以进行json解码
	var content = &PayloadCollection{}
	err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
	if err != nil {
		w.Header().Set("Content-Type", "application/json; charset=UTF-8")
		w.WriteHeader(http.StatusBadRequest)
		return
	}
	
	// 遍历每个有效负载并将每个队列分别排队以发布到S3
	for _, payload := range content.Payloads {
		go payload.UploadToS3()   // <----- DON'T DO THIS
	}

	w.WriteHeader(http.StatusOK)
}
```

对于中等负载，这可能适合大多数人，但是很快就证明了这种方法在大规模情况下效果不佳。 我们期待着很多请求，但是当我们将第一个版本部署到生产环境时，数量并没有达到我们开始看到的数量级。 我们完全低估了流量。

上面的方法在几种不同的方式上是不好的。无法控制我们产生的goroutines。而且，由于我们每分钟收到一百万个POST请求，因此该代码很快就崩溃了。

## 再次尝试

我们需要找到一种不同的方式。 从一开始，我们就开始讨论如何保持请求处理程序的生命周期非常短，并在后台生成处理程序。 当然，这是在Ruby on Rails世界中必须执行的操作，否则，无论您使用的是puma，unicorn，passenger，您都将阻止所有可用的工作程序Web处理器（请不要进入JRuby讨论）。 然后，我们将需要利用常见的解决方案来执行此操作，例如Resque，Sidekiq，SQS等。之所以继续，是因为有许多方法可以实现此目的。

因此，第二次迭代是创建一个缓冲通道，我们可以在其中排队一些作业并将其上传到S3，并且由于我们可以控制队列中的最大项目数，并且我们有大量的RAM可以在内存中排队作业，因此我们 以为只将作业缓存在通道队列中是可以的。

```go
var Queue chan Payload

func init() {
    Queue = make(chan Payload, MAX_QUEUE)
}

func payloadHandler(w http.ResponseWriter, r *http.Request) {
    ...
    // 遍历每个有效负载并将每个队列分别排队以发布到S3
    for _, payload := range content.Payloads {
        Queue <- payload
    }
    ...
}
```

然后实际上要使作业出队并对其进行处理，我们使用了类似的方法：

```go
func StartProcessor() {
    for {
        select {
        case job := <-Queue:
            job.payload.UploadToS3()  // <-- STILL NOT GOOD
        }
    }
}
```

老实说，我不知道我们在想什么。 这一定是一个充满红牛的深夜。 这种方法并没有给我们带来任何好处，我们已经将有缺陷的并发与缓冲队列进行了交换，而这只是推迟了问题的发生。 我们的同步处理器一次只向S3上传一个有效负载，并且由于传入请求的速率远大于单个处理器向S3上传的能力，因此我们的缓冲通道很快达到了极限，并阻止了请求处理程序将更多项目排队的能力。

我们只是在避免问题，并最终倒计时到我们系统的死亡。在部署此有缺陷的版本后，我们的等待时间以固定的分钟数保持增长。

![](http://cdn.mjava.top//20210327112831.png)

## 更好的解决方案

我们已决定在使用Go channels时使用一种通用模式，以便创建一个2层通道系统，一个用于排队job，另一个用于控制同时在JobQueue上worker的数量。

想法是将上传到S3的文件并行化到某种程度的可持续性，这样既不会削弱计算机的性能，也不会导致S3产生连接错误。 因此，我们选择了创建Job / Worker模式。对于那些熟悉Java，C＃等的人，可以将其视为使用通道来实现Worker Thread-Pool的Golang方法。

```go
var (
	MaxWorker = os.Getenv("MAX_WORKERS")
	MaxQueue  = os.Getenv("MAX_QUEUE")
)

// 代表要运行的job
type Job struct {
	Payload Payload
}

// 发送工作请求的缓冲通道。
var JobQueue chan Job

// Worker 执行的 job
type Worker struct {
	WorkerPool  chan chan Job
	JobChannel  chan Job
	quit    	chan bool
}

func NewWorker(workerPool chan chan Job) Worker {
	return Worker{
		WorkerPool: workerPool,
		JobChannel: make(chan Job),
		quit:       make(chan bool)}
}

// Start方法启动工作程序的运行循环，监听退出通道，以防我们需要停止它
func (w Worker) Start() {
	go func() {
		for {
			// 将当前工作程序注册到worker程序队列中。
			w.WorkerPool <- w.JobChannel

			select {
			case job := <-w.JobChannel:
				// 收到请求
				if err := job.Payload.UploadToS3(); err != nil {
					log.Errorf("Error uploading to S3: %s", err.Error())
				}

			case <-w.quit:
				// we have received a signal to stop
				return
			}
		}
	}()
}

// worker停止监听工作请求
func (w Worker) Stop() {
	go func() {
		w.quit <- true
	}()
}
```



我们已经修改了Web请求处理程序，以使用有效负载创建Job结构的实例，并将其发送到JobQueue通道中，以供workers提取。

```go
func payloadHandler(w http.ResponseWriter, r *http.Request) {

    if r.Method != "POST" {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

    // Read the body into a string for json decoding
	var content = &PayloadCollection{}
	err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
    if err != nil {
		w.Header().Set("Content-Type", "application/json; charset=UTF-8")
		w.WriteHeader(http.StatusBadRequest)
		return
	}

    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {

        // let's create a job with the payload
        work := Job{Payload: payload}

        // Push the work onto the queue.
        JobQueue <- work
    }

    w.WriteHeader(http.StatusOK)
}
```



在我们的Web服务器初始化期间，我们创建一个Dispatcher并调用`Run()`创建工作池并开始侦听JobQueue中将出现的jobs。

```go
dispatcher := NewDispatcher(MaxWorker) 
dispatcher.Run()
```

以下是我们调度程序实现的代码：

```go
type Dispatcher struct {
	// A pool of workers channels that are registered with the dispatcher
	WorkerPool chan chan Job
}

func NewDispatcher(maxWorkers int) *Dispatcher {
	pool := make(chan chan Job, maxWorkers)
	return &Dispatcher{WorkerPool: pool}
}

func (d *Dispatcher) Run() {
    // starting n number of workers
	for i := 0; i < d.maxWorkers; i++ {
		worker := NewWorker(d.pool)
		worker.Start()
	}

	go d.dispatch()
}

func (d *Dispatcher) dispatch() {
	for {
		select {
		case job := <-JobQueue:
			// a job request has been received
			go func(job Job) {
				// try to obtain a worker job channel that is available.
				// this will block until a worker is idle
				jobChannel := <-d.WorkerPool

				// dispatch the job to the worker job channel
				jobChannel <- job
			}(job)
		}
	}
}
```



请注意，我们提供了要实例化并添加到我们的worker pool中的最大worker数。 由于我们已在带有docker化Go环境的项目中使用Amazon Elasticbeanstalk，并且我们始终尝试遵循[12要素](https://12factor.net/)方法来配置生产环境中的系统，因此我们从环境变量中读取这些值。 这样，我们可以控制多少个worker和Job Queue的最大大小，因此我们可以快速调整这些值，而无需重新部署群集。

```go
var ( 
  MaxWorker = os.Getenv("MAX_WORKERS") 
  MaxQueue  = os.Getenv("MAX_QUEUE") 
)
```



部署之后，我们立即发现所有延迟率都下降到了微不足道的水平，并且处理请求的能力急剧增加。

![](http://cdn.mjava.top//20210327115034.png)



在弹性负载均衡器完全热身后的几分钟，我们看到我们的ElasticBeanstalk应用程序每分钟可处理近一百万个请求。 通常，我们早上有几个小时的流量高峰，每分钟超过一百万。

一旦我们部署了新代码，服务器的数量就从100台服务器大幅下降到大约20台服务器。

![](http://cdn.mjava.top/blog/20210327115331.png)



正确配置集群和自动缩放设置后，我们甚至可以将其降低到仅4个EC2 c4。如果CPU连续5分钟超过90％，则大型实例和Elastic Auto-Scaling设置为生成新实例。

![](http://cdn.mjava.top/blog/20210327115602.png)

## 结论

在我的书中，简单总是胜出的。 我们本来可以设计一个包含许多队列，后台worker和复杂部署的复杂系统，但是相反，我们决定利用Elasticbeanstalk自动扩展的功能以及Golang开箱即用的效率和并发性的简单方法。

并非每天都有四台机器组成的集群，这些集群的功能可能不如我目前的MacBook Pro强大，每分钟处理POST请求写入Amazon S3存储桶的次数为100万次。

总有适合这项工作的工具。有时，当您的Ruby on Rails系统需要功能非常强大的Web处理程序时，请在ruby生态系统之外考虑一下，以获取更简单但功能更强大的替代解决方案。