# go-atomic-demo

This is demo for how to use mutex and atomic for race condition data

## Situation

Say we have a struct as follow


```go
type Player struct {
	health int
}

func NewPlayer() *Player {
	return &Player{
		health: 100,
	}
}

func startUILoop(p *Player) {
	ticker := time.NewTicker(time.Second)
	for {
		fmt.Printf("player health: %d\r", p.getHealth())
		<-ticker.C
	}
}
func (p *Player) getHealth() int {
	return p.health
}
func (p *Player) takeDamage(value int) {
	p.health -= value
}
func startGameLoop(p *Player) {
	ticker := time.NewTicker(time.Millisecond * 300)
	for {
		p.takeDamage(rand.Intn(40))
		if p.getHealth() <= 0 {
			fmt.Println("Game Over")
			break
		}
		<-ticker.C
	}
}

func main() {
	player := NewPlayer()
	go startUILoop(player)
	startGameLoop(player)
}
```

in this kind of scene

we found that the health data maybe 

possible access by more than one goroutine at the same time

if use 

```shell
go test -v ./... --race
```

will found out the race condition problem

## Solution for race condition data

1. mutex

2. atomic value


### mutex 

use lock for Read/Write data

like following

```shell
type Player struct {
	mu     sync.RWMutex
	health int
}

func NewPlayer() *Player {
	return &Player{
		health: 100,
	}
}

func startUILoop(p *Player) {
	ticker := time.NewTicker(time.Second)
	for {
		fmt.Printf("player health: %d\r", p.getHealth())
		<-ticker.C
	}
}
func (p *Player) getHealth() int {
	p.mu.RLock()
	defer p.mu.RUnlock()
	return p.health
}
func (p *Player) takeDamage(value int) {
	p.mu.Lock()
	defer p.mu.Unlock()
	p.health -= value
}
func startGameLoop(p *Player) {
	ticker := time.NewTicker(time.Millisecond * 300)
	for {
		p.takeDamage(rand.Intn(40))
		if p.getHealth() <= 0 {
			fmt.Println("Game Over")
			break
		}
		<-ticker.C
	}
}

func main() {
	player := NewPlayer()
	go startUILoop(player)
	startGameLoop(player)
}
```

### atomic

