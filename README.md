# scheduler-extender-17341116
# Extender的工作逻辑
  整个程序的文件组织基本参照了助教给的模板，只在调度器的业务逻辑（是否应该批准该节点）处进行了修改。
## main.go
  main.go负责接收请求和初始随机数种子，并按照请求的url执行Filter和Prioritize
  ```
func init() {
	rand.Seed(time.Now().UTC().UnixNano())
}
func main() {
	router := httprouter.New()
	router.GET("/", controller.Index)
	router.POST("/filter", controller.Filter)
	router.POST("/prioritize", controller.Prioritize)
	log.Printf("start up scheduler-extender!\n")
	log.Fatal(http.ListenAndServe(":8888", router))
}
  ```
## controller
  controller中实现了filter和prioritize。
  filter通过判断随机数个位是否为7来判断是否批准该节点，如果是的话我们就认为这是⼀个“被批准”的节点，否则拒绝批准该节点。
  ```
//判断函数
func LuckyPredicate(pod *v1.Pod, node v1.Node) (bool, []string, error) {
	lucky := rand.Intn(10) == 7
	if lucky {
		log.Printf("pod %v/%v fits on node %v\n", pod.Name, pod.Namespace, node.Name)
		return true, nil, nil
	}
	log.Printf("pod %v/%v doesn't fit on node %v\n", pod.Name, pod.Namespace, node.Name)
	return false, []string{LuckyPredFailMsg}, nil
}
  ```
  prioritize沿用了助教给出的模板，在最⼤优先级内随机取⼀个值。
  ```
  func prioritize(args schedulerapi.ExtenderArgs) *schedulerapi.HostPriorityList {
	  pod := args.Pod
	  nodes := args.Nodes.Items
	  hostPriorityList := make(schedulerapi.HostPriorityList, len(nodes))
	  for i, node := range nodes {
		  score := rand.Intn(schedulerapi.MaxPriority + 1)
		  log.Printf(luckyPrioMsg, pod.Name, pod.Namespace, score)
		  hostPriorityList[i] = schedulerapi.HostPriority{
			  Host:  node.Name,
			  Score: score,
	    }
	  }
    return &hostPriorityList
  }
  ```
