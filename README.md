# example

```golang

package main

import (
	"fmt"
	"github.com/cc14514/mdns"
	"os"
	"time"
)

var tag = "_foobar._tcp"

func run() {
	// Setup our service export
	host, _ := os.Hostname()
	n := time.Now().Unix()
	selfid := fmt.Sprintf("myid:%d", n)
	fmt.Println("self.id", selfid)
	service, _ := mdns.NewMDNSService(host, tag, "", "", 8000, nil, []string{selfid})
	// Create the mDNS server, defer shutdown
	server, _ := mdns.NewServer(&mdns.Config{Zone: service})
	defer server.Shutdown()
	ticker := time.NewTicker(time.Second * 5)

	for {
		//execute mdns query right away at method call and then with every tick
		entriesCh := make(chan *mdns.ServiceEntry, 16)
		go func() {
			for entry := range entriesCh {
				if entry != nil && entry.Info != selfid {
					fmt.Println(" 🏠-- 👬 -->", entry.Info, entry.InfoFields)
				}
			}
		}()

		fmt.Println("starting mdns query")
		qp := &mdns.QueryParam{
			Domain:  "local",
			Entries: entriesCh,
			Service: tag,
			Timeout: time.Second * 5,
		}

		mdns.Query(qp)
		close(entriesCh)
		fmt.Println("mdns query complete")

		select {
		case <-ticker.C:
			continue
		}
	}
}

func main() {
	go run()
	c := make(chan int)
	<-c
}

```