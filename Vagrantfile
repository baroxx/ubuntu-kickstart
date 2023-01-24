BASE_BOX = "ubuntu/focal64"
BOX_VERSION = "20230110.0.0"
CPUS = 2
MEMORY = "4096"
USER_NAME = "ubuntu"
PASSWORD = "ubuntu"
KEYMAP = "de"

KUBERNETES_VERSION="1.24.4"
CP_IP="192.168.56.10"
POD_SUBNET_CIDR="192.168.0.0/16"
NODE_IP_RANGE="192.168.56." # keep the last number empty
NUMBER_OF_NODES = 1

Vagrant.configure("2") do |config|

    config.vm.provider "virtualbox" do |virtualbox|
        virtualbox.cpus = CPUS
        virtualbox.memory = MEMORY
    end

    config.vm.define "k8s-cp" do |cp|
        cp.vm.box = BASE_BOX
        config.vm.box_version = BOX_VERSION

        cp.vm.network "private_network", ip: CP_IP
        cp.vm.hostname = "k8s-cp"

        cp.vm.provision "main", type: "shell", args: [USER_NAME, PASSWORD, KEYMAP], path: "provisioner/main.sh"
        # Container runtime
        cp.vm.provision "crio", type: "shell", path: "provisioner/crio.sh"
        #cp.vm.provision "containerd", type: "shell", path: "provisioner/containerd.sh"
        cp.vm.provision "kubernetes", type: "shell", args: [USER_NAME, KUBERNETES_VERSION], path: "provisioner/kubernetes.sh"
        cp.vm.provision "control", type: "shell", args: [CP_IP, CP_IP, POD_SUBNET_CIDR, USER_NAME, KUBERNETES_VERSION], path: "provisioner/control.sh"
        # CNI
        cp.vm.provision "calico", type: "shell", path: "provisioner/calico.sh"
        cp.vm.provision "final", type: "shell", args: [USER_NAME], path: "provisioner/final.sh"
    end

    (1..NUMBER_OF_NODES).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = BASE_BOX
            config.vm.box_version = BOX_VERSION

            node.vm.network "private_network", ip: NODE_IP_RANGE+"#{i + 10}"
            node.vm.hostname = "node-#{i}"

            node.vm.provision "main", type: "shell", args: [USER_NAME, PASSWORD, KEYMAP], path: "provisioner/main.sh"
            # Container runtime
            node.vm.provision "crio", type: "shell", path: "provisioner/crio.sh"
            #node.vm.provision "containerd", type: "shell", path: "provisioner/containerd.sh"
            node.vm.provision "kubernetes", type: "shell", args: [USER_NAME, KUBERNETES_VERSION], path: "provisioner/kubernetes.sh"
            node.vm.provision "worker", type: "shell", args: [CP_IP, NODE_IP_RANGE+"#{i + 10}"], path: "provisioner/worker.sh"
            node.vm.provision "final", type: "shell", args: [USER_NAME], path: "provisioner/final.sh"
        end
    end

end
  