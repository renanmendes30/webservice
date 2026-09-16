
# Raspberry Pi — Site Web para Controlar LED

# 1. Montar o circuito

Vamos utilizar o **GPIO 17**.

### Componentes

* Raspberry Pi
* 1 LED
* 1 resistor de 220 Ω ou 330 Ω
* Protoboard
* Jumpers

### Ligação

Na Raspberry Pi, o GPIO 17 corresponde ao **pino físico 11**.

```text
Raspberry Pi

GPIO 17 → Pino físico 11
GND     → Pino físico 6
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

# 6. Testar o site 
 vamos testar diretamente o Flask.

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
