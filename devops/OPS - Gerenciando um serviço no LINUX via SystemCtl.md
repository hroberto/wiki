# Gerenciando um serviço no LINUX


## Modelo de um serviço


```ini
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/MyApp
ExecStart=dotnet /opt/MyApp/AppService.dll
User=root
Group=root

[Install]
WantedBy=multi-user.target

```



## Publicando um novo serviço

```sh
sudo cp <sample-app.service>  /etc/systemd/system/
sudo systemctl daemon-reload
```


## Inicializando o serviço

```sh
# marca o serviço para a subida automatica
sudo systemctl enable <sample-app.service>

# inicializa o serviço
sudo systemctl start <sample-app.service>
```


## Visualiza os serviços disponiveis no servidor

```sh
sudo systemctl list-unit-files  *
```

## Verificar as saidas do serviço

```sh
# muda a saida para VERBOSE
sudo systemctl start  <sample-app.service>  --output=verbose

# visualizar o historico do serviço
sudo journalctl --unit <sample-app.service>
```
