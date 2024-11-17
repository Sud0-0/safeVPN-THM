# TryHackMe safe VPN access
iptables rules to only have incoming connections from the machine on TryHackMe

usage:

```bash
sudo chmod +x ./safevpn-thm.sh
sudo ./safevpn-thm.sh <ip of deployed machine>
```
example:

```bash
sudo ./safevpn-thm.sh 10.10.10.10
```


# Cambios
Validación de entrada ($1)
Agregué un chequeo para asegurarse de que $1 sea una dirección IP válida antes de aplicar las reglas. Esto evita que el script falle o aplique reglas incorrectas.

```bash
if [[ -z "$1" || ! "$1" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    echo "Error: Proporcione una dirección IP válida como argumento."
    exit 1
fi
```
# Bloqueo de tráfico no iniciado (DROP para conexiones nuevas/invalidas)
Agregué una regla para bloquear cualquier conexión entrante nueva o malformada que no haya sido iniciada por tu máquina

```bash
iptables -A INPUT -m state --state NEW,INVALID -j DROP
```
# Registro de tráfico bloqueado
Añadí reglas de LOG para registrar paquetes bloqueados tanto en entrada como en salida. Esto es útil para monitorear intentos de conexión no deseados

```bash
iptables -A INPUT -j LOG --log-prefix "Paquete bloqueado: " --log-level 4
iptables -A OUTPUT -j LOG --log-prefix "Salida bloqueada: " --log-level 4
```

# Manejo de tráfico DNS
Agregué reglas para permitir tráfico DNS hacia y desde servidores de nombres

```bash
iptables -A OUTPUT -o tun0 -p udp --dport 53 -j ACCEPT
iptables -A INPUT -i tun0 -p udp --sport 53 -j ACCEPT
```
# Persistencia de reglas
Sugerí usar iptables-save para guardar las reglas y restaurarlas automáticamente tras un reinicio

```bash
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
```
# Compatibilidad con IPv6
Para mayor seguridad, sugerí deshabilitar IPv6 completamente si no lo usas

```bash
echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
```
