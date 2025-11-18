Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 2

    # Create 4 disks 400 MB each
    (1..4).each do |i|
      disk = "disk#{i}.vdi"
      vb.customize ["createhd", "--filename", disk, "--size", 400] 
      vb.customize [
        "storageattach", :id,
        "--storagectl", "SATA Controller",
        "--port", i,
        "--device", 0,
        "--type", "hdd",
        "--medium", disk
      ]
    end
  end

  config.vm.provision "shell", inline: <<-SHELL
    set -eux

    apt update -y
    apt install -y lvm2 parted

    # Partition disks
    for d in /dev/sd{b,c,d,e}; do
      parted --script $d mklabel gpt mkpart primary 1MiB 100%
    done

    # Create PVs
    pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1

    # Create VG
    vgcreate vgdata /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1

    # Create LV 780 MB each
    lvcreate -L 780M -n vol0 vgdata
    lvcreate -L 780M -n vol1 vgdata

    mkfs.ext4 /dev/vgdata/vol0
    mkfs.ext4 /dev/vgdata/vol1

    mkdir -p /mnt/vol0 /mnt/vol1

    echo "/dev/vgdata/vol0 /mnt/vol0 ext4 defaults 0 0" >> /etc/fstab
    echo "/dev/vgdata/vol1 /mnt/vol1 ext4 defaults 0 0" >> /etc/fstab

    mount -a
  SHELL
end