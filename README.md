# k3s-homelab
Welcome to my GitOps-powered lab of (mostly) controlled Kubernetes chaos! This repo documents my thrilling descent into managing a K3s cluster with FluxCD, where MetalLB, Traefik, and Cert-Manager all try their best not to explode at the same time. If YAML is your love language and you believe in the GitOps prophecy of "everything as code," you're in the right dimension. Expect battle scars, and the occasional commented-out cry for help.
FluxCD keeps everything synchronized like a slightly neurotic librarian, and Helm releases are the dusty grimoires of this digital dungeon. Contributions, spells, and debugging rituals welcome... but maybe too late.

My preferred OS of choice is openSUSE. openSUSE makes a rock-solid foundation for a K3s cluster, especially when paired with the dark magic of AutoYaST for fully automated, reproducible node deployments. It's like running your cluster on a resilient, self-healing spellbook. Combine that with GitOps on top, and you've got automation inception - auto-deployed nodes running auto-managed apps, all summoned by YAML incantations. Just don’t fat-finger your manifests, or the automation gods will smite your cluster faster than you can say kubectl -f. Remember: with great YAML comes great responsibility - and even greater downtime if you mess it up.

Disclaimer, adventurer: I chose openSUSE for this quest because I like it, and I have... creative disagreements with RHEL (very, very much so). But whether you ride with Debian, Arch (which btw I use too :) ), or some cursed RPM-based fork, the principles of GitOps remain the same. Use the distro you love, slay your cluster demons your own way—the outcome is what truly matters.

To avoid the ancient ritual of clicking "Next" a thousand times, I summoned the great [AutoYaST](https://doc.opensuse.org/documentation/leap/autoyast/single-html/book-autoyast/index.html) to preordain the fate of my nodes. With one sacred XML file, I scripted the full metal resurrection of each machine — disks partitioned with surgical precision, packages hand-picked like artisanal cheese, and system settings carved in stone before the OS even had a chance to ask questions. The installer didn't just boot, it *complied*, silently crafting perfect little minions without bothering me for a single keyboard stroke. Automation is beautiful when it obeys.

By the time the first boot was done, my nodes emerged like trained warhorses at the ready - configured, networked, SSH-ready, and morally prepared for the burden of running Kubernetes. No extra tinkering, no post-install patch-fu, no "oops I forgot to install curl" drama. Just pure, industrial-grade repeatability. Honestly, if your infra doesn't greet you with "Sir, I'm ready to sync with FluxCD and stuff" on first boot, are you even doing your job right?

A three-node server cluster hits the sweet spot for a homelab - it’s just enough to simulate the glorious chaos of a real Kubernetes environment without needing a data center or sacrificing your living room to racks and fans. With embedded etcd cluster, you get the real control plane experience in all its distributed, high-availability glory, minus the overhead of running an external etcd stack that requires more ceremony than a royal wedding. It’s stable, it behaves (mostly), and it lets you break things with impunity while still teaching you what a production-grade system *feels* like. Think of it as Kubernetes on "medium heat" - hot enough to burn you, but not enough to melt your face off.

That said, once you venture beyond the comforting chaos of your homelab and into the wild world of enterprise clusters, you’d best leave the training wheels behind. Real production workloads deserve external etcd, dedicated nodes, fencing mechanisms, and enough redundancy to survive not just a power outage but a mild apocalypse. Following best practices in production isn’t just a checkbox—it’s the difference between “Oops, my blog is down” and “Khm... Houston we have a problem... I just cost the company $300k in SLA penalties.” So yes, server ndoes only is perfect for learning the ropes, or host a few, non critical apps, but if you take it to EE production, may your pager be loud and your coffee strong and have dedicated master and worker nodes.


### Assuming "The Nodes" are up and running in their desired, init state...

On "NODE1" the initiation ritual begins with a single command—cryptic, yet powerful (especially as you call this as root):  
> curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --cluster-init" K3S_TOKEN=k3stoken sh -s - server  

This spell does many things at once: it calls upon the great K3s script-oracle from the internet, feeding it flags to shape our destiny. --cluster-init declares this node as the founding monarch of the cluster. --disable=service-lb and --disable=traefik banish K3s’s default load balancer and ingress demon, for we shall summon mightier ones - MetalLB and Traefik - in their place, under our terms. --write-kubeconfig-mode=644 ensures the kubeconfig is readable to the humble mortals who dare interact with it. And of course, K3S_TOKEN binds all future nodes into a brotherhood, swearing fealty to this first node’s reign. Thus, with one command, the cluster was born. Silent, headless, but very much alive.  

With the cluster heart beating and the first node crowned as master of its destiny, the next steps are ones of convenience and command—a touch of wizardry to ease the toil of the everyday kube-warrior.  

This line etches a binding spell into your shell’s arcane tome (.bashrc), ensuring that every new terminal knows where to find the sacred kubeconfig scroll—the map to the cluster’s soul. Without it, kubectl stumbles in the dark, lost and confused, unsure of which cluster it serves.
> echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> .bashrc  

And this one? A charm of brevity, reducing the mighty yet often mistyped kubectl to a single keystroke: k. Because real mages don’t waste time on extra letters when the pods are burning and the logs are pouring like rain.
> echo "alias k=kubectl" >> .bashrc  

*A small confession from the homelab archives: all this wizardry runs under the all-powerful root user. Yes, yes, it’s not exactly professional, and somewhere a security engineer just felt a disturbance in the Force. But in this realm, the goal is not to impress auditors; it's to eliminate terminal toil entirely. Once the cluster is summoned, all magic flows through Git - commit, push, and the occasional desperate revert are the only incantations we truly need.*  


With the first node enthroned and the control plane awakened, it was time to summon the others - NODE2, NODE3... and NODEX. These were not mere workers, but full-fledged server nodes, sworn to share the burden of etcd and uphold the sacred uptime pact. Their role: reinforce the cluster’s spine, provide high availability, and—when things inevitably break—quietly suffer with dignity.

Each joined the cause with the following invocation:
> curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --server https://192.168.69.20:6443" K3S_TOKEN=k3stoken sh -s - server 

Here, --server https://192.168.69.20:6443 points each follower to the primary node’s API throne, letting them bind to the cluster’s will. The K3S_TOKEN acts as the shared secret handshake - no token, no entry, no exceptions. The --disable=service-lb and --disable=traefik flags once again cast out the default minions; we’ve got grander plans, remember? With one spell per node, the council of at least three was formed, an embedded etcd cluster ready to rule, replicate, and roll over for GitOps glory.

With the cluster assembled and the nodes sworn into high-availability service, it was time to unveil the true power behind the throne—the great automation daemon, the GitOps herald, the one who watches all: Flux.

Back on the first node, where kubeconfig and destiny align, we summon it with a single, deceptively simple command:

> curl -s https://fluxcd.io/install.sh | bash  

This is no ordinary curl-and-hope. This is the ritual that installs the Flux CLI, your personal conduit to the GitOps plane. It's the whispering staff you’ll use to bind your cluster to your Git repository, where manifests await like sleeping spells, ready to be applied, rolled back, or overwritten with terrifying efficiency. From this moment on, Git is law, Flux is the enforcer, and manual kubectl apply is a forgotten relic of the dark pre-GitOps era.  
  
export GITHUB_TOKEN=github_pat_which_is_secure_AF!_j3zo4w7T6IIABRweLX  
flux bootstrap github --owner=lyynxxx --repository=k3s-homelab --path=cluster --private=false --personal=true --components-extra=image-reflector-controller,image-automation-controller --read-write-key  
  
  
 
