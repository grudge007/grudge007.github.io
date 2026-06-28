# Why I Built Incubator

As I said in my last blog about [Vulta](https://github.com/grudge007/vulta), I was always curious about learning how things work under the hood. I even tried reading the Linux TCP stack code from GitHub even though I didn't know much C.

## Discovering Containers and Dooker

I started learning about Docker, but I was not satisfied because I had no clue what Docker actually was. For a long time, since my initial weeks as a cyber security student, I thought Docker was some fresh technology built from nothing.

When I finally had to work with Docker, I started researching how it works and found that it is mostly using Linux kernel internals like namespaces and cgroups. I hate Windows and cannot afford a Mac, so Linux was my only choice anyway. Around the same time, I also discovered LXC.

The same day, I built a script to create a container using the `unshare` command, Open vSwitch and Linux bridges for networking, and a Debian root filesystem. I named it **Dooker**.

I had already started learning Golang, so I decided to rewrite Dooker in Go. My dream became too big, the architecture became even bigger, and the Go I knew at that time was definitely not ready for it. My day job was not Go-heavy either, and between work and travel I did not have enough time to learn everything needed.

So I decided to stop working on it after a couple of days. Still, I was happy because my original goal was achieved. I finally understood what Docker actually was, and that "I know the commands but have no clue what is happening underneath" problem was solved.

## Vulta

After that, I had to work on a few remote nodes regularly. My workflow was basically:

```text
write script -> tar -> scp -> untar -> run -> fail -> rewrite -> tar -> scp -> untar
```

On my third attempt to improve this workflow, I first tried shell scripts and then Ansible, but Ansible felt like too much for my small use case.

So I started working on [Vulta](https://github.com/grudge007/vulta), my mini-Ansible project that could push files to remote nodes.

It eventually grew into something much bigger. I ended up learning about state tracking, goroutines, pointers, references, and many other Go concepts. Implementing state tracking was especially interesting because it helped me understand how tools like Git detect changes and how ignore rules work internally.

Vulta was also the first application I built with the help of AI. AI did not write the application for me, but it helped me discover libraries and approaches. Before the final release, I asked Antigravity to improve logging and readability, and since Vulta already satisfied both of my goals, learning and building tooling, I decided to keep those improvements.

## OpenZiti Experiment

I was still not done exploring things.

My next plan was to build something useful for the open source community. Many of my previous projects ended up being tools mainly for myself, and Vulta was no different. Even when I tried making colleagues use it, most of them were perfectly happy with `tar`, `scp`, and `untar`. Still, I never regretted building it.

Later, I accidentally heard one of my seniors talking about OpenZiti. I became curious and started experimenting with it.

The idea was to build a Docker network plugin using OpenZiti. I manually tested everything using an OpenZiti server running on my VPS and Docker services running locally.

What amazed me was that I could create multiple Docker services and access them through OpenZiti identities and service names without worrying about container IPs or exposed ports. Clients only needed access to the service identity, and OpenZiti policies handled the rest.

I managed to prove the concept using Bash scripts, but implementing it as a proper Docker plugin in Go turned out to be harder than I expected. Docker plugin internals involved many handshakes and details that I was not yet comfortable with. Also, I am not a Go engineer by profession, so I decided to postpone the project. It is still on my list, and I believe I can finish it someday.

## Birth of Incubator

Talking about Incubator, it was initially planned as something built around LXD. But I quickly found that LXD was already too complete, and I could not think of much to build around it, or maybe I simply did not know enough to contribute meaningfully.

So I changed the plan and started building a VMM around QEMU. I had already spent a lot of time working with virtualization platforms, so understanding the virtualization stack itself felt like the next rabbit hole to jump into.

I started with `os/exec` and completed the first phase with basic operations like create, delete, start, stop, list, and JSON-based metadata.

Soon I became uncomfortable with JSON files because manual modifications could easily corrupt metadata. So I moved everything to SQLite and reorganized the codebase by defining clear components and responsibilities. I also kept the storage layer separate so that replacing SQLite with something like etcd in the future would be easier.

I also brought Open vSwitch into the project as the first step toward my long-term goal of eventually integrating OVN. I was slightly disappointed that Proxmox did not support OVN natively, so this became another excuse to build things myself.

As the project grew, I decided to daemonize it and separate the CLI from the backend. The CLI and daemon now communicate over Unix sockets.

This architectural shift was heavy. Separating the CLI and daemon alone required significant changes, and sometimes it honestly feels like rewriting the project from scratch would be easier than modifying the existing code.

The first two versions of [Incubator](https://github.com/grudge007/incubator) are available on GitHub, and I am currently redesigning parts of the socket-based communication. Once I fully understand the new design, I think I can finish the remaining parts fairly quickly. Famous last words, probably.

Incubator is still evolving, but it has already taught me much more about virtualization internals, software architecture, and system design than I originally expected.
