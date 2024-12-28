# Automation for Laziness: Controlling My Raspberry Pi with Minimal Effort (or Almost)

As a Software Engineer and lover of efficiency, I thrive on automating repetitive tasks. After all, why waste time doing something manually when you can automate it, right? So, I found myself constantly tired of SSHing into my Raspberry Pi and writing the shutdown command or restarting containers. It wasn’t the act of running the commands that bugged me – it was the act of writing them every time. That’s when I decided to automate everything. No more typing commands, no more repeated actions. Just a few taps on an Android app or the push of a button.

## Choosing Go: The Efficient Solution for 1GB of RAM

My Raspberry Pi 3 B is quite limited with only 1GB of RAM, so I had to pick something lightweight. While I could have gone for Python or even Node.js, I ended up choosing Go for a very good reason – it’s efficient, fast, and perfect for small-scale projects like mine. I didn’t need something resource-heavy; I needed something that was performant and capable of running smoothly on a device with limited hardware.

Go’s concurrency model is like a dream, and its memory footprint is tiny. So, it was a natural choice for creating this API that I could use to manage my Raspberry Pi efficiently.

## A Familiar Architecture with a Touch of Beauty

Since I’m a big fan of Django (mainly because of its simplicity and the beauty of its architecture), I decided to use something that resembled it but was a bit more lightweight and efficient. Enter Gin, a Go framework that gave me exactly what I was looking for: a clean and simple architecture that allowed me to structure the app like Django would, without all the bloat.

Each service in my API is neatly divided into its own controller, and I used a REST architecture to keep everything flexible and easily extendable. For example, the system monitoring routes are beautifully handled with just a few lines of code. Here’s a look at the code for setting up routes:

```go
package routes

import (
    "github.com/gin-gonic/gin"
    "raspberry-controller/system_monitor/controllers"
)

func SetupSystemMonitorRoutes(r *gin.Engine) {
    systemMonitorGroup := r.Group("/system-monitor")
    {
        systemMonitorGroup.GET("/", controllers.TestService)
        systemMonitorGroup.GET("/ram-usage", controllers.RamUsageService)
        systemMonitorGroup.GET("/cpu-usage", controllers.CpuUsageService)
        systemMonitorGroup.GET("/system-info", controllers.SystemInfoService)
        systemMonitorGroup.GET("/cpu-temperature", controllers.CpuTemperatureService)
        systemMonitorGroup.POST("/shutdown", controllers.ShutdownSystemService)
    }
}

```

This code sets up all the routes for system monitoring, from RAM usage to CPU temperature, and even allows me to send a shutdown command via HTTP. For a quick reference, here's a breakdown of the models I’m using to keep track of the data:

```go
package models

type RamUsage struct {
    TotalMB     uint64  `json:"totalMB"`
    UsedMB      uint64  `json:"usedMB"`
    FreeMB      uint64  `json:"freeMB"`
    UsedPercent float64 `json:"usedPercent"`
}

type CpuUsage struct {
    CpuUsagePercent float64 `json:"cpuUsagePercent"`
}

type SystemInfo struct {
    Hostname           string `json:"hostname"`
    Uptime             uint64 `json:"uptime"`
    BootTime           uint64 `json:"bootTime"`
    OS                 string `json:"os"`
    Platform           string `json:"platform"`
    PlatformFamily     string `json:"platformFamily"`
    PlatformVersion    string `json:"platformVersion"`
    KernelVersion      string `json:"kernelVersion"`
    VirtualizationSystem string `json:"virtualizationSystem"`
    VirtualizationRole   string `json:"virtualizationRole"`
}

type CpuTemperature struct {
    CpuTemperature float64 `json:"cpuTemperature"`
}

```

These models track critical system information such as RAM usage, CPU temperature, and more, making it easy to monitor the health of my Raspberry Pi from anywhere.

## What Really Matters: The Application

What led me to create this app wasn’t an ambitious project or the need for a robust solution (though the architecture is solid enough that I could have scaled it up significantly). What really motivated me was my laziness. Yes, sitting there typing SSH commands like docker-compose up or shutdown -h now was taking up too much of my time. And while I’m not one to make excuses, this was a great reason to automate everything.

The main services of the app are:

### Docker Container Management

I can start and stop containers (such as `minidlna`, `transmission`, etc.) with simple HTTP calls. This is all powered by a series of Go commands that leverage Docker’s API to check container statuses, and if they’re not running, I start them with `docker-compose up`.

```go
cmd := exec.Command("docker-compose", "up", "-d", "minidlna")
```

### RAM and CPU Usage

Aside from controlling the containers, I also have basic system information like RAM usage, CPU load, and CPU temperature. After all, it never hurts to know if your Raspberry Pi is about to turn into an oven.

```go
type RamUsage struct {
    TotalMB     uint64  `json:"totalMB"`
    UsedMB      uint64  `json:"usedMB"`
    FreeMB      uint64  `json:"freeMB"`
    UsedPercent float64 `json:"usedPercent"`
}
```

This allows me to monitor the system’s health directly from my app. I can keep an eye on the temperature, the resources being consumed, and ensure nothing is overloading the Pi.

## Automation: The Beauty of Build and Deploy Scripts

While the architecture and API structure are great, the real fun begins when it comes to automation. The most satisfying part of this whole project was creating a smooth and automated build and deploy process. With just a few commands, I could compile the app, build the Docker containers, and deploy everything to the Raspberry Pi.

### deploy.sh: The Simplicity of Building

The build script (`deploy.sh`) is a beautiful thing. It takes the compiled file and deploy including the daemon:

```sh
#!/usr/bin/env zsh

# Solicitar la contraseña una sola vez
echo -n "Ingrese la contraseña SSH: "
stty -echo
read SSHPASS
stty echo
echo

# Detener el daemon raspberry-controller
echo "Deteniendo el daemon raspberry-controller..."
sshpass -p "$SSHPASS" ssh pi@incredibleipaddress 'sudo systemctl stop raspberry-controller' && \
echo "Daemon detenido." && \

# Copiar el nuevo controlador
echo "Copiando el nuevo controlador..."
sshpass -p "$SSHPASS" scp build/raspberry-controller pi@incredibleipaddress:/home/pi/raspberry-controller && \
echo "Controlador copiado." && \

# Reiniciar el servicio
echo "Reiniciando el servicio raspberry-controller..."
sshpass -p "$SSHPASS" ssh pi@incredibleipaddress 'sudo systemctl start raspberry-controller' && \
echo "Servicio reiniciado."
```

It’s as simple as that. The script makes sure the built service is ready to go.

## The Android App: A Simple Interface to Control It All

In addition to the API, I created a mobile app to interact with the system even more easily. I decided to use Jetpack Compose for the Android app, which allowed me to design the UI without the hassle of XML. The app provides a clean, simple interface to control the Raspberry Pi, monitor its health, and manage its services.

The beauty of this Android app is that it brings together the power of automation and system monitoring in one sleek interface. You can:

- Check RAM usage
- Monitor CPU temperature
- Control Docker containers
- Shutdown the system

All of this, controlled from your mobile device with a touch of a button. It’s efficient, fast, and really fun to use.

## Final Thoughts: Why I Built This

So why did I go through all of this trouble? Well, because I was bored, honestly. It was a fun project, and in the end, it gave me something incredibly useful. I don’t need to mess around with SSH or type out commands to manage my Raspberry Pi anymore. I can do everything with just a few taps or script executions.

At the end of the day, this app is a reflection of how lazy and efficient I can be. Why waste time on repetitive tasks when you can automate them? If you feel the same way and your Raspberry Pi needs a little extra automation, this might be just what you need.