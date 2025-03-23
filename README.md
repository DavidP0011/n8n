# Guía de instalación de **n8n** en **GCP**

### 1. Actualizar la máquina virtual
```bash
sudo apt update
```
---

### 2. Instalar Docker
```bash
sudo apt install docker.io
```
> Docker permite empaquetar y ejecutar n8n con todas sus dependencias, asegurando que pueda replicarse sin problemas.

---

### 3. Iniciar Docker
```bash
sudo systemctl start docker
```
---

### 4. Habilitar Docker al inicio
```bash
sudo systemctl enable docker
```
> Esto asegura que Docker se inicie automáticamente cada vez que se encienda el servidor.

---

### 5. Ejecutar n8n con Docker
```bash
sudo docker run -d --restart unless-stopped -it \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST="n8nagent.animum-service.cloud" \
  -e WEBHOOK_TUNNEL_URL="https://n8nagent.animum-service.cloud/" \
  -e WEBHOOK_URL="https://n8nagent.animum-service.cloud/" \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```
- `-p 5678:5678`: expone el puerto 5678 del contenedor en el host.  
- `-e N8N_HOST`, `-e WEBHOOK_TUNNEL_URL` y `-e WEBHOOK_URL`: establecen las variables de entorno para que n8n funcione con el dominio `n8nagent.animum-service.cloud`.  
- `-v ~/.n8n:/root/.n8n`: asigna un volumen local para persistir datos de n8n en `~/.n8n` del host.

---

### 6. Instalar Nginx
```bash
sudo apt install nginx
```

---

### 7. Crear/editar el archivo de configuración de Nginx
```bash
sudo nano /etc/nginx/sites-available/n8n
```
> Se define este archivo para que Nginx actúe como proxy inverso de n8n.

---

### 8. Configuración del bloque `server` en Nginx
```nginx
server {
    listen 80;
    server_name n8nagent.animum-service.cloud;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
    }
}
```
- `listen 80;` indica que Nginx escuchará en el puerto 80.  
- `server_name n8nagent.animum-service.cloud;` define el dominio.  
- `proxy_pass http://localhost:5678;` redirige el tráfico a n8n.

---

### 9. Habilitar la configuración
```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
```
> Se crea un enlace simbólico en el directorio `sites-enabled` para habilitar el sitio.

---

### 10. Comprobar la configuración de Nginx
```bash
sudo nginx -t
```
> Si hay errores, aparecerán aquí antes de reiniciar el servicio.

---

### 11. Reiniciar Nginx
```bash
sudo systemctl restart nginx
```
> Aplica los cambios de configuración.

---

### 12. Instalar Certbot y su plugin para Nginx
```bash
sudo apt install certbot python3-certbot-nginx
```
> Certbot facilita la obtención de certificados SSL/TLS.

---

### 13. Obtener e instalar el certificado SSL
```bash
sudo certbot --nginx -d n8nagent.animum-service.cloud
```
> Este comando configura automáticamente un certificado SSL para el dominio indicado.

---

### 14. Ajustar configuración de Nginx para WebSockets
```bash
sudo nano /etc/nginx/sites-available/n8n
```
> Dentro del bloque `location /`, añadir:
```nginx
proxy_set_header Connection 'Upgrade';
proxy_set_header Upgrade $http_upgrade;
```
> Estas directivas permiten que las conexiones WebSocket funcionen correctamente.

---

### 15. Reiniciar Nginx (importante)
```bash
sudo systemctl restart nginx
```
> Paso esencial para aplicar las modificaciones recién realizadas.

---

**Dirección IP del servidor**: `34.175.50.92`# n8n
