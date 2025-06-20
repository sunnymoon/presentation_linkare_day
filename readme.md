# 1. Install crc

## 1.1 Download the binaries and pull secret
Access https://console.redhat.com/openshift/create/local 
Note: You might have to register (free) for a redhat account, otherwise, log in with your redhat account

> Download what you need to get started 
> Choose: Linux!!! :) x86_64 and you have two buttons 
> Download Openshift Local  -> we will not do this in the presentation because it takes long 
> Download Pull Secret -> store it in file pull-secret.txt on the same path as the crc binary 

## 1.2 Unpack the binary 
tar -zxvf crc-linux-amd64.tgz
(check the proper name of your dir)
ln -s crc-linux-2.46.0-amd64 crc-linux


## 1.3 installing OS requirements
> sudo dnf install libvirt NetworkManager 

## 1.4 Avoid telemetry (optional - adjust for your own machine)
> ./crc-linux/crc config set consent-telemetry
> ./crc-linux/crc config set memory 20480
> ./crc-linux/crc config set cpus 6
> ./crc-linux/crc config set disk-size 200

(check with crc config view)
./crc-linux/crc config view
consent-telemetry                     : no
cpus                                  : 6
disk-size                             : 200
memory                                : 20480
(this is stored in ~/.crc/crc.json) 

## 1.5. setup and start
> ./crc-linux/crc setup 
(this command will take long if never installed befora - or if you delete the cache!)
(o bundle é guardado em: $HOME/.crc/cache)

> ./crc-linux/crc start --pull-secret-file pull-secret.txt --disable-update-check



# 2. create github project (github cli)

# 3. create app (quarkus cli)

# 4. deploy in crc (s2i)

# 5. odo