# Why I Built Vulta

Ever since I joined the IT field, I always had this obsession with knowing how things actually work. **What is under the hood? How was this built?** Questions like these took a lot of my time. Sometimes it was frustrating because whenever I found a tool interesting, the next thing I would think about was how they built it and whether I could build my own version.

People usually say *“don’t reinvent the wheel,”* but I used to do exactly that. Most of the time, once I understood how something was built, I would stop halfway because I realized the complexity involved and started questioning whether it was worth the time.

I am mostly a **localhost person**. Before testing anything on a server, VM, staging, or production environment, I always test it locally first. After that, my usual workflow was simple: **tar** the entire codebase, **SCP** it to the target machine, **untar** it there, and run it.

Sometimes I use Git, but Git was not always the best option for me. Internet access is not always available everywhere, and in some environments local Git usage is restricted or protected with rules. So my **tar + SCP** loop was the most reliable workflow for me.

But when you are actively working on something and constantly making changes, doing this loop again and again becomes annoying very quickly. So I started with a small shell script to automate it. My workflow became:

```bash
modify code -> ./tar-scp.sh

```

That already made life easier.

Later I tried **Ansible**, but for my specific use case it felt like too much. It is a powerful tool, but I only wanted a fast and simple way to push files and run commands. I also noticed some of my colleagues were doing similar manual workflows, so for me it felt worth building a tool around this idea.

That is how **Vulta** started.

The original idea was simple: configure some remote nodes and push files from a local directory to all of them easily. I used a **YAML** configuration file where I could define node IPs, users, remote paths, and related settings.

I built it in **Golang** mainly because I wanted to learn Go better. It also helped me move one step closer toward platform engineering, which was something I was interested in.

At first, the application only handled pushing files, and it worked pretty well. Later I wanted a centralized way to run commands on all configured nodes. So I added the `run` feature.

For example:

```bash
vulta run "ls /opt"

```

This would execute the command on all configured remote nodes and return the output from each server.

After using it more, I noticed another problem. Every `vulta push` was uploading all files again, even if nothing had changed. That made deployments slower than necessary, and honestly it did not feel right to consume bandwidth for unchanged files again and again.

So I added **hash tracking** to detect modified files and upload only the changed ones. That made pushes much faster and reduced unnecessary transfers between systems. I had already added support for `.vultaignore` files so directories like `node_modules`, `__pycache__`, and other unnecessary files could be skipped.

Eventually I added options to:

* Push only specific files
* Target only selected nodes
* Run commands on selected machines

At that point I redesigned the CLI using **Cobra** instead of the basic `os.Args` style parsing I started with.

Even though Vulta was open source, almost nobody knew about it. Even some of my colleagues preferred continuing the tar + SCP loop instead of learning another tool. So from my point of view, the project was both a success and a failure at the same time.

People did not really adopt it, but it took me far beyond printing “Hello World” in Go. I explored many libraries, learned how to structure a real CLI application, and built something I actually use regularly in my own workflow.

I also built a small website for it using **Antigravity IDE** and hosted it on my VPS at `vulta.iamgrudge.online`. Later I even created an **APT repository** so Vulta could be installed directly using `apt`.

Throughout this journey I enjoyed almost everything about building it. More importantly, I ended up with a working application that I genuinely use a lot.

One more thing worth mentioning is that **I used AI while building Vulta**. Mostly to understand Go syntax, explore ideas, and speed up learning. But I never blindly copy-pasted code just to finish the project quickly. The main goal was always learning how things actually work.
