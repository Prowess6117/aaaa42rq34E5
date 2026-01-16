# MANUAL OPERATIVO - BASE + BLOQUES + GIT/GH (Mint / LMDE)
## Propósito:
- Tener una "máquina base" clonable (VM) y luego activar "bloques por caso"
- Mantener orden lógico: update → install → configuración → utilidades
- Nada se ejecuta por abrir este archivo

## Reglas:
- Copiar/pegar SOLO lo que necesitas
- Lo destructivo/peligroso queda al final
- Los "bloques por caso" se aplican en clones, NO en la base


:

## A) SISTEMA / ENTORNO BASE

# ------------------------------------------------------------
# A1) Actualizar índice de paquetes
# ------------------------------------------------------------
# Actualiza la lista local de paquetes disponibles.
# No instala ni modifica paquetes por sí mismo.
sudo apt update


# ------------------------------------------------------------
# A2) Paquetes base del sistema (transversales)
# ------------------------------------------------------------
# ca-certificates            → HTTPS/TLS (evita errores SSL)
# apt-transport-https        → soporte repos HTTPS (aún útil en algunos casos)
# software-properties-common → add-apt-repository / utilidades de repos
sudo apt install -y \
  ca-certificates \
  apt-transport-https \
  software-properties-common


# ------------------------------------------------------------
# A3) Herramientas CLI transversales (base clonable)
# ------------------------------------------------------------
# git     → control de versiones
# gh      → GitHub CLI
# curl    → requests HTTP
# wget    → descargas
# tree    → estructura de carpetas
# unzip   → descomprimir zip
# zip     → comprimir zip
# rsync   → copias / sync (muy útil para backups y mover proyectos)
# lsof    → ver procesos/puertos
# htop    → monitor de procesos (más cómodo que top)
# nano    → editor simple de rescate (por si no hay otro editor)
# openssh-client → ssh (normalmente ya está, pero lo fijamos en base)
#
# Nota: npm lo dejamos fuera de BASE (va como bloque por caso),
#       porque amarra toolchain de Node/JS y eso suele variar por proyecto.
```
sudo apt install -y \
  git \
  gh \
  curl \
  wget \
  tree \
  unzip \
  zip \
  rsync \
  lsof \
  htop \
  nano \
  openssh-client
```

# ------------------------------------------------------------
# A4) Soporte de compilación (genérico)
# ------------------------------------------------------------
# build-essential → gcc, g++, make (compilar cosas nativas)
# dkms            → recompila módulos cuando cambia el kernel
sudo apt install -y \
  build-essential \
  dkms


# ------------------------------------------------------------
# A5) Soporte kernel (VM / drivers)
# ------------------------------------------------------------
# Headers EXACTOS del kernel en uso (clave para Guest Additions, etc.)
sudo apt install -y \
  linux-headers-$(uname -r)


# ------------------------------------------------------------
# A6) VirtualBox: carpetas compartidas (vboxsf)
# ------------------------------------------------------------
# Permite acceder a carpetas compartidas sin sudo.
# Requiere cerrar sesión o reiniciar para aplicar el grupo.
sudo usermod -aG vboxsf $USER



# ============================================================
# B) UTILIDADES DE USO (NO afectan base, son comandos de día a día)
# ============================================================

# ------------------------------------------------------------
# B1) Servidor HTTP rápido (Python) - primer plano
# ------------------------------------------------------------
# Sirve el directorio actual en http://localhost:8080
python3 -m http.server 8080


# ------------------------------------------------------------
# B2) Servidor HTTP (Python) - background persistente
# ------------------------------------------------------------
# nohup       → no muere al cerrar la terminal
# >server.log → guarda stdout
# 2>&1        → junta stderr con stdout
# &           → ejecuta en segundo plano
# disown      → lo separa del shell actual
nohup python3 -m http.server 8080 > server.log 2>&1 &
disown


# ------------------------------------------------------------
# B3) Ver qué proceso usa un puerto y detenerlo
# ------------------------------------------------------------
# Lista el proceso que está escuchando en el puerto 8080
lsof -i :8080

# Finaliza el proceso manualmente (reemplazar <PID>)
kill <PID>


# ------------------------------------------------------------
# B4) Generar logs de estructura del proyecto
# ------------------------------------------------------------
# tree completo, se muestra y se guarda
tree | tee estructura.log

# tree limitado a 4 niveles
tree -L 4 | tee estructura.log

# Ignora node_modules y .git, nombre con timestamp
tree -I 'node_modules|.git' > tree_$(date +%F_%H-%M).log



# ============================================================
# C) BLOQUES POR CASO (NO van en la BASE; aplicarlos en CLONES)
# ============================================================

# ------------------------------------------------------------
# C1) Caso: Node.js / Frontend básico (NO recomendado en imagen base)
# ------------------------------------------------------------
# Ojo: según distro puede instalar versiones distintas.
# Si quieres versiones por proyecto, considera nvm (bloque alternativo fuera de apt).
sudo apt install -y nodejs npm


# ------------------------------------------------------------
# C2) Caso: VirtualBox Guest Additions (si no usaste la sección A4/A5)
# ------------------------------------------------------------
# Repite paquetes clave por conveniencia (si lo ejecutas da igual si ya están).
sudo apt install -y build-essential dkms linux-headers-$(uname -r)


# ------------------------------------------------------------
# C3) Caso: Debug / red
# ------------------------------------------------------------
# net-tools     → ifconfig/netstat (legacy, aún útil a veces)
# iputils-ping  → ping
# traceroute    → traceroute
sudo apt install -y \
  net-tools \
  iputils-ping \
  traceroute


# ------------------------------------------------------------
# C4) Caso: utilidades gráficas (desktop)
# ------------------------------------------------------------
# xclip            → copiar/pegar desde terminal al portapapeles
# gnome-screenshot → capturas de pantalla rápidas
sudo apt install -y \
  xclip \
  gnome-screenshot


# ------------------------------------------------------------
# C5) Caso: Codex CLI (si lo vas a usar)
# ------------------------------------------------------------
# Requiere npm presente (ver C1).
npm i -g @openai/codex



# ============================================================
# D) GIT / GITHUB / GH CLI
# ============================================================

# ------------------------------------------------------------
# D1) Configuración SSH para GitHub
# ------------------------------------------------------------
# Genera par de llaves ED25519
ssh-keygen -t ed25519 -C "tu-email@github.com"

# Inicia el agente SSH
eval "$(ssh-agent -s)"

# Carga la llave privada en el agente
ssh-add ~/.ssh/id_ed25519

# Muestra la llave pública (copiar y pegar en GitHub)
cat ~/.ssh/id_ed25519.pub

# Prueba conexión con GitHub
ssh -T git@github.com


# ------------------------------------------------------------
# D2) Inicializar repositorio local y subirlo
# ------------------------------------------------------------
git init
git branch -m main

# Crear .gitignore (editar según proyecto)
nano .gitignore

# Ejemplo mínimo:
# node_modules/
# .env
# *.log
# data/

git add -A
git commit -m "init: repositorio local"

# Conectar con remoto por SSH
git remote add origin git@github.com:Prowess6117/repo.git
git push -u origin main


# ------------------------------------------------------------
# D3) Estado y diferencias (solo lectura)
# ------------------------------------------------------------
git status                 # estado local
git diff                   # cambios locales no commiteados
git fetch                  # actualiza info del remoto
git diff origin/main       # local vs remoto
git pull                   # remoto → local
git push                   # local → remoto


# ------------------------------------------------------------
# D4) Cambiar URL del remoto a SSH
# ------------------------------------------------------------
git remote set-url origin git@github.com:samuelmartinez/mi-repo.git

git remote -v
git branch -vv


# ------------------------------------------------------------
# D5) Quitar archivos del control de versiones (sin borrarlos del disco)
# ------------------------------------------------------------
git rm -r --cached ruta
git commit -m "eliminar ruta del control de versiones"


# ------------------------------------------------------------
# D6) Guardar estado (snapshot)
# ------------------------------------------------------------
git status
git add -A
git commit -m "snapshot: comentario"
git push -u origin main


# ------------------------------------------------------------
# D7) Crear repos con GitHub CLI
# ------------------------------------------------------------
# Autenticación (solo primera vez)
gh auth login


# Crear repo desde carpeta actual y pushear
gh repo create nombre_de_repositorio_nuevo \
  --private \
  --source=. \
  --remote=origin \
  --push


# Crear repo vacío y clonarlo
gh repo create nombre_de_repositorio_nuevo --private --clone
cd emul-device
# copia/pega tus archivos aquí
git add -A
git commit -m "init"
git push -u origin main


# Inicializar + crear repo en un solo flujo
cd /ruta/a/tu/carpeta
git init
git branch -M main
git add -A
git commit -m "init"
gh repo create emul-device \
  --private \
  --source=. \
  --remote=origin \
  --push


# ------------------------------------------------------------
# D8) Pull Requests desde terminal (GH CLI)
# ------------------------------------------------------------
# Crear branch de trabajo
git checkout -b feature/nombre

# Trabajar normalmente
git add -A
git commit -m "feat: cambio descriptivo"
git push -u origin feature/nombre

# Crear Pull Request
gh pr create \
  --base main \
  --head feature/nombre \
  --title "Título del PR" \
  --body "Descripción del cambio"

# Ver PRs
gh pr list

# Ver detalles de un PR
gh pr view <ID>

# Hacer checkout de un PR
gh pr checkout <ID>



# ============================================================
# E) COMANDOS PELIGROSOS / DESTRUCTIVOS (AL FINAL)
# ============================================================
# Estos pueden borrar estado local, archivos no versionados o sobrescribir remoto.
# Úsalos SOLO si entiendes el objetivo y ya respaldaste lo importante.


# ------------------------------------------------------------
# E1) Forzar sincronización REMOTO → LOCAL (sobreescribe local)
# ------------------------------------------------------------
# Línea 1: trae info del remoto
git fetch origin
# Línea 2: deja tu rama EXACTA como origin/main (pierdes cambios locales no pusheados)
git reset --hard origin/main
# Línea 3: borra archivos NO trackeados (pero NO ignora ignorados)
git clean -fd

# Variante más agresiva:
# -x: también borra ignorados (por ejemplo caches/ builds/ node_modules si está ignorado)
# -e: excluye un archivo específico de la limpieza
git clean -fdx -e .env.local


# ------------------------------------------------------------
# E2) Limpiar entorno Node completamente (borra dependencias)
# ------------------------------------------------------------
# Borra carpeta de dependencias
rm -rf node_modules
# Borra lockfiles (forzará resolver de nuevo dependencias al instalar)
rm -f package-lock.json
rm -f yarn.lock
rm -f pnpm-lock.yaml


# ------------------------------------------------------------
# E3) Forzar sincronización LOCAL → REMOTO (sobreescribe remoto)
# ------------------------------------------------------------
git add -A
git commit -m "comentario"

# Más seguro: falla si el remoto cambió y tú no lo tienes
git push --force-with-lease origin main

# Más peligroso: pisa el remoto sí o sí (puede borrar trabajo de terceros)
git push --force origin main
