# Installation

```
git clone https://github.com/atareao/self-hosted.git
cd self-hosted/zinc
mv sample.env .env
sed -i "s/zinc.tuservidor.es/el_fqdn_que_quieras/g" .env
mkdir data
sudo chown -R 10001:10001 data
```

Si modificas las credenciales y quieres utilizar el archivo sample.http, necesitas cambiar el TOKEN.

A la hora de levantar el servicio dependerá del proxy inverso que hayas seleccionado. Si has elegido Caddy, simplemente,


```
echo -n admin:Compolexpass#123 | base64
```

```
docker-compose -f docker-compose.yml -f docker-compose.caddy.yml up -d
docker-compose logs -f
```

Mientras que si has elegido Traefik,

```
docker-compose -f docker-compose.yml -f docker-compose.traefik.yml up -d
docker-compose logs -f
```
