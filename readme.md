# 1. Install crc
➜ bin crc config view
consent-telemetry                     : yes
cpus                                  : 6
disk-size                             : 200
memory                                : 20480
(this is stored in ~/.crc/crc.json)

o bundle é guardado em: /home/aleitao/.crc/cache

## 1.1 installing requirements
sudo dnf install libvirt NetworkManager

## 1.2. download openshift local
https://console.redhat.com/openshift/create/local

"Copy to path".../usr/local/bin for instance

## 1.x. setup and start
crc setup && crc start

# 2. create github project (github cli)

# 3. create app (quarkus cli)

# 4. deploy in crc (s2i)

# 5. odo