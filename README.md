# k3s-homelab
Welcome to my GitOps-powered lab of (mostly) controlled Kubernetes chaos! This repo documents my thrilling descent into managing a K3s cluster with FluxCD, where MetalLB, Traefik, and Cert-Manager all try their best not to explode at the same time. If YAML is your love language and you believe in the DevOps prophecy of "everything as code," you're in the right dimension. Expect battle scars, and the occasional commented-out cry for help.
FluxCD keeps everything synchronized like a slightly neurotic librarian, and Helm releases are the dusty grimoires of this digital dungeon. Contributions, spells, and debugging rituals welcome... but maybe too late.

openSUSE makes a rock-solid foundation for a K3s cluster, especially when paired with the dark magic of AutoYaST for fully automated, reproducible node deployments. It's like running your cluster on a resilient, self-healing spellbook. Combine that with GitOps on top, and you've got automation inception - auto-deployed nodes running auto-managed apps, all summoned by YAML incantations. Just don’t fat-finger your manifests, or the automation gods will smite your cluster faster than you can say kubectl -f. Remember: with great YAML comes great responsibility - and even greater downtime if you mess it up.

Assuming "The Nodes" are up and running in their desired, init state...

NODE1:
zypper in -y helm which  
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --cluster-init" K3S_TOKEN=k3stoken sh -s - server  
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> .bashrc  
echo "alias k=kubectl" >> .bashrc  
curl -s https://fluxcd.io/install.sh | bash  
  
export GITHUB_TOKEN=github_pat_which_is_secure_AF!_j3zo4w7T6IIABRweLX  
flux bootstrap github --owner=lyynxxx --repository=k3s-homelab --path=cluster --private=false --personal=true --components-extra=image-reflector-controller,image-automation-controller --read-write-key  
  
  
  
NODE[2345]:  
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode=644 --disable=service-lb --disable=traefik --server https://192.168.69.20:6443" K3S_TOKEN=k3stoken sh -s - server  
