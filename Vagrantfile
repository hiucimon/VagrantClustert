# -*- mode: ruby -*-
# vi: set ft=ruby :
 
=begin
Vagrant linux cluster code
Copyright 2018, Joseph Bryant & Capital One
Apache 2.0 License
=end
 
=begin
Global setting block
---------------------------------------------------------------
These variable control what is stood up in the cluster
$box=                   What vagrant box to use for each Linux instance
$memory=                          How many Meg of memory to allocat to each instance
$clustername=  The prefix name of the cluster
$clustercount=  How many total nodes in the cluster
$masternodes= How many nodes are for the Master nodes
$localport=[]      Array of ports to expose
$segment=                         Nextwork segment "x.x.x." for private net
$ipstart=                              What IP to start ap on $segment
=end
$box="hiucimon/Ubuntu1604ZFSRoot"
$memory=1024
$clustername="node"
$clustercount=3
$masternodes=3
$localport=[80,8500,4646]
$segment="192.168.99."
$ipstart=201
 
=begin
required global declarations
=end
$hosts={}
$nodename=[]
$nodes=[]
$externalport=[]
$etc_hosts=""
$nodeindex={}
 
(0..$clustercount-1).each do |index|
  ind=(1+index)*10000
  tmp=[]
  $localport.each do |port|
    tmp.push(port+ind)
  end
  $externalport.push(tmp)
  ip=$ipstart+index
  $nodes[index] = "#{$segment}#{ip}"
  $nodename[index] = "#{$clustername}#{index+1}"
end
 
(0..$nodename.length-1).each do |index|
  $nodeindex[$nodename[index]]=index
end
 
(0..$nodes.length-1).each do |index|
  if index==$masternodes-1 then
    mn="last_master_node.localdomain"
  else
    mn=""
  end
  if index==0 then
    fn="first_node.localdomain"
  else
    fn=""
  end
  if index==$nodes.length-1 then
    ln="last_node.localdomain"
  else
    ln=""
  end
  $etc_hosts+="#{$nodes[index]}\t#{$nodename[index]}\t#{$nodename[index]}.localdomain\t#{mn}\t#{fn}\t#{ln}\n"
end
 
Vagrant.configure("2") do |config|
                config.vm.box = $box
                $nodename.each do |node|
                                config.vm.provision "shell", inline: "echo >>/etc/hosts '#{$etc_hosts}'"
                                config.vm.define node do |cluster|        
                                                p=$localport[$nodeindex[node]]
                                                (0..$localport.length-1).each do |portindex|
                                                                cluster.vm.network :forwarded_port, guest: $localport[portindex], host: $externalport[$nodeindex[node]][portindex], auto_correct: true
                                                end
                                                cluster.vm.network "private_network",  name: "vboxnet0", auto_config: true, ip: $nodes[$nodeindex[node]]
                                                cluster.vm.provider "virtualbox" do |v|
                                                v.customize ["modifyvm", :id, "--memory", "#{$memory}"]
                                                v.name = node
                                                end
                                end
                end
                config.vm.provision "ansible" do |ansible|
                  ansible.playbook = "playbook.yml"
                end
end
