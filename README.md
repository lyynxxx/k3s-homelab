# k3s-homelab
Learning how to do stuff...

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
