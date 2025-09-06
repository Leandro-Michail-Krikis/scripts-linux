# RyzenAdj Auto Profile Script & System76 Power/Brightness Auto Switch

Este repositório contém dois scripts para otimizar o gerenciamento de energia e desempenho em notebooks System76 com Ubuntu 24+:

- **script-ryzen-adj.sh**: Ajusta automaticamente os limites de energia do RyzenAdj conforme o perfil de energia e estado da bateria.
- **auto-power-brightness.sh**: Alterna o perfil do `system76-power` entre Performance/Battery e ajusta o brilho automaticamente, mantendo o valor do brilho e mostrando logs no terminal.

## Scripts Disponíveis

---

## Resumo dos scripts

- **script-ryzen-adj.sh**: Otimiza os limites de energia do RyzenAdj automaticamente conforme o perfil e estado da bateria.
- **auto-power-brightness.sh**: Alterna entre perfis Performance/Battery do system76-power e ajusta brilho do display conforme o estado da bateria, exibindo logs.

---

### 1. script-ryzen-adj.sh

Ajusta os limites do RyzenAdj automaticamente de acordo com:
- Perfil de energia (`system76-power profile`)
- Estado da bateria (carregando/descarregando)
- Executa comandos do `ryzenadj` conforme o contexto para otimizar consumo ou desempenho

#### Principais flags usadas no RyzenAdj

- `--fast-limit`: Consumo de boost da cpu
- `--apu-slow-limit`: Consumo continuo maximo da cpu.
- `--slow-limit`: Consumo continuo maximo da cpu.
- `--tctl-temp`: Define a temperatura onde vai começar o throtling.

#### Instalação e Uso

**1. Instale o RyzenAdj e configure o sudo para não pedir senha ao executar o comando.**

**2. Crie o arquivo do script:**
```bash
sudo nano /usr/local/bin/script-ryzen-adj.sh
```
**3. Cole o conteúdo abaixo e salve:**
```bash
unplugged_battery="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=15000 --apu-slow-limit=5000 --slow-limit=5000 --power-saving"
unplugged_balanced="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=25000 --apu-slow-limit=15000 --slow-limit=15000 --power-saving"
unplugged_performance="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=35000 --apu-slow-limit=25000 --slow-limit=25000 --power-saving"

plugged_battery="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=25000 --apu-slow-limit=15000 --slow-limit=15000 --power-saving"
plugged_balanced="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=55000 --apu-slow-limit=45000 --slow-limit=45000 --max-performance"
plugged_performance="ryzenadj --tctl-temp=85 --apu-skin-temp=60 --dgpu-skin-temp=60 --fast-limit=75000 --apu-slow-limit=65000 --slow-limit=65000 --max-performance"

battery_state=$(upower -i $(upower -e | grep BAT) | grep -E "state" | awk '{print $2}')
current_profile=$(system76-power profile | grep "Power Profile" | awk '{print $3}')

echo "$(date '+%Y-%m-%d %H:%M:%S') - Estado bateria: $battery_state"
echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil: $current_profile"

if [ "$battery_state" = "discharging" ]; then
    if [ "$current_profile" = "Battery" ]; then
        sudo bash -c "$unplugged_battery"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: unplugged battery"
    elif [ "$current_profile" = "Balanced" ]; then
        sudo bash -c "$unplugged_balanced"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: unplugged balanced"
    else
        sudo bash -c "$unplugged_performance"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: unplugged performance"
    fi
else
    if [ "$current_profile" = "Battery" ]; then
        sudo bash -c "$plugged_battery"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: plugged battery"
    elif [ "$current_profile" = "Balanced" ]; then
        sudo bash -c "$plugged_balanced"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: plugged balanced"
    else
        sudo bash -c "$plugged_performance"
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil-aplicado: plugged performance"
    fi
fi
```

**4. Dê permissão de execução:**
```bash
sudo chmod +x /usr/local/bin/script-ryzen-adj.sh
```

**5. Configure para executar automaticamente:**
Edite o crontab do root:
```bash
sudo crontab -e
```
Adicione ao final:
```
@reboot /usr/local/bin/script-ryzen-adj.sh
* * * * * /usr/local/bin/script-ryzen-adj.sh
```

**6. Pronto!**  
O script será executado automaticamente a cada minuto e ao iniciar o sistema.

Para monitorar:
```bash
watch -n 2 sudo ryzenadj -i
```

> **Observação:**  
O script requer permissões de superusuário para executar o `ryzenadj`.  
Certifique-se de configurar o sudoers para não pedir senha.

---

### 2. auto-power-brightness.sh

Alterna automaticamente o perfil de energia do `system76-power` entre **Performance** e **Battery**, mantendo o valor de brilho do display (limitando a 80% na bateria) e exibindo logs detalhados no terminal.

#### Instalação e Uso

**1. Crie o arquivo do script:**
```bash
sudo nano /usr/local/bin/auto-power-brightness.sh
```

**2. Cole o conteúdo abaixo e salve:**
```bash
#!/bin/bash
# Ajusta o profile system76-power mantendo brilho e mostrando logs no terminal

echo "$(date '+%Y-%m-%d %H:%M:%S') - Iniciando script"

# Pega o estado da bateria
battery_state=$(upower -i $(upower -e | grep BAT) | grep -E "state" | awk '{print $2}')
echo "$(date '+%Y-%m-%d %H:%M:%S') - Estado da bateria: $battery_state"

# Pega o profile atual
current_profile=$(system76-power profile | grep "Power Profile" | awk '{print $3}')

# Pega o brilho atual
current_brightness=$(system76-power profile | grep "Backlight" | awk -F'=' '{print $2}' | awk '{print $1}' | tr -d '%')

echo "$(date '+%Y-%m-%d %H:%M:%S') - Perfil atual: $current_profile, Brilho atual: $current_brightness"

if [ "$battery_state" = "discharging" ]; then
    if [ "$current_profile" = "Performance" ]; then
        system76-power profile battery
        if [ "$current_brightness" -gt 80 ]; then
            current_brightness=80
        fi
        sudo brightnessctl set ${current_brightness}%
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Mudou para Battery com brilho $current_brightness%"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Já está no Battery"
    fi
else
    if [ "$current_profile" = "Battery" ]; then
        system76-power profile performance
        sudo brightnessctl set ${current_brightness}%
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Mudou para Performance com brilho $current_brightness%"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Já está no Performance"
    fi
fi
```

**3. Dê permissão de execução:**
```bash
sudo chmod +x /usr/local/bin/auto-power-brightness.sh
```

**4. Configure para executar automaticamente:**
Edite o crontab do root:
```bash
sudo crontab -e
```
Adicione ao final:
```
@reboot /usr/local/bin/auto-power-brightness.sh
* * * * * /usr/local/bin/auto-power-brightness.sh
```

**5. Pronto!**  
O script será executado automaticamente a cada minuto e ao iniciar o sistema.
