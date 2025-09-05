# RyzenAdj Auto Profile Script

Este script ajusta automaticamente os limites de energia do RyzenAdj conforme o perfil de energia e estado da bateria do seu computador System76 (Ubuntu 24+).

## Passo a Passo de Instalação e Uso

### 1. Instale o RyzenAdj e configure para não precisar senha ao executar lo com sudo

### 2. Salve o Script

Crie o arquivo do script:

```bash
sudo nano /usr/local/bin/script-ryzen-adj.sh
```

Cole o conteúdo abaixo e salve:

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

### 3. Dê permissão de execução

```bash
sudo chmod +x /usr/local/bin/script-ryzen-adj.sh
```

### 4. Configure para executar automaticamente

Edite o crontab do root:

```bash
sudo crontab -e
```

Adicione as linhas abaixo ao final do arquivo:

```
@reboot /usr/local/bin/script-ryzen-adj.sh
* * * * * /usr/local/bin/script-ryzen-adj.sh
```

### 5. Pronto!

O script será executado automaticamente a cada minuto e ao iniciar o sistema.

---

**Observação:**  
O script requer permissões de superusuário para executar o `ryzenadj`.  
Certifique-se de que o usuário root tem permissão para rodar o comando sem senha, ou configure o sudoers conforme necessário.
