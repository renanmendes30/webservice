Claro. Vou partir **exatamente do ponto em que o seu tutorial termina**: Apache funcionando, Virtual Host `meusite.local` configurado e `index.html` sendo servido. O arquivo confirma essa estrutura e o `DocumentRoot` `/var/www/meusite.local`. 

A partir daqui vamos transformar o site em um **painel web para ligar e desligar um LED físico**.

# Raspberry Pi — Site Web para Controlar LED

## Visão final do projeto

Ao final, teremos:

```text
                    REDE LOCAL
                         │
                         ▼
              ┌──────────────────┐
              │     Navegador    │
              │                  │
              │  CONTROLE LED    │
              │                  │
              │ [ LIGAR ]        │
              │ [ DESLIGAR ]     │
              └────────┬─────────┘
                       │
                       │ HTTP
                       ▼
              ┌──────────────────┐
              │      Apache      │
              │   meusite.local  │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │    Flask/Python  │
              │                  │
              │      GPIO 17     │
              └────────┬─────────┘
                       │
                       ▼
                    ┌─────┐
                    │ LED │
                    └─────┘
```

Vou usar **Python + Flask** para fazer a comunicação entre o site e o GPIO. Isso deixa o projeto mais simples e didático do que executar Python diretamente como CGI.

---

# 1. Montar o circuito

Vamos utilizar o **GPIO 17**.

### Componentes

* Raspberry Pi
* 1 LED
* 1 resistor de 220 Ω ou 330 Ω
* Protoboard
* Jumpers

### Ligação

```text
GPIO 17
   │
   │
 [220Ω]
   │
   │
   ├────► LED ──── GND
```

Na Raspberry Pi, o GPIO 17 corresponde ao **pino físico 11**.

```text
Raspberry Pi

GPIO 17 → Pino físico 11
GND     → Pino físico 6
```

Então:

```text
Pino 11 ─── resistor ─── LED ─── Pino 6
GPIO 17                         GND
```

**Atenção:** o LED deve ter resistor em série.

---

# 2. Testar se o GPIO funciona

Antes de colocar o site no meio, vamos testar o LED.

Primeiro verifique a versão do sistema:

```bash
cat /etc/os-release
```

Depois instale o pacote que vamos utilizar:

```bash
sudo apt update
sudo apt install python3-flask python3-rpi.gpio -y
```

Teste se o Python consegue importar o GPIO:

```bash
python3 -c "import RPi.GPIO as GPIO; print('GPIO OK')"
```

Se aparecer:

```text
GPIO OK
```

podemos continuar.

---

# 3. Criar um programa simples para testar o LED

Crie uma pasta para o projeto:

```bash
mkdir -p ~/controle-led
cd ~/controle-led
```

Crie o arquivo:

```bash
nano teste_led.py
```

Coloque:

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)

LED = 17

GPIO.setup(LED, GPIO.OUT)

try:
    print("LED ligado")
    GPIO.output(LED, GPIO.HIGH)

    time.sleep(5)

    print("LED desligado")
    GPIO.output(LED, GPIO.LOW)

finally:
    GPIO.cleanup()
```

Salve:

**CTRL + O**

Enter

**CTRL + X**

Execute:

```bash
python3 teste_led.py
```

O resultado esperado é:

```text
LED ligado
LED desligado
```

E fisicamente:

```text
LED
 ↓
LIGA
 ↓
espera 5 segundos
 ↓
DESLIGA
```

### Se isso funcionar

Perfeito.

Agora sabemos que:

```text
Python → GPIO → LED
```

está funcionando.

---

# 4. Criar o sistema web

Agora vamos criar o programa que ficará responsável por receber os comandos do navegador.

Entre novamente na pasta:

```bash
cd ~/controle-led
```

Crie:

```bash
nano app.py
```

Coloque:

```python
from flask import Flask, render_template, redirect, url_for
import RPi.GPIO as GPIO

app = Flask(__name__)

LED = 17

GPIO.setmode(GPIO.BCM)
GPIO.setup(LED, GPIO.OUT)

estado_led = False

@app.route("/")
def index():
    return render_template("index.html", estado=estado_led)


@app.route("/ligar")
def ligar():
    global estado_led

    GPIO.output(LED, GPIO.HIGH)
    estado_led = True

    return redirect(url_for("index"))


@app.route("/desligar")
def desligar():
    global estado_led

    GPIO.output(LED, GPIO.LOW)
    estado_led = False

    return redirect(url_for("index"))


if __name__ == "__main__":
    try:
        app.run(host="0.0.0.0", port=5000)

    finally:
        GPIO.output(LED, GPIO.LOW)
        GPIO.cleanup()
```

Salve:

```text
CTRL + O
ENTER
CTRL + X
```

---

# 5. Criar a página HTML

Ainda dentro de:

```bash
~/controle-led
```

crie a pasta:

```bash
mkdir templates
```

Agora:

```bash
nano templates/index.html
```

Coloque:

```html
<!DOCTYPE html>
<html lang="pt-br">

<head>
    <meta charset="UTF-8">

    <meta name="viewport"
          content="width=device-width, initial-scale=1.0">

    <title>Controle de LED</title>

    <style>

        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: #f2f2f2;

            display: flex;
            justify-content: center;
            align-items: center;

            min-height: 100vh;
        }

        .container {
            background: white;

            width: 400px;

            padding: 40px;

            border-radius: 15px;

            text-align: center;

            box-shadow:
                0 5px 20px rgba(0,0,0,0.15);
        }

        h1 {
            margin-bottom: 30px;
        }

        .led {
            width: 100px;
            height: 100px;

            border-radius: 50%;

            margin: 20px auto;

            border: 5px solid #555;
        }

        .ligado {
            background: red;

            box-shadow:
                0 0 30px red;
        }

        .desligado {
            background: #333;
        }

        .status {
            font-size: 22px;

            margin: 25px 0;
        }

        .botoes {
            display: flex;

            justify-content: center;

            gap: 15px;
        }

        button {
            border: none;

            padding: 15px 25px;

            font-size: 18px;

            border-radius: 8px;

            cursor: pointer;
        }

        .ligar {
            background: green;
            color: white;
        }

        .desligar {
            background: #555;
            color: white;
        }

        button:hover {
            opacity: 0.8;
        }

    </style>

</head>

<body>

    <div class="container">

        <h1>Controle de LED</h1>

        {% if estado %}

            <div class="led ligado"></div>

            <div class="status">
                LED <strong>LIGADO</strong>
            </div>

        {% else %}

            <div class="led desligado"></div>

            <div class="status">
                LED <strong>DESLIGADO</strong>
            </div>

        {% endif %}


        <div class="botoes">

            <a href="/ligar">
                <button class="ligar">
                    LIGAR
                </button>
            </a>

            <a href="/desligar">
                <button class="desligar">
                    DESLIGAR
                </button>
            </a>

        </div>

    </div>

</body>

</html>
```

---

# 6. Testar o site sem Apache

Antes de configurar o Apache, vamos testar diretamente o Flask.

Entre na pasta:

```bash
cd ~/controle-led
```

Execute:

```bash
sudo python3 app.py
```

Você deverá ver algo parecido com:

```text
* Running on all addresses
* Running on http://127.0.0.1:5000
* Running on http://192.168.x.x:5000
```

Descubra o IP:

```bash
hostname -I
```

Por exemplo:

```text
192.168.10.100
```

Em outro computador da mesma rede, acesse:

```text
http://192.168.10.100:5000
```

Deverá aparecer:

```text
┌──────────────────────────┐
│     CONTROLE DE LED      │
│                          │
│          🔴             │
│                          │
│      LED DESLIGADO      │
│                          │
│  [ LIGAR ] [ DESLIGAR ] │
└──────────────────────────┘
```

Clique em:

**LIGAR**

O LED físico deverá acender.

Clique:

**DESLIGAR**

O LED deverá apagar.

---

# 7. Se funcionar, temos a parte principal pronta

Neste momento temos:

```text
Navegador
    ↓
Flask
    ↓
Python
    ↓
GPIO 17
    ↓
LED
```

Agora precisamos colocar isso atrás do Apache.

---

# 8. Parar o Flask

No terminal onde está rodando:

```bash
sudo python3 app.py
```

pressione:

```text
CTRL + C
```

---

# 9. Criar um serviço para o Flask

Não queremos ficar executando:

```bash
sudo python3 app.py
```

manualmente toda vez.

Vamos criar um serviço do Linux.

Crie:

```bash
sudo nano /etc/systemd/system/controle-led.service
```

Coloque:

```ini
[Unit]
Description=Servidor Flask - Controle de LED
After=network.target

[Service]
User=root
WorkingDirectory=/home/SEU_USUARIO/controle-led
ExecStart=/usr/bin/python3 /home/SEU_USUARIO/controle-led/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

### IMPORTANTE

Troque:

```text
SEU_USUARIO
```

pelo usuário da sua Raspberry Pi.

Por exemplo, se seu usuário for:

```text
renanmendes
```

ficaria:

```ini
WorkingDirectory=/home/renanmendes/controle-led
ExecStart=/usr/bin/python3 /home/renanmendes/controle-led/app.py
```

---

# 10. Ativar o serviço

Execute:

```bash
sudo systemctl daemon-reload
```

Depois:

```bash
sudo systemctl enable controle-led
```

E:

```bash
sudo systemctl start controle-led
```

Verifique:

```bash
sudo systemctl status controle-led
```

Deve aparecer:

```text
Active: active (running)
```

---

# 11. Testar novamente

Descubra o IP:

```bash
hostname -I
```

No navegador:

```text
http://IP_DA_RASPBERRY:5000
```

Por exemplo:

```text
http://192.168.10.100:5000
```

Se funcionar, agora podemos integrar com o Apache.

---

# 12. Fazer o Apache encaminhar para o Flask

Aqui está a parte que conecta o seu **`meusite.local`** ao Flask.

Seu arquivo atual usa:

```text
/var/www/meusite.local
```

como `DocumentRoot`. Isso está definido no Virtual Host do tutorial. 

Vamos manter o mesmo domínio:

```text
meusite.local
```

Abra:

```bash
sudo nano /etc/apache2/sites-available/meusite.local.conf
```

Substitua o conteúdo pelo seguinte:

```apache
<VirtualHost *:80>

    ServerAdmin admin@meusite.local

    ServerName meusite.local

    ServerAlias www.meusite.local


    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:5000/

    ProxyPassReverse / http://127.0.0.1:5000/


    ErrorLog ${APACHE_LOG_DIR}/meusite.local_error.log

    CustomLog ${APACHE_LOG_DIR}/meusite.local_access.log combined

</VirtualHost>
```

---

# 13. Habilitar os módulos necessários

Execute:

```bash
sudo a2enmod proxy
```

Depois:

```bash
sudo a2enmod proxy_http
```

Agora:

```bash
sudo systemctl restart apache2
```

---

# 14. Verificar a configuração do Apache

Muito importante:

```bash
sudo apache2ctl configtest
```

O resultado precisa ser:

```text
Syntax OK
```

O próprio tutorial recomenda fazer essa validação antes de recarregar o Apache. 

---

# 15. Testar pelo domínio

Agora você não precisa mais colocar:

```text
:5000
```

No navegador:

```text
http://meusite.local
```

O caminho será:

```text
Navegador
     │
     │ HTTP :80
     ▼
 Apache
     │
     │ Proxy
     ▼
 Flask :5000
     │
     ▼
 Python
     │
     ▼
 GPIO 17
     │
     ▼
 LED
```

---

# 16. Verificar o arquivo hosts

Seu tutorial utiliza o arquivo `/etc/hosts` para associar o domínio ao IP da Raspberry Pi. 

Na Raspberry Pi:

```bash
sudo nano /etc/hosts
```

Adicione:

```text
192.168.10.100 meusite.local
```

Troque pelo IP real.

Mas existe um detalhe importante:

### Se você acessar de um computador Windows

O Windows também precisa saber que:

```text
meusite.local
```

corresponde à Raspberry Pi.

No Windows, abra o **Bloco de Notas como administrador**.

Abra:

```text
C:\Windows\System32\drivers\etc\hosts
```

Adicione:

```text
192.168.10.100 meusite.local
```

Salve.

Depois abra o navegador:

```text
http://meusite.local
```

---

# 17. Teste final

Agora faça exatamente esta sequência:

### Teste 1

Abra:

```text
http://meusite.local
```

### Teste 2

Clique:

```text
LIGAR
```

Resultado:

```text
Site → Apache → Flask → Python → GPIO 17 → LED LIGADO
```

### Teste 3

Clique:

```text
DESLIGAR
```

Resultado:

```text
Site → Apache → Flask → Python → GPIO 17 → LED DESLIGADO
```

---

# 18. Verificar o serviço do LED

Se o site não responder, execute:

```bash
sudo systemctl status controle-led
```

Se aparecer erro:

```bash
sudo journalctl -u controle-led -n 50
```

Isso mostra os últimos erros do programa.

---

# 19. Verificar o Apache

Execute:

```bash
sudo systemctl status apache2
```

Depois:

```bash
sudo apache2ctl configtest
```

Deve retornar:

```text
Syntax OK
```

Você também pode verificar os Virtual Hosts:

```bash
sudo apache2ctl -S
```

O tutorial também indica esse comando para diagnosticar os Virtual Hosts ativos. 

---

# 20. Verificar se o Flask está funcionando

Execute:

```bash
sudo ss -lntp | grep 5000
```

Deverá aparecer algo semelhante a:

```text
LISTEN 0 128 0.0.0.0:5000
```

Isso significa que o Flask está escutando na porta 5000.

---

# 21. Verificar se o Apache está na porta 80

```bash
sudo ss -lntp | grep :80
```

Deverá aparecer:

```text
LISTEN ... :80
```

---

# 22. Estrutura final do projeto

Ao final, você terá:

```text
/home/SEU_USUARIO/
│
└── controle-led/
    │
    ├── app.py
    │
    ├── teste_led.py
    │
    └── templates/
        │
        └── index.html
```

E o Apache:

```text
/etc/apache2/
│
├── apache2.conf
│
├── sites-available/
│   │
│   └── meusite.local.conf
│
└── sites-enabled/
    │
    └── meusite.local.conf
```

---

# 23. O que cada arquivo faz

| Arquivo                | Função                                          |
| ---------------------- | ----------------------------------------------- |
| `index.html`           | Interface do site                               |
| `app.py`               | Servidor web e controle do GPIO                 |
| `teste_led.py`         | Teste inicial do LED                            |
| `meusite.local.conf`   | Configuração do Apache                          |
| `controle-led.service` | Mantém o programa Flask funcionando             |
| `/etc/hosts`           | Faz `meusite.local` apontar para a Raspberry Pi |

---

# 24. O fluxo completo

Quando o aluno clicar em **LIGAR**:

```text
┌──────────────┐
│   NAVEGADOR  │
│              │
│ [ LIGAR ]    │
└──────┬───────┘
       │
       │ HTTP
       ▼
┌──────────────┐
│    APACHE    │
│     :80      │
└──────┬───────┘
       │
       │ Proxy
       ▼
┌──────────────┐
│    FLASK     │
│     :5000    │
└──────┬───────┘
       │
       │ /ligar
       ▼
┌──────────────┐
│    PYTHON    │
│              │
│ GPIO.output  │
└──────┬───────┘
       │
       │ GPIO 17
       ▼
     ┌─────┐
     │ LED │
     │  ON │
     └─────┘
```

E para desligar:

```text
NAVEGADOR
    ↓
APACHE
    ↓
FLASK
    ↓
/desligar
    ↓
GPIO 17 = LOW
    ↓
LED OFF
```

---

## 25. Um detalhe importante sobre o seu tutorial

Você **não precisa apagar o que já fez**.

O tutorial original já deixou preparado:

```text
Apache
  ↓
Virtual Host
  ↓
meusite.local
  ↓
/var/www/meusite.local
```

Isso continua sendo útil. O que estamos adicionando é:

```text
Apache
   ↓
Proxy
   ↓
Flask
   ↓
GPIO
```

O Apache continua sendo o servidor de entrada, enquanto o Flask passa a cuidar da lógica do controle do LED.

---

## 26. Sequência resumida para os alunos

Depois que o servidor Apache do seu arquivo estiver funcionando, a aula pode seguir exatamente esta ordem:

```bash
# 1. Instalar dependências
sudo apt update
sudo apt install python3-flask python3-rpi.gpio -y

# 2. Criar projeto
mkdir -p ~/controle-led
cd ~/controle-led

# 3. Criar programa
nano teste_led.py

# 4. Testar LED
sudo python3 teste_led.py

# 5. Criar aplicação
nano app.py

# 6. Criar página
mkdir templates
nano templates/index.html

# 7. Testar Flask
sudo python3 app.py

# 8. Criar serviço
sudo nano /etc/systemd/system/controle-led.service

# 9. Ativar serviço
sudo systemctl daemon-reload
sudo systemctl enable controle-led
sudo systemctl start controle-led

# 10. Habilitar proxy no Apache
sudo a2enmod proxy
sudo a2enmod proxy_http

# 11. Configurar Virtual Host
sudo nano /etc/apache2/sites-available/meusite.local.conf

# 12. Testar Apache
sudo apache2ctl configtest

# 13. Reiniciar Apache
sudo systemctl restart apache2

# 14. Acessar
http://meusite.local
```

### Resultado esperado da aula

O aluno terá construído um pequeno sistema de **IoT/Web**:

**HTML → HTTP → Apache → Flask/Python → GPIO → LED**

Isso é uma evolução muito boa do seu primeiro exercício de servidor web, porque o aluno deixa de apenas **hospedar uma página** e passa a **controlar um dispositivo físico através de uma interface web**.
