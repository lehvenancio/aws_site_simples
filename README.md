# 📄 Deploy de Site Estático no EC2 com Nginx e HTTPS

Este projeto é um site estático (como um currículo ou portfólio) usando **AWS EC2**, **Nginx**, **DuckDNS** e **HTTPS via Let’s Encrypt**, puxando os arquivos diretamente de um **repositório GitHub**.

---

## 1️⃣ Criar instância EC2

1. Acesse **EC2 → Launch Instance**.  
2. Escolha **Amazon Linux 2023**.  
3. Tipo de instância: `t2.micro` (Free Tier).  
4. Configure **Security Group**:

| Porta | Protocolo | Origem          |
|-------|-----------|----------------|
| 22    | SSH       | Seu IP         |
| 80    | HTTP      | 0.0.0.0/0      |
| 443   | HTTPS     | 0.0.0.0/0      |

5. Crie um **key pair** (`.pem`) e baixe-o.  

---

## 2️⃣ Conectar via SSH

```bash
ssh -i "sua-chave.pem" ec2-user@IP_PUBLICO
```
---

## 3️⃣ Instalar e configurar Nginx
 
```bash 
sudo dnf update -y
sudo dnf install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
Teste com http://IP_PUBLICO → deve aparecer a página padrão do Nginx.

---

## 4️⃣ Subir site do GitHub

4.1 Limpar pasta do Nginx

```bash
sudo rm -rf /usr/share/nginx/html/*
sudo rm -rf /usr/share/nginx/html/.[^.]*  # remove arquivos ocultos
```

4.2 Clonar repositório

```bash
cd /usr/share/nginx/html
sudo git clone https://github.com/SEU_USUARIO/SEU_REPO.git .
```

4.3 Ajustar permissões

```bash
sudo chown -R nginx:nginx /usr/share/nginx/html
sudo chmod -R 755 /usr/share/nginx/html
```

4.4 Testar site

Acesse http://IP_PUBLICO → deve aparecer sua página do GitHub.

---

## 5️⃣ Configurar DuckDNS

1. Crie um subdomínio em DuckDNS
2. Configure o IP público da EC2.
3. Teste:
```bash
ping seusite.duckdns.org
```
- Deve responder com o IP da EC2.

---

## 6️⃣ Configurar Nginx para o DuckDNS

1. Crie arquivo de configuração do site:
```bash
sudo nano /etc/nginx/conf.d/seusite.duckdns.org.conf
```
2. Cole o conteúdo:
```bash
server {
    listen 80;
    server_name seusite.duckdns.org;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
3. Salve e saia (Ctrl+O, Enter, Ctrl+X).
4. Teste e reinicie Nginx:
```bash
sudo nginx -t
sudo systemctl restart nginx,
```

---

## 7️⃣ Ativar HTTPS com Let’s Encrypt

7.1 Instalar Certbot:

```bash
sudo dnf install certbot python3-certbot-nginx -y
```

7.2 Gerar certificado:

```bash
sudo certbot --nginx -d seusite.duckdns.org
```
* Informe seu e-mail
* Concorde com os termos
* Escolha redirecionar HTTP → HTTPS

7.3 Testar HTTPS

Acesse https://seusite.duckdns.org → deve aparecer cadeado verde.

7.4 Testar renovação automática:

```bash
sudo certbot renew --dry-run
```

---

## 8️⃣ Atualizar site com novas alterações do GitHub

```bash
cd /usr/share/nginx/html
sudo git fetch origin
sudo git reset --hard origin/main   # ou origin/master
sudo chown -R nginx:nginx .
sudo chmod -R 755 .
sudo systemctl restart nginx
```

---


## ✅ Resumo do fluxo

1. Criar EC2 e abrir portas no Security Group

2. Instalar Nginx

3. Limpar pasta do Nginx e clonar GitHub

4. Configurar DuckDNS

5. Configurar Nginx com server_name correto

6. Ativar HTTPS com Let’s Encrypt

7. Atualizar site com git fetch e git reset

---

## 📌 Observações

Todos os comandos foram testados no Amazon Linux 2023.

Para Ubuntu, substitua dnf por apt.

Certifique-se de que exista index.html na raiz /usr/share/nginx/html.

DuckDNS é usado para domínios gratuitos; se usar outro domínio, ajuste server_name e DNS

---

## 🌐 Como acessar o site

<a href="https://leandrovenancio.duckdns.org/" target="_blank">
    Clique aqui para acessar o site
</a>
