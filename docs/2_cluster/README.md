## Assuming "The Nodes" are up and running in their desired, init state...

On "NODE1" the initiation ritual begins with a single command—cryptic, yet powerful (especially as you call this as root):  
> curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --cluster-init" K3S_TOKEN=k3stoken sh -s - server  

This spell does many things at once: it calls upon the great K3s script-oracle from the internet, feeding it flags to shape our destiny. --cluster-init declares this node as the founding monarch of the cluster. --disable=service-lb and --disable=traefik banish K3s’s default load balancer and ingress demon, for we shall summon mightier ones - MetalLB and Traefik - in their place, under our terms. --write-kubeconfig-mode=644 ensures the kubeconfig is readable to the humble mortals who dare interact with it. And of course, K3S_TOKEN binds all future nodes into a brotherhood, swearing fealty to this first node’s reign. Thus, with one command, the cluster was born. Silent, headless, but very much alive.  

With the cluster heart beating and the first node crowned as master of its destiny, the next steps are ones of convenience and command—a touch of wizardry to ease the toil of the everyday kube-warrior.  

This line etches a binding spell into your shell’s arcane tome (.bashrc), ensuring that every new terminal knows where to find the sacred kubeconfig scroll—the map to the cluster’s soul. Without it, kubectl stumbles in the dark, lost and confused, unsure of which cluster it serves.
> echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> .bashrc  

And this one? A charm of brevity, reducing the mighty yet often mistyped kubectl to a single keystroke: k. Because real mages don’t waste time on extra letters when the pods are burning and the logs are pouring like rain.
> echo "alias k=kubectl" >> .bashrc  

*A small confession from the homelab archives: all this wizardry runs under the all-powerful root user. Yes, yes, it’s not exactly professional, and somewhere a security engineer just felt a disturbance in the Force. But in this realm, the goal is not to impress auditors; it's to eliminate terminal toil entirely. Once the cluster is summoned, all magic flows through Git - commit, push, and the occasional desperate revert are the only incantations we truly need. We are trying to understand and learn the principle only!*  


With the first node enthroned and the control plane awakened, it was time to summon the others - NODE2, NODE3... and NODEX. These were not mere workers, but full-fledged server nodes, sworn to share the burden of etcd and uphold the sacred uptime pact. Their role: reinforce the cluster’s spine, provide high availability, and—when things inevitably break—quietly suffer with dignity.

Each joined the cause with the following invocation:
> curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --server https://192.168.69.20:6443" K3S_TOKEN=k3stoken sh -s - server 

Here, --server https://192.168.69.20:6443 points each follower to the primary node’s API throne, letting them bind to the cluster’s will. The K3S_TOKEN acts as the shared secret handshake - no token, no entry, no exceptions. The --disable=service-lb and --disable=traefik flags once again cast out the default minions; we’ve got grander plans, remember? With one spell per node, the council of at least three was formed, an embedded etcd cluster ready to rule, replicate, and roll over for GitOps glory.

With the cluster assembled and the nodes sworn into high-availability service, it was time to unveil the true power behind the throne—the great automation daemon, the GitOps herald, the one who watches all: Flux.

Back on the first node, where kubeconfig and destiny align, we summon it with a single, deceptively simple command:

> curl -s https://fluxcd.io/install.sh | bash  

This is no ordinary curl-and-hope. This is the ritual that installs the Flux CLI, your personal conduit to the GitOps plane. It's the whispering staff you’ll use to bind your cluster to your Git repository, where manifests await like sleeping spells, ready to be applied, rolled back, or overwritten with terrifying efficiency. From this moment on, Git is law, Flux is the enforcer, and manual kubectl apply is a forgotten relic of the dark pre-GitOps era.  

Before we let Flux work its magic and bind the cluster’s fate to a Git repository, we must offer it a key—a [GitHub personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens). Why? Because even automation needs credentials to knock on GitHub’s door and say, “I’m here to sync your YAML and possibly your sanity.”

This token gives Flux the ability to read (and optionally write) to your repo, fetch manifests, watch for changes, and faithfully apply them to your cluster. Without it, Flux is just a powerful automation engine staring longingly at a private repo it can’t touch. Like a wizard locked out of their own tower.

> export GITHUB_TOKEN=github_pat_which_is_secure_AF!_j3zo4w7T6IIABRweLX

And now, the grand finale—the binding of realms. With Flux installed and the GitHub token in hand, it’s time to perform the bootstrap ritual, the final spell that ties your cluster’s fate to your Git repository:

> flux bootstrap github --owner=lyynxxx --repository=k3s-homelab --path=cluster --private=false --personal=true --components-extra=image-reflector-controller,image-automation-controller --read-write-key  

This command does more than just wave a wand, it forges the sacred link between your local cluster and the repository where your dreams (Kustomiztions and YAML manifests) live. It sets up Flux, commits the initial configuration, and deploys itself from the repo, like a snake eating its own tail but way more DevOps. With image-reflector-controller and image-automation-controller included, the cluster can now even watch for container updates and upgrade itself, because manual updates are for mere mortals. 