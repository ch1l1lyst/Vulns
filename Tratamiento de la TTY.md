**Objetivo:** Convertir una reverse shell básica (que no tiene historial, ni permite `Ctrl+C`, ni autocompletado) en una terminal completamente interactiva.
```bash
# 1. DENTRO DE LA SHELL REMOTA (Spawnear TTY)
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 2. PRESIONA Ctrl+Z EN TU TECLADO (Suspende la shell)

# 3. EN TU KALI LOCAL
stty raw -echo; fg
# (Presiona Enter 2 veces)

# 4. DENTRO DE LA SHELL REMOTA (Resetear y variables)
reset xterm
export TERM=xterm-256color
export SHELL=/bin/bash

# 5. DENTRO DE LA SHELL REMOTA (Fix Prompt PS1)
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# 6. DENTRO DE LA SHELL REMOTA (Fix Ctrl+L / clear)
alias clear='printf "\033c"'

# 7. EN TU KALI LOCAL (Ver tamaño)
stty size

# 8. DENTRO DE LA SHELL REMOTA (Ajustar tamaño, cambia 40 y 120 por tu output)
stty rows 40 columns 120
```