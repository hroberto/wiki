# Docker- Guia


## 1. Instalando o Docker

```bash
# instalando prerequisito
sudo apt-get install -y  apt-transport-https ca-certificates curl gnupg lsb-release

# registrando repositorio
curl --insecure -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg ; echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo 'Acquire::https::download.docker.com::Verify-Peer "false";' | sudo tee /etc/apt/apt.conf > /dev/null


# iniciando instalacao
sudo apt clean ;sudo apt update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# configurando 
sudo usermod -aG docker ${USER}
sudo chmod 666 /var/run/docker.sock
id -nG

# inicializando o servico do docker
## sudo systemctl status docker 
sudo /etc/init.d/docker  start

```

## 2. Habilitacao da rede

> /etc/docker/daemon.json
```json
{
    "bip": "192.168.199.1/28",
    "dns": ["8.8.8.8", "8.8.4.4"]
}
```


## 3. Limpando configurações nao utilizadas

```bash
docker system prune -a
```

