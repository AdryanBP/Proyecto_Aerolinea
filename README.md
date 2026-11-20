  Comandos en la nueva máquina
  1. Clonar el repositorio y entrar a la carpeta

    git clone <URL_DE_TU_REPOSITORIO>
    cd proy_aerolineas-main
  ──────
  2. Crear el entorno virtual (venv)
  • En Linux / macOS:
    python3 -m venv venv

  • En Windows:
    python -m venv venv

  ──────
  3. Activar el entorno virtual

  • En Linux / macOS:
    source venv/bin/activate

  • En Windows (PowerShell):
    venv\Scripts\Activate.ps1
  (Si da error de permisos en PowerShell, usa: Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass y vuelve a ejecutarlo)
  • En Windows (CMD):
    venv\Scripts\activate.bat

  • En Windows (Git Bash):
    source venv/Scripts/activate


  (Sabrás que está activo porque aparecerá (venv) al inicio de tu línea de terminal).
  ──────
  4. Instalar las dependencias

  Con el entorno (venv) activo:

    pip install -r requirements.txt
  ──────
  5. Verificar que todo funcione

  • Probar en consola:
    python prueba_consola.py

  • Iniciar el servidor web:
    python app.py


  Abre tu navegador en:

    http://127.0.0.1:5050
