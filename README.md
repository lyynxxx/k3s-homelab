# Directory Structure (not in alphabetical order...)
```bash
├── README.md : ❗ YOU ARE HERE <---  
├── cluster : dominium of FluxCD. After bootstrap this is the folder, where FluxCD lives.  
├── helmrepositories : all the source definitions we plan to use in our cluster with FluxCD.  
├── infra : manifests for our core infrastructure only (Traefik, MetalLB, cert-manager)  
│   ├── config : different infra related configs, like: Traefik middleware, custom services with ip filter, MetalLB pool definitions, etc.  
│   └── controller : namespace definition, HelmRelease installation with custom values for infra elements  
├── secrets : our encrypted secrets, organized per namespace  
│   └──<namespace>  
├── templates : FluxCD templates (for example reusable, isolated namespace definition)  
├── apps : all the app definitions  
└── docs: lastly, the archives... all the suffering and magic  
	├── 1_autoyast : Swing your wand and auto-deply openSuse, install the k3s cluster  
	├── 2_cluster : The birth of FluxCD, to rule them all  
	├── 3_sops : Storing passwords and sensitive data in encrypted secrets, in Git.  
	└── 4_apps : And the fun begins... let's host our apps  
```

# k3s-homelab
Welcome to my GitOps-powered lab of (mostly) controlled Kubernetes chaos! This repo documents my thrilling descent into managing a K3s cluster with FluxCD, where MetalLB, Traefik, and Cert-Manager all try their best not to explode at the same time. If YAML is your love language and you believe in the GitOps prophecy of "everything as code," you're in the right dimension. Expect battle scars, and the occasional commented-out cry for help.
FluxCD keeps everything synchronized like a slightly neurotic librarian, and Helm releases are the dusty grimoires of this digital dungeon. Contributions, spells, and debugging rituals welcome... but maybe too late.

Before we start conjuring clusters and binding them to Git, a word of caution: this is not a scroll for absolute beginners. To follow along without summoning chaos, you should already speak fluent YAML, know your way around a terminal, and have at least shaken hands with Kubernetes before. We're skipping the “what is a pod?” introductions and diving straight into GitOps incantations, infrastructure rituals, and the unfrequent ZFS snapshot rollback...  

My preferred OS of choice is openSUSE. openSUSE makes a rock-solid foundation for a K3s cluster, especially when paired with the dark magic of AutoYaST for fully automated, reproducible node deployments. It's like running your cluster on a resilient, self-healing spellbook. Combine that with GitOps on top, and you've got automation inception - auto-deployed nodes running auto-managed apps, all summoned by YAML incantations. Just don’t fat-finger your manifests, or the automation gods will smite your cluster faster than you can say kubectl -f. Remember: with great YAML comes great responsibility - and even greater downtime if you mess it up.

*Disclaimer, adventurer: I chose openSUSE for this quest because I like it, and I have... creative disagreements with RHEL... IBM (very, very much so). But whether you ride with Debian, Arch (which btw I use too :) ), or some cursed RPM-based fork, which is inside the Matrix - you know, the support one - and stuff works, the principles of GitOps remain the same. Use the distro you love, slay your cluster demons your own way—the outcome is what truly matters.*

> [:arrow_right: **Step 1: AutoYast:** A mystical rite that breathes life into metal, conjures fully armed and operational nodes from the void, no clicks, no questions, just pure scripted destiny. Let us begin the first incantation...](docs/1_autoyast)

A three-node server cluster hits the sweet spot for a homelab - it’s just enough to simulate the glorious chaos of a real Kubernetes environment without needing a data center or sacrificing your living room to racks and fans. With embedded etcd cluster, you get the real control plane experience in all its distributed, high-availability glory, minus the overhead of running an external etcd stack that requires more ceremony than a royal wedding. It’s stable, it behaves (mostly), and it lets you break things with impunity while still teaching you what a production-grade system *feels* like. Think of it as Kubernetes on "medium heat" - hot enough to burn you, but not enough to melt your face off.

That said, once you venture beyond the comforting chaos of your homelab and into the wild world of enterprise clusters, you’d best leave the training wheels behind. Following best practices in production isn’t just a checkbox — it’s the difference between “Oops, my blog is down” and “Khm... Houston we have a problem... I just cost the company $300k in SLA penalties.” So yes, three server nodes only is perfect for learning the ropes, or host a few, non critical apps, but if you take it to EE production, may your pager be loud and your coffee strong and have dedicated master and worker nodes, node taints and other advanced magic.

> [:arrow_right: **Step 2: Cluster init:** A double invocation of power — K3s awakens the cluster’s heart, and FluxCD binds it to the Git-bound plane, where manifests rule and kubectl becomes but a backup spell....](docs/2_cluster)

From this moment on, Git is the single source of truth, and git commit is your wand. Welcome to the age of true GitOps, or dare I say... welcome to the "I want GitOps but I don't want my passwords on GitHub" club! 😄

> [:arrow_right: **Step 3: Encrypting secrets:** A double invocation of power — K3s awakens the cluster’s heart, and FluxCD binds it to the Git-bound plane, where manifests rule and kubectl becomes but a backup spell....](docs/3_sops)


* There’s something undeniably beautiful about learning new things — especially in the world of infrastructure, where every new tool or technique feels like unlocking a fresh layer of reality. It’s a strange kind of thrill: the excitement of cracking a complex problem, the adrenaline rush when something finally works, and yes, even the despair of debugging YAML that stares back at you with silent judgment. This path will take you into dark places - broken containers, lost packets, and mysterious logs — but with every stumble, you gain wisdom, confidence, and scars that quietly whisper "I’ve seen some real sht!" And while your future self may not remember every flag or workaround, your CV absolutely will. Because what starts as curiosity often ends as capability, and there's real power in being the person who can say, "Yeah, I’ve automated that."

...but no matter how you look at it... My mana is spent, my scrolls are smoldering, and even my staff refuses to stand upright. Let the logs roll where they may — I must rest, or risk turning into a null reference myself.
*

