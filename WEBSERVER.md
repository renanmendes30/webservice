# Servidor Web com Raspberry Pi

## 1. Objetivo

Transformar uma Raspberry Pi em um servidor web local utilizando o **Apache HTTP Server**.

Ao final, a Raspberry Pi estará apta a:

- Servir páginas HTML pela rede local.
- Ser administrada por comandos `systemctl`.
- Hospedar um site usando um **Virtual Host**.
- Utilizar um domínio local, como `meusite.local`.
- Servir uma página HTML personalizada.

---

## 2. Pré-requisitos

Antes de começar, verifique se a Raspberry Pi possui:

- Raspberry Pi OS ou outro sistema baseado em Debian.
- Conexão com a Internet.
- Acesso ao terminal ou SSH.
- Um endereço IP acessível na rede local.

Para descobrir o endereço IP:

```bash
hostname -I
```

Exemplo:

```text
192.168.10.100
```

---

# 3. Atualizar o sistema

Atualize a lista de pacotes:

```bash
sudo apt-get update
```

Depois, atualize os pacotes instalados:

```bash
sudo apt-get upgrade -y
```

---

# 4. Instalar o Apache

Instale o Apache:

```bash
sudo apt-get install apache2 -y
```

Após a instalação, o serviço normalmente é iniciado automaticamente.

Para verificar:

```bash
sudo systemctl status apache2
```

---

# 5. Testar o Apache

Descubra o IP da Raspberry Pi:

```bash
hostname -I
```

Em outro computador conectado à mesma rede, abra:

```text
http://IP_DA_RASPBERRY
```

Por exemplo:

```text
http://192.168.10.100
```

Se estiver funcionando, será apresentada a página padrão do Apache.

---

# 6. Comandos básicos do Apache

### Verificar status

```bash
sudo systemctl status apache2
```

### Iniciar

```bash
sudo systemctl start apache2
```

### Parar

```bash
sudo systemctl stop apache2
```

### Reiniciar

```bash
sudo systemctl restart apache2
```

### Recarregar configurações

```bash
sudo systemctl reload apache2
```

### Habilitar inicialização automática

```bash
sudo systemctl enable apache2
```

---

# 7. Estrutura de configuração do Apache

Os principais arquivos do Apache ficam em:

```text
/etc/apache2/
```

O arquivo principal é:

```text
/etc/apache2/apache2.conf
```

Para editar:

```bash
sudo nano /etc/apache2/apache2.conf
```

Os sites podem ser configurados por meio de **Virtual Hosts**.

Os arquivos de sites ficam principalmente em:

```text
/etc/apache2/sites-available/
```

e os sites habilitados são relacionados em:

```text
/etc/apache2/sites-enabled/
```

---

# 8. Criar um site com Virtual Host

Neste exemplo será utilizado o domínio local:

```text
meusite.local
```

Crie o diretório do site:

```bash
sudo mkdir -p /var/www/meusite.local
```

Defina inicialmente as permissões:

```bash
sudo chown -R $USER:$USER /var/www/meusite.local
sudo chmod -R 755 /var/www/meusite.local
```

---

# 9. Criar o arquivo do Virtual Host

Copie a configuração padrão:

```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/meusite.local.conf
```

Edite:

```bash
sudo nano /etc/apache2/sites-available/meusite.local.conf
```

Utilize uma configuração semelhante a:

```apache
<VirtualHost *:80>

    ServerAdmin admin@meusite.local
    ServerName meusite.local
    ServerAlias www.meusite.local

    DocumentRoot /var/www/meusite.local

    ErrorLog ${APACHE_LOG_DIR}/meusite.local_error.log
    CustomLog ${APACHE_LOG_DIR}/meusite.local_access.log combined

</VirtualHost>
```

## Principais diretivas

| Diretiva | Função |
|---|---|
| `ServerAdmin` | E-mail do administrador |
| `ServerName` | Nome principal do site |
| `ServerAlias` | Nome alternativo |
| `DocumentRoot` | Diretório dos arquivos do site |
| `ErrorLog` | Arquivo de log de erros |
| `CustomLog` | Arquivo de log de acessos |

---

# 10. Habilitar o Virtual Host

Execute:

```bash
sudo a2ensite meusite.local.conf
```

Depois, recarregue o Apache:

```bash
sudo systemctl reload apache2
```

Antes de recarregar, é recomendável verificar a configuração:

```bash
sudo apache2ctl configtest
```

O resultado esperado é:

```text
Syntax OK
```

---

# 11. Configurar o arquivo hosts

Para que `meusite.local` seja associado ao IP da Raspberry Pi, edite o arquivo:

```bash
sudo nano /etc/hosts
```

Adicione:

```text
192.168.10.100 meusite.local
```

Substitua `192.168.10.100` pelo IP real da Raspberry Pi.

> **Importante:** se o acesso for feito a partir de outro computador, o arquivo `hosts` desse computador também precisará resolver `meusite.local` para o IP da Raspberry Pi, ou deverá ser utilizado outro mecanismo de DNS local.

---

# 12. Configurar permissões do site

Para que o Apache possa acessar os arquivos:

```bash
sudo chown -R www-data:www-data /var/www/meusite.local
sudo chmod -R 755 /var/www/meusite.local
```

O usuário `www-data` é utilizado pelo Apache em sistemas Debian/Ubuntu.

---

# 13. Desabilitar listagem de diretórios

A listagem automática de diretórios pode expor arquivos quando não existe um `index.html`.

Edite:

```bash
sudo nano /etc/apache2/apache2.conf
```

Localize:

```apache
Options Indexes FollowSymLinks
```

E altere para:

```apache
Options FollowSymLinks
```

Depois:

```bash
sudo apache2ctl configtest
```

Se aparecer:

```text
Syntax OK
```

recarregue:

```bash
sudo systemctl reload apache2
```

---

# 14. Criar uma página HTML

O diretório padrão do Apache é:

```text
/var/www/html
```

Entre no diretório:

```bash
cd /var/www/html
```

Crie a página:

```bash
sudo nano index.html
```

Exemplo de página:

```html
<!DOCTYPE html>
<html lang="pt-br">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Servidor Web - Raspberry Pi</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            text-align: center;
            padding: 50px;
        }

        h1 {
            color: #333;
        }

        p {
            color: #666;
        }

        .button {
            display: inline-block;
            padding: 10px 20px;
            margin: 20px;
            font-size: 16px;
            background-color: #008cba;
            color: white;
            text-decoration: none;
            border-radius: 5px;
        }

        .button:hover {
            background-color: #005f73;
        }
    </style>
</head>

<body>

    <h1>Bem-vindo ao meu servidor web!</h1>

    <p>
        Esta página está sendo hospedada em uma Raspberry Pi.
    </p>

    <a
        href="https://www.raspberrypi.com/"
        class="button"
        target="_blank">
        Site da Raspberry Pi
    </a>

</body>

</html>
```

---

# 15. Ajustar permissões do `index.html`

Execute:

```bash
sudo chown www-data:www-data /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html
```

---

# 16. Testar o servidor

Abra um navegador em um dispositivo conectado à mesma rede.

Utilizando o IP:

```text
http://192.168.10.100
```

Ou utilizando o Virtual Host:

```text
http://meusite.local
```

Se tudo estiver configurado corretamente, a página HTML criada será exibida.

---

# 17. Diagnóstico de problemas

## Verificar se o Apache está ativo

```bash
sudo systemctl status apache2
```

## Verificar a configuração

```bash
sudo apache2ctl configtest
```

Esperado:

```text
Syntax OK
```

## Verificar se a porta 80 está em uso

```bash
sudo ss -lntp | grep :80
```

## Consultar erros do Apache

```bash
sudo tail -f /var/log/apache2/error.log
```

## Consultar acessos

```bash
sudo tail -f /var/log/apache2/access.log
```

## Listar Virtual Hosts ativos

```bash
sudo apache2ctl -S
```

---

# 18. Fluxo resumido

```text
Raspberry Pi
     |
     | Apache
     v
Porta 80
     |
     v
Virtual Host
     |
     v
/var/www/meusite.local
     |
     v
index.html
     |
     v
Navegador
```

## Referência

Tutorial original:

https://www.makerhero.com/guia/raspberry-pi/servidor-web-raspberry-pi/

Conteúdo reorganizado para uso didático, mantendo os conceitos e procedimentos essenciais do tutorial original.
