# Target (IP address on Terminal - copy/pasteable address)

* Update: Conservar VPN IP address (step 5)
  * Quitar las f*king "XXX" placeholder: restarurar en el segundo Gemmon el `command` al script default `/usr/share/kali-themes/xfce4-panel-genmon-vpnip.sh`

## 1. ~/.config/target/target.sh

```sh
# Source this file from ~/.zshrc or ~/.bashrc
TARGET_FILE="$HOME/.config/target/target"

settarget() {
  local ip="$1"
  local name="$2"

  if [ -z "$ip" ] || [ -z "$name" ]; then
    echo "Usage: settarget <ip> <name>" >&2
    return 2
  fi

  printf '%s %s\n' "$ip" "$name" > "$TARGET_FILE"
}

cleartarget() {
  : > "$TARGET_FILE" 2>/dev/null || true
}

showtarget() {
  if [ -s "$TARGET_FILE" ]; then
    cat "$TARGET_FILE"
  else
    echo "No target"
  fi
}

_target_prompt() {
  if [ -s "$TARGET_FILE" ]; then
    local ip name
    ip="$(awk '{print $1}' "$TARGET_FILE" 2>/dev/null)"
    name="$(awk '{print $2}' "$TARGET_FILE" 2>/dev/null)"
    if [ -n "$ip" ] && [ -n "$name" ]; then
      printf '%s - %s' "$ip" "$name"
      return 0
    fi
  fi
}
```


## 2. Kali Zsh: ~/.zshrc

```sh
# Target indicator
source "$HOME/.config/target/target.sh"
RPROMPT='$(_target_prompt)'
```


---


# Audit (IP address on Task bar - no copy/pasteable address)


## 1. ~/.config/target/audit.sh

```sh
AUDIT_FILE="$HOME/.config/target/audit"

setaudit() {
  local ip="$1"
  local name="$2"

  if [ -z "$ip" ] || [ -z "$name" ]; then
    echo "Usage: setaudit <ip> <name>" >&2
    return 2
  fi

  printf '%s %s\n' "$ip" "$name" > "$AUDIT_FILE"
}

clearaudit() {
  : > "$AUDIT_FILE" 2>/dev/null || true
}

showaudit() {
  if [ -s "$AUDIT_FILE" ]; then
    cat "$AUDIT_FILE"
  else
    echo "No target"
  fi
}

_audit_text() {
  if [ -s "$AUDIT_FILE" ]; then
    local ip name
    ip="$(awk '{print $1}' "$AUDIT_FILE" 2>/dev/null || true)"
    name="$(awk '{print $2}' "$AUDIT_FILE" 2>/dev/null || true)"
    if [ -n "$ip" ] && [ -n "$name" ]; then
      printf '%s - %s' "$ip" "$name"
      return 0
    fi
  fi
  #printf 'No target'
}
```

## 2. ~/.config/target/audit-panel.sh

The file should have execution permissions **chmod +x "$HOME/.config/target/audit-panel.sh"**

```sh
#!/usr/bin/env bash
set -euo pipefail

# Genmon ejecuta con /bin/sh a veces; forzamos bash aquí por shebang.
source "$HOME/.config/target/audit.sh"

# Genmon requiere XML simple
printf '<txt>%s</txt>\n' "$(_audit_text)"
```


## 3. Kali Zsh: ~/.zshrc

```sh
# Target indicator
source "$HOME/.config/target/audit.sh"
```


## 4. “Generic Monitor” (genmon)

Para que ejecute un script y pinte el target en el panel, a la izquierda (junto al menú del dragón) o a la derecha (cerca del reloj).

```sh
sudo apt update
sudo apt install -y xfce4-genmon-plugin
```

### Installation

Por GUI (recomendado)
- Click derecho sobre el panel (la barra de arriba)
- Panel -> Panel Preferences
- Tab Items
- Pulsa +
- Añade “Generic Monitor”
- Selecciona “Generic Monitor” en la lista y pulsa el icono de configuración (la llave/engranaje) o click izquierdo
- Command: `/bin/bash -lc "$HOME/.config/target/audit-panel.sh"`
- Interval: 1 (o 2 si quieres menos refrescos)
- Marca “Label” vacío (o desactívalo si aparece)


## 5. VPN IP address

1. Incluir un segundo Gemmon
2. Command: `/bin/bash -lc "ip -o -4 addr show tun0 2>/dev/null | awk '{print \$4}' | cut -d/ -f1"`